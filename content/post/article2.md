---
title: "Writing a simple Linux Kernel-1"
date: 2021-04-14T00:05:06-04:00
author: "ozan ocak"
tags: [
    "aarch64",
    "docker",
    "linux",
]
series : ["kernel development"]
---
<h3>Setting up aarch64 platform within docker container on x86 architecture</h3>

Docker Desktop for Mac ships with hypervisors for the host OS.  The hypervisor is responsible for running a lightweight Linux kernel (LinuxKit), which is included as part of Docker Desktop.  This lightweight container OS comes packaged with the QEMU emulator, and comes pre-configured with binfmt_misc to run binaries of any supported architecture. Binfmt_misc is a kernel subsystem which allows to set the magic numbers of a binary file to specific actions for their execution. 

![arm64 docker on x86](/img/docker_emu.png) 

```bash
docker run --rm -t arm64v8/ubuntu uname -m
docker container ls
docker exec -it <container_id> /bin/bash
```
When we run docker image, docker cli pull the img than we execute it in bash terminal of the container.

```bash
uname -a
Linux dfe4f9696511 4.19.121-linuxkit SMP Tue Dec 1 17:50:32 UTC 2020 aarch64 GNU/Linux
```
We can see linuxkit as the  running operating sysyem of the container. Now that we create the enviroment for little kernel, we can install cross compiler(gcc) and a text editor. 

```bash
apt update
apt install gcc-aarch64-linux-gnu
apt install vim
```
