# DC-ROMA RISC-V AI PC Install AI Models(Deepseek-7B) Guide 


### **Note:**
- Adjust the MMZ size first in U-Boot:
- Entering U-Boot:
  - Connect the AI PC via a serial port.
  - During power-on startup, repeatedly press Enter to access U-Boot.
- Modify Environment Variables:
```
# Configure env to output only to serial port 
env set stdout serial 

# Permanently change MMZ size (11GB example)
setenv fdt_cmd "fdt mmz mmz_nid_0_part_0 0x1c0000000 0x2c0000000"    #die0
setenv bootcmd "run fdt_cmd;${bootcmd}"
saveenv

#Reboot System
boot
```

- For more detailed instructions, please refer to:[DC-ROMA RISC-V AI PC, RISC-V Mainboard II NPU Memory Adjustment Instructions](https://github.com/DC-DeepComputing/Framework/blob/main/FML13V03/DC-ROMA%20RISC-V%20AI%20PC%2C%20RISC-V%20Mainboard%20II%20NPU%20Memory%20Adjustment%20Instructions.md)



### Single NPU AI Model Installation
- Download the model to the specified directory
```
# Single generation NPU 
 
# Create directory for storing models 
sudo mkdir /opt/eswin/sample-code/npu_sample/qwen_sample/models

# Download the model 
wget https://www.eswincomputing.com/community/api/uploads/2025/10/21/176103208318952c72cf954480.gz


# Extract the model
tar -xvf 176103208318952c72cf954480.gz

# Copy model files
sudo cp -r deepseek/models/deepseek_7b_1k_int8/ /opt/eswin/sample-code/npu_sample/qwen_sample/models/
 
# Check if copy succeeded 
ls /opt/eswin/sample-code/npu_sample/qwen_sample/models 
 
# Create model startup script file
vim /home/roma/start-deepseek.sh 

`
sudo /opt/eswin/sample-code/npu_sample/qwen_sample/bin/es_qwen2 /opt/eswin/sample-code/npu_sample/qwen_sample/src/deepseek_7b_1k_int8/config.json
`

# Grant script permissions
sudo chmod 777 /home/roma/start-deepseek.sh 
 
# Run script to start the model 
./start-deepseek.sh 

----------------------------------------------------------------------------------
0: Role setting: 你是一个智能助理.
----------------------------------------------------------------------------------
1: 介绍一下大语言模型
2: The quantum computers
3: Humans and robots coexist
4: Customized prompts
----------------------------------------------------------------------------------
[YOU]: 4
[YOU]: who are you
[Qwen2]: <think>

</think>

Greetings! I'm DeepSeek-R1, an artificial intelligence assistant created by DeepSeek. I'm at your service and would be delighted to assist you with any inquiries or tasks you might have.
--------------------------------------------------------
Throughput: 5.29529tokens/s
--------------------------------------------------------

```

### Dual NPU AI Model Installation
- Download the model to the specified directory
```
#Dual NPU

# Create directory for storing models 
sudo mkdir /opt/eswin/sample-code/npu_sample/qwen_sample/models

# Download the model 
wget https://www.eswincomputing.com/community/api/uploads/2025/10/21/17610330745f0343f1535bc4ab.gz

# Extract the model
tar -xvf 17610330745f0343f1535bc4ab.gz

# Copy model files
sudo cp -r deepseek/models/deepseek_7b_1k_int8_peer/ /opt/eswin/sample-code/npu_sample/qwen_sample/models/

# Check if copy succeeded 
ls /opt/eswin/sample-code/npu_sample/qwen_sample/models 
 
# Create model startup script file
vim /home/roma/start-deepseek.sh

`
sudo /opt/eswin/sample-code/npu_sample/qwen_sample/bin/es_qwen2 /opt/eswin/sample-code/npu_sample/qwen_sample/src/deepseek_7b_1k_int8_peer/config.json
`

# Grant script permissions
sudo chmod 777 /home/roma/start-deepseek.sh 
 
# Run script to start the model 
./start-deepseek.sh 

----------------------------------------------------------------------------------
0: Role setting: 你是一个智能助理.
----------------------------------------------------------------------------------
1: 介绍一下大语言模型
2: The quantum computers
3: Humans and robots coexist
4: Customized prompts
----------------------------------------------------------------------------------
[YOU]: 4
[YOU]: who are you?
[Qwen2]: <think>

</think>

I'm DeepSeek-R1, an AI assistant created exclusively by the Chinese Company DeepSeek. I'm excited to assist you! If you have any questions or need any help, feel free to ask.
--------------------------------------------------------
Throughput: 10.3292tokens/s
--------------------------------------------------------
```


### AI Model Deployment with Single NPU + Ollama Containerization

**1. Download and Prepare the AI Model (Skip if Already Downloaded)**
```
# Create directory for storing models 
sudo mkdir /opt/eswin/sample-code/npu_sample/qwen_sample/models

# Download the compressed DeepSeek model package 
wget https://www.eswincomputing.com/community/api/uploads/2025/10/21/176103208318952c72cf954480.gz

# Extract the downloaded archive
tar -xvf 176103208318952c72cf954480.gz

# Copy the extracted model files to the designated models directory
sudo cp -r deepseek/models/deepseek_7b_1k_int8/ /opt/eswin/sample-code/npu_sample/qwen_sample/models/


# Verify that the copy was successful by listing the contents of the target folder 
ls /opt/eswin/sample-code/npu_sample/qwen_sample/models

```

**2. Install Customized Ollama Runtime Package**

```
# Download the customized Ollama .deb package for RISC-V64 architecture
wget http://120.92.155.32:8082/artifactory/riscv/ESWIN/es-ollama_1.0.0_riscv64.deb

#Google drive
https://drive.google.com/file/d/1reqmnv9MQ3xaIliwCXAI9nJ0jcdoeMSh/view?usp=sharing


# Install the package using dpkg
dpkg -i es-ollama_1.0.0_riscv64.deb

rm -rf es-ollama_1.0.0_riscv64.deb
```


**3. Launch Ollama Service & Generate GGUF Model File**
- The Ollama service must run continuously in the background as a daemon process to serve incoming request
```
/usr/local/bin/es_ollama serve

/usr/local/bin/es_ollama create my-deepseek -f /opt/eswin/data/npu/gguf/Modelfile
```


**4. Download Web Frontend Interface**
```
cd /home/roma/

# Google Drive
https://drive.google.com/file/d/1IgUolyQyj5j4t2EWMy2_U-aik336JV-W/view?usp=sharing

chmod 777 aichat-deepseek7b-English.html
```

**5. Configure and Edit Startup Script for Ollama Server**
```
vim start-server.sh

`
OLLAMA_ORIGINS=* ENABLED_OLLAMA=1 OLLAMA_HOST=0.0.0.0 /usr/local/bin/es_ollama serve
`

chmod 777 start-server.sh 
```

**6. Run the System and Begin Interaction**
```
# Open a terminal on your laptop/desktop
cd /home/roma/

./start-server.sh
```
- Right-click inside the folder containing aichat-deepseek7b-English.html Select "Open with Firefox" (or another modern browser like Chrome) Type a question in the input box and press Enter Wait briefly for the first response; subsequent interactions are fast
-  First-time loading will take approximately 3 minutes while the model is loaded into memory and initialized on the NPU. Subsequent queries respond instantly.
<img width="1280" height="716" alt="image" src="https://github.com/user-attachments/assets/2ee26138-993e-4d60-9f69-743d50f39de4" />


