+++
title = "My VFIO Setup"
slug = "2022/vfio"
description = "My VFIO setup"

date = 2022-04-14
#updated = 2020-10-10

draft = true

[taxonomies]
tags = ["linux", "virtualization", "gaming"]
+++

# What is VFIO / GPU passthrough?


# BIOS
Enable VT-D and IOMMU

# IOMMU groups

```bash
for d in /sys/kernel/iommu_groups/*/devices/*; do
  n=${d#*/iommu_groups/*}; n=${n%%/*}
  printf 'IOMMU Group %s ' "$n"
  lspci -nns "${d##*/}"
done
```

```
IOMMU Group 14 06:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU106 [GeForce RTX 2060 Rev. A] [10de:1f08] (rev a1)
IOMMU Group 14 06:00.1 Audio device [0403]: NVIDIA Corporation TU106 High Definition Audio Controller [10de:10f9] (rev a1)
IOMMU Group 14 06:00.2 USB controller [0c03]: NVIDIA Corporation TU106 USB 3.1 Host Controller [10de:1ada] (rev a1)
IOMMU Group 14 06:00.3 Serial bus controller [0c80]: NVIDIA Corporation TU106 USB Type-C UCSI Controller [10de:1adb] (rev a1)
```

# Packages
```bash
sudo yum -y install @virtualization
sudo useradd -aG libvirt $(id -un)
```

# GRUB
`/etc/default/grub`

GRUB_CMDLINE_LINUX

`iommu=1 amd_iommu=on`

`rd.driver.pre=vfio-pci vfio-pci.ids=GPU_ID1,GPU_ID2`
`rd.driver.pre=vfio-pci vfio-pci.ids=10de:1f08,10de:10f9,10de:1ada,10de:1adbo`


Create `/etc/dracut.conf.d/10-vfio.conf` with content:

```
add_drivers+=" vfio_pci vfio vfio_iommu_type1 vfio_virqfd "
```

Rebuild GRUB and initrd
```bash
dracut -fv
grub2-mkconfig -o /etc/grub2-efi.cfg
```


# Nested virt
`/etc/modprobe.d/kvm.conf`

```
options kvm_amd nested=1
```

# Reboot
Screen should be black


Check drivers
```
lspci -nnv -d 10de:
```

Kernel driver in use: vfio-pci

# VM Creation
`Customize configuration before install`
Firmware: UEFI x86_64: /usr/share/edk2/ovmf/OVMF_CODE.secboot.fd

Add Hardware -> Device type -> CDROM device
NIC: Link state: disable

## CPUs
Topology: Manually set CPU topology: 4 threads

## Windows Installation

# GPU binding to VM
Add Hardware -> PCI Host Device

```xml
<features>
    <hyperv>
        <vendor_id state="on" value="whatever"/>
    </hyperv>
    <kvm>
        <hidden state="on"/>
    </kvm>
</features>
```


Edit `/etc/default/grub`, add to CMD_LINE `video=efifb:off`

```bash
grub2-mkconfig -o /etc/grub2-efi.cfg
```


# Evdev setup

```shell-session
[yann@fedora vfio]$ ll /dev/input/by-id/
total 0
lrwxrwxrwx. 1 root root 9 Apr  4 21:05 usb-1532_Razer_DeathAdder_V2_Pro_000000000000-event-if01 -> ../event4
lrwxrwxrwx. 1 root root 9 Apr  4 21:05 usb-1532_Razer_DeathAdder_V2_Pro_000000000000-event-mouse -> ../event2
lrwxrwxrwx. 1 root root 9 Apr  4 21:05 usb-1532_Razer_DeathAdder_V2_Pro_000000000000-if01-event-kbd -> ../event3
lrwxrwxrwx. 1 root root 9 Apr  4 21:05 usb-1532_Razer_DeathAdder_V2_Pro_000000000000-if02-event-kbd -> ../event5
lrwxrwxrwx. 1 root root 9 Apr  4 21:05 usb-1532_Razer_DeathAdder_V2_Pro_000000000000-mouse -> ../mouse0
lrwxrwxrwx. 1 root root 9 Apr  4 21:05 usb-Logitech_G413_Silver_Mechanical_Gaming_Keyboard_0C71366F3933-event-kbd -> ../event6
lrwxrwxrwx. 1 root root 9 Apr  4 21:05 usb-Logitech_G413_Silver_Mechanical_Gaming_Keyboard_0C71366F3933-if01-event-kbd -> ../event7
```

Check which one get event by running `cat`, and trying to type / move the mouse
```bash
sudo cat /dev/input/by-id/usb-Logitech_G413_Silver_Mechanical_Gaming_Keyboard_0C71366F3933-event-kbd
sudo cat /dev/input/by-id/usb-1532_Razer_DeathAdder_V2_Pro_000000000000-event-mouse
```

Add to the XML:
```xml
<input type="evdev">
    <source dev="/dev/input/by-id/usb-Logitech_G413_Silver_Mechanical_Gaming_Keyboard_0C71366F3933-event-kbd" grab="all" repeat="on"/>
</input>
<input type="evdev">
    <source dev="/dev/input/by-id/usb-1532_Razer_DeathAdder_V2_Pro_000000000000-event-mouse"/>
</input>```

Need to stop the VM, then start again (restart doesn't work)



# Audio
TODO


# Sources & References
1. [Archlinux Wiiki - VFIO](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF) main VFIO resource
1. [How-to Guide - Single GPU Passthrough using Fedora 34](https://forum.level1techs.com/t/how-to-guide-single-gpu-passthrough-using-fedora-34/173677)
1. [Fedora 33 VFIO Guide](https://github.com/polkaulfield/Fedora-33-VFIO-guide)
1. [evdev](https://passthroughpo.st/using-evdev-passthrough-seamless-vm-input/)


