# DeepSeek Harness (dsh) Installation Guide
## What is DeepSeek Harness (dsh)?
DeepSeek Harness (abbreviated as dsh) is an open-source Agent framework developed by DeepSeek AI, with the core philosophy of "Everything is a Plugin".
### Key Features
- Plugin-based Architecture: Built on the Cordis framework, all functionalities (Web UI, model invocation, tool execution, etc.) are loaded as plugins, allowing flexible composition and extension
- Web UI: Built-in browser interface, providing visual operations at http://127.0.0.1:3080 by default
- CLI Tool: Command-line driven, supporting profile management, plugin management, etc.
- Open Source: MIT license, code hosted on GitHub
### Use Cases
- AI Agent development and debugging
- Model conversation and tool invocation
- Plugin development and testing
## How to Install DeepSeek Harness (dsh)
### Device Information:
- Host: DeepComputing FML13V05
- CPU: SpacemiT K3 (16 cores) @ 2.40 GHz
- Memory: 16 GiB 
- Operating System: Ubuntu 26.04 (Resolute Raccoon) riscv64
- Kernel: Linux 6.18.3-fm1l3v05
<img width="2256" height="1504" alt="截图 2026-09-16 18-01-48" src="https://github.com/user-attachments/assets/9c34093b-02ed-4d72-bb68-8a36d8336693" />


### Dependencies
#### Required Tools
| Tool | Version Requirement | Description |
|------|---------------------|-------------|
| **Node.js** | ^22.19.0 or >=24.0.0 | dsh depends on `node:sqlite` and native TypeScript type-stripping, **23.x is not supported** |
| **npm** | Comes with Node.js | Used to install the dsh package |
#### Installing Node.js
- Option 1: Using NodeSource Repository (Recommended)

 ```
# Add NodeSource 22.x repository
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -

# Install Node.js
sudo apt install -y nodejs

# Install npm
sudo apt install -y npm
```

- Option 2: Using nvm (Node Version Manager)

```
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# Reload shell
source ~/.bashrc

# Install Node.js 22.x
nvm install 22
```

#### Verify Version
After installation, confirm the version meets the requirements:
```
node -v
# Should output v22.19.0 or higher (e.g., v22.23.1)

npm -v
# Confirm npm is available
```

⚠️ Note: Node.js 23.x is not supported, please do not use it. It is recommended to use the 22.x LTS series.


### Installation Steps
#### Step 1: Install dsh Globally

```
sudo npm install -g @deepseek-ai/dsh
```

- Verify after installation:
```
dsh --version
```

#### Step 2: Start dsh Web UI
- Since dsh uses Node.js internal modules internally, you need to add the --expose-internals flag when starting:
```
node --expose-internals $(which dsh) web
```
⚠️ Note: Running dsh web directly may cause errors; you must add the --expose-internals parameter.
After successful startup, dsh will provide Web UI at 127.0.0.1:3080. Then open in browser: http://127.0.0.1:3080
<img width="2256" height="1504" alt="截图 2026-09-16 18-01-48" src="https://github.com/user-attachments/assets/1a961ee7-1675-433d-9845-673b0b4cedeb" />
<img width="1674" height="945" alt="image" src="https://github.com/user-attachments/assets/010cc029-24b0-4d82-a3a2-ddc8c13f7d30" />

- Configure the API and model in settings, and you can start using DeepSeek Harness
<img width="1520" height="902" alt="image" src="https://github.com/user-attachments/assets/c7266379-34f8-48c6-acc5-0c192cdaa166" />

### Common Issues
#### Port Already in Use (EADDRINUSE)
- If you see an error similar to:

```
Error: listen EADDRINUSE: address already in use 127.0.0.1:3080
```
- It means port 3080 is already in use (usually because the previous dsh did not exit properly). Solution:

```
# Check which process is using the port
lsof -i :3080

# Kill the process using the port
kill $(lsof -t -i :3080)

# Or kill all dsh processes
pkill -f dsh
```

Then restart.

#### Starting dsh with npx (No Installation Required)
You can also run it directly with npx without installing:
```
npx @deepseek-ai/dsh web
```
However, this method downloads the package each time, resulting in slower startup. It is recommended to install globally.

---

### Quick Reference for Startup Commands
| Scenario | Command |
|----------|---------|
| Start after global installation | `node --expose-internals $(which dsh) web` |
| Run directly with npx | `npx @deepseek-ai/dsh web` |
| Run from source | `git clone` → `pnpm install` → `pnpm run build` → `pnpm dsh web` |


### Reference Links
- GitHub Repository: https://github.com/deepseek-ai/deepseek-harness
- npm Package: https://www.npmjs.com/package/@deepseek-ai/dsh
- Official Documentation: https://dshdocs.com

## RISC-V Device Instructions
### Missing Dependencies Issue
- The issue of "Upper-layer Node.js software supports RISC-V, but the underlying pre-compiled native addons do not have RISC-V packages".
- Official RISC-V dependencies are not provided; you need to manually compile the dependency packages.
- Dependency packages:https://drive.google.com/file/d/1ZrdfFuzCpV-enImPYc02f2MFLq9jUWKe/view?usp=sharing

#### Extract and Install Dependency Packages

```
tar -xzf dsh-riscv-patch.tar.gz 

cd dsh-riscv-patch

# One-click install
sudo ./install.sh



# If you see the following, installation was successful:

flock: SUCCESS
landlock: full

==============================================
 SUCCESS
 dsh RISC-V native dependency is installed.
==============================================
```


- After installing the dependency packages, restart dsh and it should work normally
