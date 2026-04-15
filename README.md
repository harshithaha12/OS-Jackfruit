# OS Project: Multi-Container Runtime System

---

## Team Information

Name: Harshitha Vangaveti
SRN: PES2UG24AM064

Name: Hamsa M N
SRN: PES2UG24AM060

---

## Overview

This project implements a mini container runtime system similar to Docker with:

- Namespace-based isolation  
- Supervisor-based architecture  
- FIFO-based control plane  
- Pipe-based logging  
- Kernel module monitoring  
- Graceful cleanup and shutdown  

---

## System Architecture

The system consists of the following components:

- **CLI Interface**: Accepts user commands (`run`, `start`)  
- **Supervisor Process**: Central controller managing containers  
- **FIFO (IPC)**: Communication channel between CLI and supervisor  
- **Container Runtime**: Uses clone() to create isolated containers  
- **Logging System**: Captures container output using pipes  
- **Kernel Module**: Monitors system activity using Linux kernel APIs  

### Flow:
CLI → FIFO → Supervisor → Container Creation → Logging → Monitoring

# Multi-Container Runtime

A lightweight Linux container runtime in C with a long-running supervisor and a kernel-space memory monitor.

Read [`project-guide.md`](project-guide.md) for the full project specification.

---

## Getting Started

### 1. Fork the Repository

