# How to install the bpftool toolkit on RISC-V

In modern Linux systems, eBPF (Extended Berkeley Packet Filter) has become a core technology for performance analysis, network monitoring, security auditing, and more. It allows users to run sandboxed programs in kernel space without modifying kernel source code or loading kernel modules. bpftool is the official command-line utility for interacting with eBPF, providing capabilities to load, manage, and debug eBPF programs and maps. It is an essential tool for eBPF developers and operators.

Whether developing eBPF applications, troubleshooting kernel issues, or monitoring eBPF programs in a system, mastering the installation and usage of bpftool is critical. This article details multiple methods to install bpftool across different Linux distributions—from simple package manager installations to source compilation—helping you get started quickly and resolve potential issues.

## What is bpftool?
bpftool is the official eBPF toolkit provided by the Linux kernel, initially maintained directly by kernel developers for interacting with the eBPF subsystem. Its key functionalities include:
- Managing eBPF Programs: Loading, unloading, and listing active eBPF programs (e.g., bpftool prog list).
- Operating eBPF Maps: Creating, deleting, querying, and inspecting map contents (e.g., bpftool map dump).
- Debugging & Analysis: Disassembling eBPF bytecode, viewing program licenses, and checking kernel BPF feature support (e.g., bpftool feature).
- Handling BPF Links and Attachment Points: Managing links between eBPF programs and kernel hooks (e.g., networking, tracepoints).

bpftool’s capabilities evolve with kernel updates. For example:
- Kernel 5.8 introduced bpftool trace for tracing eBPF programs.
- Kernel 5.13 enhanced support for BPF CO-RE (Compile Once – Run Everywhere).
Thus, installing a bpftool version matching your kernel is crucial.

## Installation Prerequisites
1. Kernel Version Requirements
bpftool depends on the kernel’s BPF subsystem, with minimum requirements:
- Minimum Kernel: 4.15 (released 2018; first feature-complete bpftool).
- Recommended Kernel: 5.4+ (LTS with expanded eBPF features and bpftool commands).
```
# Check kernel version
uname -r  # Example output: Linux roma 6.6.92-eic7x-2025.07
```
If your kernel is older than 4.15, upgrade it (e.g., via apt upgrade linux-image-generic or compiling a new kernel).
In this blog, we use the DC-ROMA RISC-V AI PC as the test environment. 
<img width="2256" height="1504" alt="image" src="https://github.com/user-attachments/assets/fad5b3a3-a4c5-406b-a7be-6abcd5543e6e" />



## Required Tools
- Source Compilation: Requires build tools (e.g., gcc, make) and libraries (e.g., libelf, zlib).
Verify tools with:
```
gcc --version  # Check compiler 
make --version # Check build tool 
git --version  # Check version control (for source download)
```
Install missing tools (e.g., on Ubuntu: `sudo apt install build-essential git`).

## Installing bpftool
Compile from Source
### Step 1: Download Kernel Source
- Option A (Git, latest version):
```  
it clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git  
cd linux 
git checkout v6.5   # Optional: Specify a version
```
  
- Option B (Stable release tarball):
```
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.5.tar.xz  
tar -xvf linux-6.5.tar.xz 
cd linux-6.5
```
### Step 2: Install Build Dependencies (Ubuntu Example)
```
sudo apt install -y libelf-dev zlib1g-dev llvm clang 
```
### Step 3: Compile & Install
```
cd tools/bpf/bpftool 
make              # Build dynamically linked binary 
sudo make install # Install to /usr/local/bin 
```

### Step 4: Verify Installation
```
which bpftool              # Should return /usr/local/bin/bpftool 
bpftool --version          # Show version and build info 
sudo bpftool prog list     # List loaded eBPF programs (empty if none)
19: cgroup_skb  name sd_fw_egress  tag 6deef7357e7b4530  gpl
        loaded_at 2026-01-27T10:41:41+0800  uid 0
        xlated 80B  jited 68B  memlock 4096B
20: cgroup_skb  name sd_fw_ingress  tag 6deef7357e7b4530  gpl
        loaded_at 2026-01-27T10:41:41+0800  uid 0
        xlated 80B  jited 68B  memlock 4096B
21: cgroup_device  name sd_devices  tag 2705a24f44b96941  gpl
        loaded_at 2026-01-27T10:41:41+0800  uid 0
        xlated 512B  jited 320B  memlock 4096B
22: cgroup_device  name sd_devices  tag 2705a24f44b96941  gpl
        loaded_at 2026-01-27T10:41:41+0800  uid 0
        xlated 512B  jited 320B  memlock 4096B
23: cgroup_device  name sd_devices  tag acde8ce2554847b8  gpl
        loaded_at 2026-01-27T10:41:41+0800  uid 0
        xlated 232B  jited 136B  memlock 4096B
24: cgroup_device  name sd_devices  tag ee0e253c78993a24  gpl
        loaded_at 2026-01-27T10:41:41+0800  uid 0
        xlated 456B  jited 290B  memlock 4096B
```
<img width="1280" height="853" alt="image" src="https://github.com/user-attachments/assets/7adaf28b-9c45-41c4-842e-3097db0ec44a" />


## Basic bpftool Usage Examples
- List All eBPF Programs
```
sudo bpftool prog list 
```
- Inspect eBPF Map Contents

```
sudo bpftool map list        # List all maps 
sudo bpftool map dump id 123 # Dump content of map ID 123 
```

- Check Kernel BPF Feature Support
```
bpftool feature  # e.g., BPF_CORE, BTF, trampolines 
```
## Conclusion
bpftool is indispensable in the eBPF ecosystem, and mastering its installation is the first step toward advanced eBPF development and operations. As eBPF evolves rapidly, regularly update bpftool (especially after kernel upgrades) to leverage new features. For further guidance, consult the bpftool official documentation or kernel mailing lists.

Follow us on [LinkedIn](https://www.linkedin.com/company/deepcomputing/), [X](https://x.com/DeepComputingio), and [Facebook](https://www.facebook.com/DeepComputingDC) for updates, join our [Discord community](https://discord.com/invite/DycykxSxWH) to connect and discuss, and visit our [website](https://store.deepcomputing.io/) to learn more about the RISC-V innovations.
