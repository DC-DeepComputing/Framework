## 1. Preparing NPU Execution Environment

Complete ESWIN's whisper library setup and NPU initialization:

**1）Install whisper library**
```
#Download files
https://drive.google.com/file/d/1rElsJe0taRlGri-YIziU_EsfMrnle1NN/view?usp=sharing
```

**2）Extract files**
```
tar -xvf voice.tar.gz
```

**3）Install whisper NPU dependencies**
```
# Create directory 
sudo mkdir -p /opt/eswin/data/npu/whisper_models

# Switch directory 
cd voice/es-hw-models/whisper/whisper_base 
 
# Copy model/config files to whisper+npu default directory 
sudo cp models/* /opt/eswin/data/npu/whisper_models/
sudo cp src/es_whisper_demo/es_whisper_config/* /opt/eswin/data/npu/whisper_models/
sudo cp -r models/audio/ /opt/eswin/data/npu/whisper_models/



# Copy library files 
sudo cp lib/* /usr/lib

## Verify directory contents 
ls /opt/eswin/data/npu/whisper_models/

audio/                  vocab.bin
config_demo.json        whisper-init_layer0_npu_b1.model
config_process.json     whisper-init_layer1_npu_b1.model
encoder_npu_b1.model    whisper-init_layer2_npu_b1.model
ggml-base.bin           whisper-init_layer3_npu_b1.model
mel_filters.bin         whisper-init_layer4_npu_b1.model
position_embedding.bin  whisper-init_layer5_npu_b1.model
token_embedding.bin


## Copy original model to VLC default directory 
sudo mkdir -p ~/.cache/vlc/models
sudo cp models/ggml-base.bin  ~/.cache/vlc/models/
```

-  Note: Original reference file at voice/es-hw-models/whisper/whisper_base/README.md  
```
cat voice/es-hw-models/whisper/whisper_base/README.md

# whisper_base Model: Using Quantized Models to Run Offline Models

## 1. Directory structure

whisper_base/
├── bin
│   └── whisper-cli
│   └── whisper-cli-precision
├── lib
│   └── libes_whisper_process.so
│   └── libggml-base.so
│   └── libggml-cpu.so
│   └── libggml.so
│   └── libwhisper.so
│   └── libwhisper.so.1
│   └── libwhisper.so.1.7.5
├── models
│   └── audio
│   └── encoder_npu_b1.model
│   └── ggml-base.bi
```

**4）Install es-ollama**

```
# Download es-ollama 
https://drive.google.com/file/d/10ZWuFl2_vG-OJja8ihoEl1uOwh_4T7zm/view?usp=sharing


sudo apt install ./es-ollama_1.0.2_riscv64.deb


# Verify installation paths:
cat /usr/local/bin/config.json   # Configuration 
cat /home/eswin/OLLAMA_README.md   # Documentation 
```

**5）Download qwen2 0.5B model**
```
sudo apt-get install git-lfs 

cd $HOME

git clone "http://82.157.114.25:8080/eswin/es-model-zoo"

cd es-model-zoo  

git lfs pull --include="llm/es-hw-models/qwen2/models/qwen2_0_5b_1k_int8" --exclude=""  
# es-model-zoo/llm/es-hw-models/qwen2/models/qwen2_0_5b_1k_int8

## Copy model to OLLAMA directory 
sudo mkdir -p /opt/eswin/sample-code/npu_sample/qwen_sample/models/
sudo cp -r llm/es-hw-models/qwen2/models/qwen2_0_5b_1k_int8 /opt/eswin/sample-code/npu_sample/qwen_sample/models

# Verify model copy 
ls /opt/eswin/sample-code/npu_sample/qwen_sample/models

# Expected: qwen2_0_5b_1k_int8/
```

**6）Start ollama service (run in background)**
```
# Start ollama
OLLAMA_ORIGINS=* OLLAMA_KEEP_ALIVE=-1 OLLAMA_HOST=0.0.0.0 /usr/local/bin/es_ollama serve

## First-time model initialization (critical!)

/usr/local/bin/es_ollama create my-qwen2 -f /opt/eswin/data/npu/gguf/Modelfile

parsing GGUF
using existing layer sha256:de4b758916f40e8b97ccaf660aa3da5a4d77e1f2ce6f66c5d7589090234e0f60
creating new layer sha256:a47b02e00552cd7022ea700b1abf8c572bb26c9bc8c1a37e01b566f2344df5dc
creating new layer sha256:75357d685f238b6afd7738be9786fdafde641eb6ca9a3be7471939715a68a4de
creating new layer sha256:8bb1c97c9209b4b6c0181fe6d4328b15a7cf639d37fe7ea5cbf5892575011c29
writing manifest
success
```

