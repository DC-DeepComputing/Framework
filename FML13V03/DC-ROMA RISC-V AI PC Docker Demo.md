Docker is an open-source containerization platform built on Linux kernel technologies such as cgroups and namespaces, enabling process-level resource isolation. Its primary objective is to standardize the packaging of applications along with their dependencies, thereby addressing the common operational issue: "It works on my machine, but fails in production."

## Host Machine Configuration
(Refer to attached image)
- Note: The host system details (e.g., CPU, memory, OS) are visible in the provided screenshot, which shows a Linux environment prepared for Docker deployment.
- This demo is running on DC ROMA RISC-V AI PC ,RISC-V Mainboard II device. The OS version is Ubuntu 24.04.
<img width="1280" height="611" alt="image" src="https://github.com/user-attachments/assets/9147da1a-715d-44d4-97d8-3a17a5bd95eb" />


## Environment Preparation
```
#Before installing Docker, ensure the package index is up-to-date:
sudo apt-get update

#Install essential system tools and dependencies required for Docker setup:
sudo apt-get install -y systemd systemd-sysv sudo net-tools ethtool ifupdown iputils-ping vim ssh bash-completion parted apt-transport-https ca-certificates curl gnupg-agent software-properties-common


sudo apt-get install fuse-overlayfs docker.io
```

## Running a RISC-V Architecture Ubuntu Container

```
# Execute the following command to launch an interactive session using a RISC-V 64-bit (riscv64) Ubuntu image:
sudo docker run -it riscv64/ubuntu bash 

# Once inside the container, verify the architecture and operating system:
root@ca7685280162:/# uname -a
Linux ca7685280162 5.15.0 #2 SMP Fri Aug 25 13:56:44 CST 2023 riscv64 riscv64 riscv64 GNU/Linux
root@ca7685280162:/# cat /etc/issue
Ubuntu 22.04.1 LTS \n \l
```
<img width="1280" height="853" alt="image" src="https://github.com/user-attachments/assets/9af477bd-f643-484f-881f-1a846a7bbf22" />


## Extended Guide: Docker Fundamentals
- Image Management

```
# Downloads the latest official Ubuntu image from Docker Hub. You can specify tags like ubuntu:22.04.
docker pull ubuntu

#Lists all locally stored images, showing repository, tag, image ID, creation time, and size.
docker images

#Removes a specific image by ID or name. Add -f to force deletion if the image is in use.
docker rmi <image_id>

#Loads one or more images from a tar archive created via docker save. Useful for offline transfers or backups.
docker load -i my_images.tar
```
- Container Operations

```
#Starts a new container interactively (-i), allocates a pseudo-TTY (-t), and executes /bin/bash. Ideal for debugging and exploration.
docker run -it riscv64/ubuntu bash

docker ps -a    
#Displays all containers—both running and stopped—alongside their status, start time, and names. Use -q to output only IDs.

docker start/stop/restart my_images
#Controls container lifecycle without recreating it. Useful for managing long-running services.

docker exec -it my_images bash
#Executes a command inside a running container. Commonly used to enter a shell for inspection or troubleshooting.

docker rm -f my_images 
#Forcefully removes a container regardless of its state. The -f flag sends a SIGKILL signal before removal.
```
