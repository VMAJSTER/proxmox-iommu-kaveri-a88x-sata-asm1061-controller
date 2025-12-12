# proxmox-iommu-kaveri-a88x-sata-asm1061-controller
Proxmox 8.4 IOMMU passtrought PCie controller to VM (same chip as integrated AHCI - known problem at Kavieri platform (specially A88X Asus Motherboards)

0. Enable IOMMU/Virtualization in BIOS menu options and do some undervolting ;)

dmesg | grep -e DMAR -e IOMMU
dmesg | grep 'remapping'
vi /etc/default/grub
update-grub
reboot


-------------------------------------------------------------------------------------------------------------------------------------------------------------
cat /etc/default/grub
(force vfio driver for specified ID of pci device + irqpoll (smoothswitch - network card problem after start VM with assigned PCIe device))

GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt vfio-pci.ids=1b21:0611 pcie_acs_override=downstream,multifunction pci=assign-busses,irqpoll"

#graphic-card
#GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt pcie_acs_override=downstream,multifunction nofb nomodeset video=vesafb:off,efifb:off"

#1st try
#GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt pcie_acs_override=downstream,multifunction"
#GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt"
-------------------------------------------------------------------------------------------------------------------------------------------------------------

pvesh get /nodes/NAMEOFPVEFOREXAMPLEPVE/hardware/pci --pci-class-blacklist ""
lspci
lspci -nnk
vi /etc/modules

-------------------------------------------------------------------------------------------------------------------------------------------------------------
# /etc/modules: kernel modules to load at boot time.
#
# This file contains the names of kernel modules that should be loaded
# at boot time, one per line. Lines beginning with "#" are ignored.
# Parameters can be specified after the module name.

vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
-------------------------------------------------------------------------------------------------------------------------------------------------------------

lspci -nn | grep 1b21:1060

find /sys/kernel/iommu_groups/ -type l
vi /etc/modprobe.d/vfio.conf
echo "options vfio-pci ids=1002:6649,1002:aac0 disable_vga=1" > /etc/modprobe.d/vfio.conf

-------------------------------------------------------------------------------------------------------------------------------------------------------------
cat /etc/modprobe.d/vfio.conf

options vfio-pci ids=1b21:0611
softdep ahci pre: vfio-pci

1. dodatkowe określenie sterownika dla urządzenia PCI o podanym ID
2. próba softzaładowania sterownika vfio-pci przed defaultowym AHCI (raczej nie działa / niepotrzebne, ale zostawiłem)
-------------------------------------------------------------------------------------------------------------------------------------------------------------


lspci -nnk | grep -A3 02:00.0

update-initramfs -u
update-grub
reboot
missed 3 hours