1. Go to [github.com/shivangjhalani/OS-Jackfruit](https://github.com/shivangjhalani/OS-Jackfruit)
2. Click **Fork** (top-right)
3. Clone your fork:

```bash
git clone https://github.com/<your-username>/OS-Jackfruit.git
cd OS-Jackfruit
```

### 2. Set Up Your VM

You need an **Ubuntu 22.04 or 24.04** VM with **Secure Boot OFF**. WSL will not work.

Install dependencies:

```bash
sudo apt update
sudo apt install -y build-essential linux-headers-$(uname -r)
```

### 3. Run the Environment Check

```bash
cd boilerplate
chmod +x environment-check.sh
sudo ./environment-check.sh
```

Fix any issues reported before moving on.

### 4. Prepare the Root Filesystem

```bash
mkdir rootfs-base
wget https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-minirootfs-3.20.3-x86_64.tar.gz
tar -xzf alpine-minirootfs-3.20.3-x86_64.tar.gz -C rootfs-base

# Make one writable copy per container you plan to run
cp -a ./rootfs-base ./rootfs-alpha
cp -a ./rootfs-base ./rootfs-beta
```

Do not commit `rootfs-base/` or `rootfs-*` directories to your repository.

### 5. Understand the Boilerplate

The `boilerplate/` folder contains starter files:

| File                   | Purpose                                             |
| ---------------------- | --------------------------------------------------- |
| `engine.c`             | User-space runtime and supervisor skeleton          |
| `monitor.c`            | Kernel module skeleton                              |
| `monitor_ioctl.h`      | Shared ioctl command definitions                    |
| `Makefile`             | Build targets for both user-space and kernel module |
| `cpu_hog.c`            | CPU-bound test workload                             |
| `io_pulse.c`           | I/O-bound test workload                             |
| `memory_hog.c`         | Memory-consuming test workload                      |
| `environment-check.sh` | VM environment preflight check                      |

Use these as your starting point. You are free to restructure the repository however you want — the submission requirements are listed in the project guide.

### 6. Build and Verify

```bash
cd boilerplate
make
```

If this compiles without errors, your environment is ready.

### 7. GitHub Actions Smoke Check

Your fork will inherit a minimal GitHub Actions workflow from this repository.

That workflow only performs CI-safe checks:

- `make -C boilerplate ci`
- user-space binary compilation (`engine`, `memory_hog`, `cpu_hog`, `io_pulse`)
- `./boilerplate/engine` with no arguments must print usage and exit with a non-zero status

The CI-safe build command is:

```bash
make -C boilerplate ci
```

This smoke check does not test kernel-module loading, supervisor runtime behavior, or container execution.

---

# What are the tasks to be implemented

# Task 1: Multi-Container Runtime

Implemented container creation using clone().

### Features:
- PID namespace isolation  
- UTS namespace isolation (hostname = container ID)  
- Mount namespace isolation  
- chroot-based filesystem isolation  
- /proc mounted inside container  

### Result:
- hostname shows container-specific name  
- ps shows PID starting from 1  

---

# Task 2: Control Plane (FIFO IPC)

Implemented communication between CLI and supervisor using FIFO.

### Features:
- CLI commands (run/start) send requests  
- Supervisor listens on /tmp/engine_fifo  
- Requests parsed and container created dynamically  
- Supports multiple containers  

### Flow:
CLI → FIFO → Supervisor → clone() → Container

### Commands:
./engine supervisor ../rootfs-alpha  
./engine run alpha ../rootfs-alpha /bin/hostname  

---

# Task 3: Logging System

Implemented logging using pipe-based redirection.

### Features:
- stdout and stderr captured from container  
- logs forwarded to supervisor  
- real-time output display  

### Implementation:
- pipe() used for communication  
- dup2() redirects output  
- supervisor reads and prints logs  

---

# Task 4: Kernel Module Monitoring

Implemented a Linux kernel module for monitoring system activity.

### Features:
- Kernel module loads using insmod  
- Logs printed using printk  
- Output visible using dmesg  
- Module unload supported  

### Commands:
sudo insmod monitor.ko  
sudo dmesg | tail  
sudo rmmod monitor  

---

# Task 5: System Integration

Integrated all components into a complete system.

### Workflow:
1. Start kernel module  
2. Start supervisor  
3. Send container request  
4. Container starts and logs are captured  
5. Kernel logs verified using dmesg  

### Commands:
sudo insmod monitor.ko  
sudo ./engine supervisor ../rootfs-alpha  
sudo ./engine run alpha ../rootfs-alpha /bin/hostname  
sudo dmesg | tail  

### Result:
Full system works end-to-end.

---

# Task 6: Cleanup and Graceful Shutdown

Implemented proper cleanup and error handling.

### Features:
- FIFO removed on shutdown  
- Signal handling (Ctrl + C)  
- Zombie processes prevented using waitpid  
- Safe pipe creation and file descriptor handling  
- Graceful supervisor termination  

### Result:
System shuts down cleanly with no leftover resources.

---

# Build, Load, and Run Instructions

### Build
cd boilerplate  
gcc engine.c -o engine -lpthread  
make  

---

### Load Kernel Module
sudo insmod monitor.ko  
ls -l /dev/container_monitor  

---

### Start Supervisor
sudo ./engine supervisor ../rootfs-alpha  

---

### Run Container
sudo ./engine run alpha ../rootfs-alpha /bin/hostname  

---

### Check Logs
sudo dmesg | tail  

---

### Stop Supervisor
Press CTRL + C  

---

### Cleanup Check
ls /tmp  
(engine_fifo should NOT be present)

---

# Demo (Screenshots)

Include screenshots showing:

1. Supervisor running
   
2. Client request sent
    
3. Container output (hostname)
     
4. Kernel module logs
   
5. Clean shutdown

---

# Engineering Analysis

- Namespace isolation ensures process and system separation  
- Supervisor architecture centralizes control  
- FIFO provides simple IPC  
- Pipe-based logging enables real-time output capture  
- Kernel module enables low-level monitoring  

---

# Design Decisions and Tradeoffs

- clone() used for flexibility but requires manual setup  
- FIFO chosen for simplicity over scalability  
- Pipe logging sacrifices interactivity  
- Timer-based monitoring is simpler but not real-time  

---

# Conclusion

This project demonstrates:

- Container isolation using Linux namespaces  
- IPC mechanisms (FIFO + pipes)  
- Kernel-user interaction  
- Logging and monitoring  
- Resource cleanup and stability  

---

Project Successfully Completed!
