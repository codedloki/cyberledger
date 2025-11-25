---
title: Setup QEMU/KVM on Arch Linux
date: 2025-11-18
tags:
  - arch
  - linux
  - archlinux
  - qemu
  - kvm
  - vm
  - virtual
  - setups
---
![qemuface](/images/qemu_kvm_setup/qemuarch.jpg)


## INTRODUCTION

Virtualization on Arch Linux often feels confusing, especially for users coming from VirtualBox or VMware.  
These tools are great for beginners, but they consume a lot of system resources and can slow down the overall experience.  

In this guide, we will use **QEMU with KVM acceleration**, a faster and more efficient virtualization solution built directly into the Linux kernel.  
It gives you better performance, lower resource usage, and a smoother workflow on Arch Linux.


## System Requirements

1. Virtualization Enabled in BIOS
2. Stable Internet Connection
3. Basic Terminal Settings

## Installation guide


1. Execute Below command 

```shell
sudo pacman -S qemu-full qemu-img libvirt virt-install virt-manager virt-viewer \
edk2-ovmf dnsmasq swtpm guestfs-tools libosinfo tuned

```
2. This are services required to  be running  to run the qemu  but recommend instead of enabling them which will start them when the system boot increasing the resource usage i suggest just start them when u require them 
___________________________________________________________________________________________________

 - 1. Option : Enable modular daemon
```shell
for drv in qemu interface network nodedev nwfilter secret storage; do
    sudo systemctl restart virt${drv}d.service;
    sudo systemctl restart virt${drv}d{,-ro,-admin}.socket;
done
```

 - 2. Option : Enable monolithic daemon

 ```shell
sudo systemctl enable libvirtd.service
```
______________________________________________________________________________________________________

1. if you want we can enable nested  nested virtualization its based on your use case (**optional**)

		1. Temporary session
	*For Intel*
```shell
  sudo modprobe -r kvm_intel
  sudo modprobe kvm_intel nested=1
```
		 
*for AMD*
```shell
	sudo modprobe -r kvm_amd
	sudo modprobe kvm_amd nested=1
```
2. Permanent Session 
		
*For Intel*
```shell
	echo "options kvm_intel nested=1" | sudo tee /etc/modprobe.d/kvm-intel.conf
```
*For AMD*
		
```shell
	echo "options kvm_amd nested=1" | sudo tee /etc/modprobe.d/kvm-amd.conf
```
# ---------------------------------------------------------------------------------------------------------------
###  Enabling IMMOMU  (FOR INTEL )

IOMMU = Input–Output Memory Management Unit

Think of it like a **traffic controller** for hardware devices.

It controls how PCI devices (GPU, USB controllers, NVMe disks, etc.) access system memory.
without it:

- You **cannot assign hardware directly** to a virtual machine
    
- GPU passthrough will not work
    
- Device isolation becomes weak
    
- Security decreases


1.  Open your GRUB config

```shell
sudo vim /etc/default/grub	
```


2. Add following kernel module entries

```shell
#/etc/default/grub 
GRUB_CMDLINE_LINUX="... intel_iommu=on iommu=pt"
```

3. Regenerate the grub config 
   
```shell
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

3. Set Profile to  `virtual-host` for tuned 


```shell
sudo tuned-adm profile virtual-host
```
# ------------------------------------------------------------------------------------------------------------

### ENABLE SEV (FOR AMD)

#### Option 1 : ENABLE SEV IN AMD  using modproble
_________________________________________________________

```shell
echo "options kvm_amd sev=1" | sudo tee /etc/modprobe.d/amd-sev.conf
sudo reboot now
```
__________________________________________________________

#### Option 2 : ENABLE SEV IN AMD USING GRUB

___________________________________________________________

  1. Open your grub config file 

  ```shell
    sudo vim /etc/default/grub
  ```
  2. Add following lines to the file 

  ```shell
    GRUB_CMDLINE_LINUX="... mem_encrypt=on kvm_amd.sev=1"
  ``` 

  3.Regenerate the grub config


  ```shell
    sudo grub-mkconfig -o /boot/grub/grub.cfg
    sudo reboot now
  ``` 
# -------------------------------------------------------------------------------------------------------------------

### OPTIMIZING HOST WITH TUNED

1. Enable TuneD at startup 

```shell
sudo systemctl enable --now tuned.service
```

2. Check Active profile

```shell
tuned-adm active
```


3. Set profile to `virtual-host`


```shell
sudo tuned-adm profile virtual-host
```

