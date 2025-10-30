# DC-ROMA RISC-V Mainboard II AI Model User Guide (Ubuntu AI Image)

This document applies to users of the factory-installed image system or those who download the image from:
http://120.92.155.32:8082/artifactory/virtOS/fml13v03-eswin/15019-ubuntu-24.04-desktop-grub-sdcard-AI.zip



## Ollama Containerized Deepseek-R1-7B Usage Guide
The AI PC comes pre-installed with a containerized local DeepSeek 7B model via Ollama and includes a startup script. Simply run the script to use the model. For an enhanced experience, a visual frontend is included.

1.Open Terminal and run:
```
sudo  ./start-server.sh
```

2.Double-click ollama-deepseek7b.html to open the visual interface.
<img width="495" height="285" alt="image" src="https://github.com/user-attachments/assets/b698d119-d7a8-4a40-bc0d-346180abdc34" />



3.Ask questions in the dialog box. Note: Initial query loads the model (~3 minutes).
<img width="494" height="268" alt="image" src="https://github.com/user-attachments/assets/5e090434-52d0-4c3d-95ba-bc6e37921198" />


4.Sample Execution

<img width="495" height="274" alt="image" src="https://github.com/user-attachments/assets/98157c40-ec03-434c-8f5a-2b8219e0b8f0" />



5、You can view the usage steps in the ```/home/eswin/OLLAMA_README.md``` directory.



## Terminal Execution: Deepseek-7B with Dual-Gen NPU Acceleration
1、 Launch Model via Terminal:
```
sudo /opt/eswin/sample-code/npu_sample/qwen_sample/bin/es_qwen2 /opt/eswin/sample-code/npu_sample/qwen_sample/src/deepseek_7b_1k_int8_peer/config.json
```

2、 Select the interaction pattern according to the prompt
<img width="495" height="139" alt="image" src="https://github.com/user-attachments/assets/6a2bdd62-e30a-496c-b307-d75128e972a3" />



## Terminal Execution: Whisper Model
1、Open Terminal and run the command to view the model usage guide:
```
cat  /home/eswin/whisper/WHISPER_README.md

# es_whisper_deb using

# using deb
sudo apt install ./es-whisper_1.0.0_riscv64.deb
# run whisper-cli
/usr/bin/whisper-cli -f /opt/eswin/data/npu/whisper_models/audio/jfk.wav
```

2、Enter the command to launch the model, which will run a demo converting speech to text.
```
sudo /usr/bin/whisper-cli -f /opt/eswin/data/npu/whisper_models/audio/jfk.wav
```
<img width="495" height="263" alt="image" src="https://github.com/user-attachments/assets/2509a009-9b18-4525-8c48-d2c0404fdfa2" />



## Note:
- If an "Insufficient Memory" error occurs when running AI models, adjust the MMZ size first in U-Boot:

- Entering U-Boot:
  - Connect the AI PC via a serial port.
  - During power-on startup, repeatedly press Enter to access U-Boot.


- Modify Environment Variables:

```
# Configure env to output only to serial port 
env set stdout serial 

# Permanently change MMZ size (16GB example)
setenv fdt_cmd "fdt mmz mmz_nid_0_part_0 0x1c0000000 0x5c0000000"  
setenv bootcmd "run fdt_cmd;${bootcmd}"
saveenv

#Reboot System:
boot
```