**7）Verify es_ollama functionality using this method (initial request loads qwen2 0.5B model, taking ~10s)**
```
curl http://localhost:11434/api/chat -d '{
  "model": "my-qwen2",
  "messages": [
    { "role": "user", "content": "你是一个专业翻译工具，请把下面的文字翻译为英文:今天的天气不错。" }
  ],
  "stream":  false
}'


{"model":"my-qwen2","created_at":"2025-11-03T08:26:16.634962305Z","message":
{"role":"assistant","content":"\nThe weather is quite nice today."},"done_reason":"stop","done":true,"total_duration":13806745433,"load_duration":4637933382,"prompt             _eval_count":36,"prompt_eval_duration":8919248923,"eval_count":9,"eval_duration"             :233289989}⏎   
```

- Troubleshooting ollama memory errors:
```
# Error example: 

ES_MEM：  Cannot allocate memory
Loading models: [=====================================>            ]  75.86% ( 59.940674 seconds )[E][ES_MEM]  dmabuf_heap_alloc_fdflags:  336 alloc fd failed, errno=12(Cannot allocate memory)

# Solution: Modify U-Boot memory settings 
env set stdout serial  # Output to serial only 
setenv fdt_cmd "fdt mmz mmz_nid_0_part_0 0x1c0000000 0x2c0000000"  # 11G config 
setenv bootcmd "run fdt_cmd;${bootcmd}"  # Update boot command 
saveenv  # Save changes
boot   #Reboot
# Reboot required after modification 
```


**8）Configure whisper on NPU1 / ollama on NPU0**
```
sudo vi /opt/eswin/data/npu/whisper_models/config_process.json 

# Change "device_id": 0 → "device_id": 1
```

**9) Test whisper**
```
# Expected speech-to-text output 
cd voice/es-hw-models/whisper/whisper_base

./bin/whisper-cli -f /opt/eswin/data/npu/whisper_models/audio/jfk.wav
```

- NPU permission issue and loadModel(276) error=-1609605048 encountered when executing whisper-cli in es-hw-models/whisper.
```
# Reason: NPU devices belong to 'eswin' user group by default, current user lacks permissions 
cat /etc/udev/rules.d/86-ai.rules
KERNEL=="es-dsp0", MODE="0660", GROUP="eswin", OWNER="eswin"
KERNEL=="es-dsp1", MODE="0660", GROUP="eswin", OWNER="eswin"
KERNEL=="es-dsp2", MODE="0660", GROUP="eswin", OWNER="eswin"
KERNEL=="es-dsp3", MODE="0660", GROUP="eswin", OWNER="eswin"
KERNEL=="es-dsp4", MODE="0660", GROUP="eswin", OWNER="eswin"
KERNEL=="es-dsp5", MODE="0660", GROUP="eswin", OWNER="eswin"
KERNEL=="es-dsp6", MODE="0660", GROUP="eswin", OWNER="eswin"
KERNEL=="es-dsp7", MODE="0660", GROUP="eswin", OWNER="eswin"

KERNEL=="npu0", MODE="0660", GROUP="eswin", OWNER="eswin"
KERNEL=="npu1", MODE="0660", GROUP="eswin", OWNER="eswin"
KERNEL=="es_hae", MODE="0660", GROUP="eswin", OWNER="eswin"
KERNEL=="mmz_nid_0_part_0", MODE="0660", GROUP="eswin", OWNER="eswin"
KERNEL=="mmz_nid_1_part_0", MODE="0660", GROUP="eswin", OWNER="eswin"


# Check NPU device permissions 
ls -l /dev/npu0 /dev/npu1
crw-rw---- 1 root root 498, 0 Oct 16 10:31 /dev/npu0
crw-rw---- 1 root root 497, 0 Oct 16 10:31 /dev/npu1


# Solution: Add current user to 'eswin' group 

# Switch to root privileges (admin password required)
sudo su


# Check if 'eswin' group exists
cat /etc/group | grep eswin
 
# Create group if missing 
groupadd eswin
 
# Add user 'roma' to eswin group 
usermod -aG eswin roma
 
# Reload udev rules (ensure permission changes take effect)
udevadm control --reload-rules
udevadm trigger 
 
# Exit root session
exit
 
## IMPORTANT: System reboot required after configuration
## Post-modification verification:

# Correct permissions after fix:
ls -l /dev/npu0 /dev/npu1 
crw-rw---- 1 root eswin 498, 0 Jul  2 22:04 /dev/npu0 
crw-rw---- 1 root eswin 497, 0 Jul  2 22:04 /dev/npu1 
```

