# Virtualization Internals: The Engine Room of the Cloud

---
title: "Virtualization Internals"
parent: "[[01_Foundations]]"
tags:
  - foundations
  - virtualization
  - docker
  - kubernetes
  - cloud
  - container-breakout
created: 2026-01-23
updated: 2026-01-23
---

> **Executive Summary**: Modern infrastructure is rarely "bare metal." It is layers upon layers of abstraction. To attack or defend these environments, you must understand how the abstraction works. This chapter dissects the difference between Type 1/2 Hypervisors and Containers, explains the Linux kernel features that power Docker (Namespaces/Cgroups), and provides deep technical walkthroughs on how to break out of them.

---

## Table of Contents
1. [Learning Objectives](#1-learning-objectives)
2. [Hypervisor Architecture: Rings and Privilege](#2-hypervisor-architecture-rings-and-privilege)
3. [Container Architecture: It's Just a Process](#3-container-architecture-its-just-a-process)
4. [Deep Dive: Namespaces & Cgroups](#4-deep-dive-namespaces--cgroups)
5. [Docker Internals & Security](#5-docker-internals--security)
6. [Kubernetes (K8s) Fundamentals](#6-kubernetes-k8s-fundamentals)
7. [Container Breakout Techniques](#7-container-breakout-techniques)
8. [Critical Analysis & Checkpoints](#8-critical-analysis--checkpoints)

---

## 1. Learning Objectives

By the end of this chapter, you will be able to:

- **Differentiate Architectures**: Explain the kernel-level differences between ESXi, Docker, and Firecracker.
- **Identify Environments**: Use command-line forensics to determine if you are inside a VM, a Container, or Bare Metal.
- **Exploit Misconfigurations**: Break out of a Docker container using Mounted Sockets, Privileged Mode, and Kernel Exploits.
- **Understand Micro-Segmentation**: Explain how Kubernetes Network Policies work (and fail).
- **Secure the Stack**: Implement AppArmor profiles and User Namespaces to harden containers.

---

## 2. Hypervisor Architecture: Rings and Privilege

### 2.1 CPU Protection Rings
x86 architecture uses "Rings" to manage privilege.
- **Ring 0 (Kernel Mode)**: Direct hardware access. The OS Kernel lives here.
- **Ring 3 (User Mode)**: No hardware access. Applications live here.
- **Ring -1 (Hypervisor Mode)**: Created by Intel VT-x/AMD-V. This is where the Hypervisor (VMM) lives.

### 2.2 Type 1 vs Type 2 (Revisited Technically)

**Type 1 (Native/Bare Metal)**:
- **Architecture**: The Hypervisor IS the kernel.
- **Execution**: The Guest OS Ring 0 instructions are trapped by Ring -1.
- **Examples**: ESXi, Xen, Hyper-V (Root Partition).
- **Attack Surface**: Attacks target the Hypervisor logic itself (VM Escape).

**Type 2 (Hosted)**:
- **Architecture**: Hypervisor is a process in Host Ring 3, using a Kernel Driver (Ring 0) to talk to VT-x.
- **Examples**: VMware Workstation, VirtualBox.
- **Attack Surface**: The Hypervisor process, the Host Kernel driver, and the Host OS userland.

### 2.3 VM Escape (Theory)
A VM Escape is the "Holy Grail" of exploits.
1.  **Guest Ring 0**: Attacker gains root on Guest.
2.  **The Trigger**: Attacker interacts with a virtual device (e.g., Virtual Floppy Driver, Virtual SVGA Driver).
3.  **The Bug**: The parsing code in the Hypervisor (Ring -1/Host Ring 3) has a buffer overflow.
4.  **The Escape**: Code execution jumps from Guest Memory to Host Memory.

---

## 3. Container Architecture: It's Just a Process

"Containers do not exist." - *Jessie Frazelle*

There is no "Container" object in the Linux Kernel. A container is just a normal Linux process with restricted view (Namespaces) and restricted usage (Cgroups).

### 3.1 VM vs Container
| Feature | Virtual Machine | Container |
|:---|:---|:---|
| **Kernel** | Independent (Guest Kernel) | Shared (Host Kernel) |
| **Boot Time** | Minutes | Milliseconds |
| **Isolation** | Strong (Hardware Virtualization) | Weak (Software Namespace) |
| **Overhead** | High (Duplicated OS) | Low (Just the App) |

### 3.2 The Shared Kernel Risk
If you crash the kernel in a VM, only that VM dies.
If you crash the kernel in a Container, **the Host dies** (and all other containers).
If you exploit the kernel in a Container (e.g., Dirty COW), **you own the Host**.

---

## 4. Deep Dive: Namespaces & Cgroups

These are the two pillars of Linux Containers (LXC/Docker).

### 4.1 Namespaces (Isolation)
Namespaces wrap a global system resource in an abstraction that makes it appear to the processes within the namespace that they have their own isolated instance of the global resource.

1.  **PID Namespace**:
    - **Host**: Process is PID 12345.
    - **Container**: Process sees itself as PID 1.
2.  **Network Namespace (netns)**:
    - Container gets its own `eth0`, IP, and Routing Table.
3.  **Mount Namespace (mnt)**:
    - Container sees a distinct file system root (`/`).
4.  **UTS Namespace**:
    - Container has its own Hostname.
5.  **IPC Namespace**:
    - Isolated Inter-Process Communication (Shared Memory).
6.  **User Namespace** (Critical):
    - Maps UID 0 (Root) inside container to UID 1000 (Nobody) outside. **Often disabled by default.**

### 4.2 Control Groups (Cgroups)
Cgroups limit, account for, and isolate the resource usage (CPU, memory, disk I/O, network) of a collection of processes.
- **CPU**: "This container gets 10% CPU."
- **Memory**: "This container gets 512MB RAM." (OOM Killer kills it if it exceeds).
- **Security**: Can prevent a container from DoS-ing the host.

---

## 5. Docker Internals & Security

### 5.1 The Docker Daemon
- `dockerd`: Runs as **Root**.
- Listens on `/var/run/docker.sock` (Unix Socket) by default.
- If exposed on TCP (`0.0.0.0:2375`), it is an unauthenticated RCE-as-Root API.

### 5.2 Image Layering (UnionFS)
Docker images are built like a cake (OverlayFS).
- **Base Layer**: Alpine Linux (Read-Only).
- **Layer 1**: `apt install python` (Read-Only).
- **Layer 2**: `COPY app.py` (Read-Only).
- **Top Layer**: Container Write Layer (Read-Write).
- **Secret Leak**: If you delete a secret in Layer 3, it **still exists** in Layer 2. Hackers can inspect history to recover it.

### 5.3 Capabilities
Linux splits "Root" privileges into granular units called Capabilities.
Docker drops most by default, but keeps some dangerous ones.
- `CAP_NET_RAW`: Allows ping (and ARP spoofing).
- `CAP_SYS_ADMIN`: The "God" capability. Allows mounting, namespace manipulation, etc.

---

## 6. Kubernetes (K8s) Fundamentals

Docker manages one node. Kubernetes manages a fleet.

### 6.1 Architecture
- **Control Plane (Master)**:
    - **API Server**: The brain (REST API).
    - **Etcd**: The memory (Key-Value store, contains secrets).
    - **Scheduler**: Decides where pods go.
- **Worker Nodes**:
    - **Kubelet**: Agent that talks to API Server.
    - **Kube-Proxy**: Manages network rules.
    - **Pods**: One or more containers sharing a network namespace.

### 6.2 The Attack Surface
1.  **Misconfigured RBAC**: Role-Based Access Control allowing a user to list secrets.
2.  **Exposed Kubelet (10250)**: If anonymous auth is enabled, RCE on the node.
3.  **SSR to Metadata**: Pods can hit the Cloud Metadata service (AWS IMDSv1) to steal Node credentials.

---

## 7. Container Breakout Techniques

You are inside a container. How do you get to the host?

### 7.1 Identification: "Am I in a box?"
1.  **Check `.dockerenv`**: `ls -la /.dockerenv`.
2.  **Check Cgroups**: `cat /proc/1/cgroup`. Look for `/docker/` or `/kubepods/`.
3.  **Check Process Count**: `ps aux`. If you only see 5 processes and PID 1 is your shell/app, you are contained.

### 7.2 Breakout 1: The Mounted Socket
**Scenario**: The container has access to the host's Docker socket (e.g., Jenkins, Portainer).
**Detection**: `find / -name docker.sock`
**Exploit**: Use the socket to tell the host Docker engine to spawn a *new* container that mounts the host's root directory.
```bash
# We need the docker binary. If not present, download static binary.
docker -H unix:///var/run/docker.sock run -v /:/host -it alpine chroot /host sh
```
**Result**: You are now Root on the Host OS.

### 7.3 Breakout 2: Privileged Mode (`--privileged`)
**Scenario**: Container started with `--privileged`.
**Mechanism**: This disables Seccomp filters and enables all Capabilities.
**Exploit**: Mount the host's hard drive.
```bash
# List drives
fdisk -l
# Mount host disk
mkdir /host
mount /dev/sda1 /host
# Enter host
chroot /host
```

### 7.4 Breakout 3: Kernel Exploits (Dirty COW)
**Scenario**: Host kernel is old (vulnerable to CVE-2016-5195).
**Mechanism**: Container shares the kernel.
**Exploit**: Run Dirty COW exploit inside container. It exploits the Copy-On-Write mechanism to write to a Read-Only file on the Host (e.g., vDSO or `/etc/passwd`).

### 7.5 Breakout 4: release_agent (Cgroups v1)
**Scenario**: Container runs as root + `CAP_SYS_ADMIN`.
**Mechanism**: Abuse the `notify_on_release` feature of cgroups. When a cgroup is emptied, the kernel executes a command defined in `release_agent`. We point this to a script we create.
**Complexity**: High. Requires mounting cgroupfs.

---

## 8. Critical Analysis & Checkpoints

### 8.1 Defense in Depth
- **User Namespaces**: Enable them. Even if they break out, they are UID 1000 on host (not Root).
- **Read-Only Root**: `docker run --read-only`. Prevents attackers from downloading tools.
- **Seccomp/AppArmor**: Whitelist allowed syscalls. Block `mount`, `ptrace`, etc.

### 8.2 Checkpoint Questions
1.  **Scenario**: You run `ps aux` and see system processes (cron, systemd, etc.). Are you in a container?
    - *Answer*: Likely not (or a very poorly configured "System Container"). Standard Application Containers only see their own processes.
2.  **K8s**: Why is `CAP_NET_ADMIN` dangerous?
    - *Answer*: It allows interface modification. An attacker can set the interface to promiscuous mode, spoof packets, or manipulate iptables inside the namespace.
3.  **Forensics**: How do you analyze a running container without entering it?
    - *Answer*: From the host, find the PID (`docker inspect --format '{{.State.Pid}}' ID`). Then look at `/proc/[PID]/root/`. This is the container's filesystem view.

### 8.3 Final Thoughts
The line between "Host" and "Container" is thin—it is just a few kernel flags. Understanding these flags allows the Red Team to erase the line and the Blue Team to reinforce it.

---
*End of Chapter 07*
