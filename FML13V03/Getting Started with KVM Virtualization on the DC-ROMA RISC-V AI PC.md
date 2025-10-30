# Getting Started with KVM Virtualization on the DC-ROMA RISC-V AI PC

KVM (Kernel-based Virtual Machine) is an open-source virtualization technology based on the Linux kernel. It allows the Linux kernel itself to function as a Type 1 (bare-metal) hypervisor. This means that a physical server can run multiple isolated virtual environments, or virtual machines (VMs), simultaneously. Each VM has its own virtualized hardware (CPU, memory, disk, network card, etc.) and can run independent operating systems such as Linux or Windows.
## Environment Preparation

- Device Information:

This demo is running on the DC-ROMA RISC-V AI PC, powered by a RISC-V AI SoC with a high-performance CPU and NPU. It supports hardware-accelerated virtualization through KVM (Kernel-based Virtual Machine) and QEMU, making it ideal for developers experimenting with virtualized environments on RISC-V.
<img width="1280" height="853" alt="image" src="https://github.com/user-attachments/assets/4008e4e0-3a7a-4b04-8630-9e44ff76f2af" />

- System Environment

```
sudo apt install -y virt-manager       # Graphical management tool
sudo apt install -y libvirt-dev        # Libvirt development libraries
sudo systemctl enable libvirtd         # Enable libvirtd on startup
sudo systemctl start libvirtd          # Start libvirt service
sudo apt install -y u-boot-qemu        # U-Boot
sudo apt install -y qemu-efi-riscv64   # EDK2 boot utility
sudo dpkg -i edk2-riscv64_20250221-9_all.deb   # EDK2 toolset ported from Fedora for RISC-V platform
sudo modprobe kvm                      # Manually load KVM driver (not auto-loaded at boot)
sudo adduser $USER libvirt
sudo adduser $USER kvm
```
- RISC-V platform EDK2 boot tools:
 👉 https://github.com/DC-Yanwei/RISC-V-Development-Tools.git

 ## Running Ubuntu 24.04 Server via QEMU Virtualization
 
 ### Download the image:
 
```
wget https://cdimage.ubuntu.com/releases/24.04/release/ubuntu-24.04.3-preinstalled-server-riscv64.img.xz
```

### Decompress the image:

```
xz -dkv -T0 ubuntu-24.04.3-preinstalled-server-riscv64.img.xz
```

### Run the image:

```
qemu-system-riscv64 --enable-kvm -M virt -cpu host \
    -m 8192 -smp 4 -nographic \
    -device virtio-net-device,netdev=eth0 -netdev user,id=eth0 \
    -device virtio-rng-pci \
    -kernel /usr/lib/u-boot/qemu-riscv64_smode/uboot.elf \
    -drive file=ubuntu-24.04.03-preinstalled-server-riscv64.img,format=raw,if=virtio
```

Default credentials:
- Username: ubuntu
- Password: ubuntu
- Important: On first boot, you will be required to change the default password. Please follow the on-screen instructions.



## Running Fedora Server via QEMU Virtualization

### Download the image:

```
wget https://dl.fedoraproject.org/pub/alt/risc-v/release/42/Server/riscv64/images/Fedora-Server-Host-Generic-42.20250414-8635a3a5bfcd.riscv64.raw.xz
```

### Decompress the image:

```
xz -dkv -T0 Fedora-Server-Host-Generic-42.20250414-8635a3a5bfcd.riscv64.raw.xz
```

### Run the image:

```
qemu-system-riscv64 --enable-kvm -M virt -cpu host \
    -m 8192 -smp 4 -nographic \
    -device virtio-net-device,netdev=eth0 -netdev user,id=eth0 \
    -device virtio-rng-pci \
    -kernel /usr/lib/u-boot/qemu-riscv64_smode/uboot.elf \
    -drive file=Fedora-Server-Host-Generic-42.20250414-8635a3a5bfcd.riscv64.raw,format=raw,if=virtio
```

Default credentials:
- Username: fedora
- Password: linux

## Fedora-KDE + QEMU Desktop System

### Download the image:

```
wget https://openkoji.iscas.ac.cn/pub/temp/qemu-kde.img.gz
```

### Decompress the image:

```
gzip -d qemu-kde.img.gz
```

### Run the image:

```
qemu-system-riscv64 --enable-kvm -M virt,pflash0=pflash0,acpi=off -m 6G -smp 4 \
  -blockdev node-name=pflash0,read-only=on,driver=qcow2,file.driver=file,file.filename=/usr/share/edk2/riscv/RISCV_VIRT_CODE.qcow2 \
  -device virtio-blk-pci,drive=hd0 \
  -device virtio-net-device,netdev=usernet \
  -netdev user,id=usernet \
  -device virtio-vga \
  -device qemu-xhci -usb -device usb-kbd -device usb-tablet \
  -drive file=qemu-kde.img,format=raw,id=hd0,if=none \
  -cpu rv64
```

Default credentials:
- Username: root
- Password: riscv


## Advanced Usage: Launch Fedora/Ubuntu Server via Virt-Manager
- Convert RAW image to qcow2 format:
```
qemu-img convert -p -f raw -O qcow2 ubuntu-24.04.03-preinstalled-server-riscv64.img ubuntu-server.qcow2
qemu-img convert -p -f raw -O qcow2 Fedora-Server-Host-Generic-42.20250414-8635a3a5bfcd.riscv64.raw fedora-server.qcow2
```

- Steps in Virt-Manager:
1. Go to File → New Virtual Machine → Select Import existing disk image.
 <img width="528" height="566" alt="image" src="https://github.com/user-attachments/assets/bd7f3119-6b53-43d2-8dc8-8c80ca5d6ccf" />

2. Add a new storage pool: set the directory (e.g., /home/roma) containing your qcow2 files as the disk pool.
<img width="1255" height="661" alt="image" src="https://github.com/user-attachments/assets/3eba12f5-c12d-4d6b-875b-0948b44b19e2" />

3. Copy ```/usr/lib/u-boot/qemu-riscv64_smode/uboot.elf``` into the same directory (e.g., /home/roma) for easier access.

4. Choose Customize configuration before install.
<img width="1255" height="661" alt="image" src="https://github.com/user-attachments/assets/504b3e7b-458b-47fe-ad40-2e3d9e75ae44" />

5. Add the boot kernel uboot.elf.
<img width="1064" height="872" alt="image" src="https://github.com/user-attachments/assets/2cbe98e1-210a-4b36-9915-d635f2da6393" />

6. Proceed with normal installation and run.
<img width="1280" height="735" alt="image" src="https://github.com/user-attachments/assets/11c14cdf-c634-40ae-8167-23ba8458c4f5" />

### Reference
https://milkv.io/zh/docs/megrez/development-guide/kvm
https://fedoraproject.org/wiki/Architectures/RISC-V/QEMU
https://images.fedoravforce.org/QEMU