**10) Re-test whisper to verify normal operation**
```
cd voice/es-hw-models/whisper/whisper_base

./bin/whisper-cli -f /opt/eswin/data/npu/whisper_models/audio/jfk.wav

whisper_init_from_file_with_params_no_state: loading model from '/opt/eswin/data/npu/whisper_models/ggml-base.bin'
whisper_init_with_params_no_state: use gpu    = 1
whisper_init_with_params_no_state: flash attn = 0
whisper_init_with_params_no_state: gpu_device = 0
whisper_init_with_params_no_state: dtw        = 0
whisper_init_with_params_no_state: devices    = 1
whisper_init_with_params_no_state: backends   = 1
whisper_model_load: loading model
whisper_model_load: n_vocab       = 51865
whisper_model_load: n_audio_ctx   = 1500
whisper_model_load: n_audio_state = 512
whisper_model_load: n_audio_head  = 8
whisper_model_load: n_audio_layer = 6
whisper_model_load: n_text_ctx    = 448
whisper_model_load: n_text_state  = 512
whisper_model_load: n_text_head   = 8
whisper_model_load: n_text_layer  = 6
whisper_model_load: n_mels        = 80
whisper_model_load: ftype         = 1
whisper_model_load: qntvr         = 0
whisper_model_load: type          = 2 (base)
whisper_model_load: adding 1608 extra tokens
whisper_model_load: n_langs       = 99
whisper_model_load:          CPU total size =   147.37 MB
whisper_model_load: model size    =  147.37 MB
whisper_backend_init_gpu: no GPU found
whisper_init_state: kv self size  =    6.29 MB
whisper_init_state: kv cross size =   18.87 MB
whisper_init_state: kv pad  size  =    3.15 MB
whisper_init_state: compute buffer (conv)   =   16.26 MB
whisper_init_state: compute buffer (encode) =   85.86 MB
whisper_init_state: compute buffer (cross)  =    4.65 MB
whisper_init_state: compute buffer (decode) =   96.35 MB

system_info: n_threads = 1 / 8 | WHISPER : COREML = 0 | OPENVINO = 0 | CPU : OPENMP = 1 | AARCH64_REPACK = 1 |

main: processing '/opt/eswin/data/npu/whisper_models/audio/jfk.wav' (176000 samples, 11.0 sec), 1 threads, 1 processors, 1 beams + best of 5, lang = en, task = transcribe, timestamps = 1 ...

    
[00:00:00.000 --> 00:00:07.600]   And so my fellow Americans, ask not what your country can do for you,
[00:00:07.600 --> 00:00:10.600]   ask what you can do for your country.

whisper_print_timings:     load time =  1371.81 ms
whisper_print_timings:     fallbacks =   0 p /   0 h
whisper_print_timings:      mel time =    85.67 ms
whisper_print_timings:   sample time =    58.80 ms /     1 runs (    58.80 ms per run)
whisper_print_timings:   encode time =  1755.65 ms /     1 runs (  1755.65 ms per run)
whisper_print_timings:   decode time =   943.34 ms /    29 runs (    32.53 ms per run)
whisper_print_timings:   batchd time =   108.68 ms /     3 runs (    36.23 ms per run)
whisper_print_timings:   prompt time =     0.00 ms /     1 runs (     0.00 ms per run)
whisper_print_timings:    total time =  7311.97 ms
```

## 2. Running VLC Media Player
**1) Download VLC build**
```
wget http://120.92.155.32:8082/artifactory/virtOS/AI_DEMO/vlc_build-20251015-with-stt_on_npu.tar
```

**2）Test VLC**

  a.Extract VLC build package to HOME directory
    
  ```
  tar -xvf vlc_build-20251015-with-stt_on_npu.tar
  ```

  b. For subtitle translation testing, download run.py and place it in the HOME directory at the same level as the vlc_build folder.
  ```  
  #run.py
  https://drive.google.com/file/d/16z4rruzHdbTb_g3uhSjCXSp9OkHZ0dp3/view?usp=sharing

  #VLC_Test.mp4
  https://drive.google.com/file/d/14MkPBvnesKj_thhgH5v6jg2gVKmgkZY3/view?usp=sharing
  ```

  c. Terminal 1: Start Ollama Service (required for subtitle translation)
  ```  
  OLLAMA_ORIGINS=* OLLAMA_KEEP_ALIVE=-1 OLLAMA_HOST=0.0.0.0 /usr/local/bin/es_ollama serve
  ```

  d. Terminal 2: Execute VLC Tests
  ```
  cd vlc_build

  # Set up environment variables
  export LD_LIBRARY_PATH=$(pwd)/lib:$(pwd)/lib/vlc:$LD_LIBRARY_PATH

  # Basic playback
  ./bin/vlc -f ~/Downloads/VLC_Test.mp4 

  ## STT subtitles + loop playback (for short videos)
  ./bin/vlc -f ~/Downloads/VLC_Test.mp4   --stt --repeat

  ## Speech translation
  ## (supported languages: cmn=Chinese, jpn=Japanese, eng=English, fra=French)
  ./bin/vlc -f ~/Downloads/VLC_Test.mp4  --stt --whisper-translate cmn --repeat

  # Custom resolution (optional)
  ./bin/vlc -f ~/Downloads/VLC_Test.mp4  --width=1600 --height=1024
  ```

  e. Terminal 3: Monitor NPU Load
  ```
sudo /opt/eswin/bin/es_hw_watcher 
```

- Expected Test Results

<img width="1280" height="960" alt="image" src="https://github.com/user-attachments/assets/cb978e0e-a98d-4690-8f0f-1f89c8c28625" />

<img width="1280" height="828" alt="image" src="https://github.com/user-attachments/assets/efa19cda-3259-4466-8566-b7b49133f844" />


