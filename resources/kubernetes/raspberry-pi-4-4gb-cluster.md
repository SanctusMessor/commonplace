---
description: 'Random notes from setting up my cluster, I''ll clean this up later.'
---

# Raspberry Pi 4 \(4GB\) Cluster

**Ubuntu 19.10**  
Imaged with the [official Raspberry Pi imaging tool](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#2-prepare-the-sd-card)  
Configured wifi \(dhcp\)  
Configured eth0 with static IP \(via Router, NIC MAC is tied to IP lease\)  
Generated and Added SSH keys  
Disabled password sign-in  
Installed microk8s \(x4\)  
Enabled dns and metrics-server on master

After installing microk8s via snap, all nodes \(master included\) were stuck in **NotReady** status.

> what actually worked is editing `/boot/firmware/nobtcmd.txt`, appending `cgroup_enable=memory cgroup_memory=1`. and rebooting.



