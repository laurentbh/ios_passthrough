dumping notes to have ios passthrough to qEMU
- archlinux distrib
- NUC8 machine

**warning** unbind doesn't work !!

# boot kernel with io mmu 
`intel_iommu=on iommu=pt`
# find controller
`lspci | grep USB`
```
00:14.0 USB controller: Intel Corporation Cannon Point-LP USB 3.1 xHCI Controller (rev 30)
6c:00.0 USB controller: Intel Corporation JHL6340 Thunderbolt 3 USB 3.1 Controller (C step) [Alpine Ridge 2C 2016] (rev 02)
```

# find IOMMU group
`find /sys/kernel/iommu_groups/ -type l | grep 6c`
```
/sys/kernel/iommu_groups/16/devices/0000:6c:00.0
```
> group 16


create `/usr/bin/vfio-bind` and chmoded it 755
``` bash
#!/bin/bash
modprobe vfio-pci
for dev in "$@"; do
        vendor=$(cat /sys/bus/pci/devices/$dev/vendor)
        device=$(cat /sys/bus/pci/devices/$dev/device)
        if [ -e /sys/bus/pci/devices/$dev/driver ]; then
                echo $dev > /sys/bus/pci/devices/$dev/driver/unbind
        fi
        echo $vendor $device > /sys/bus/pci/drivers/vfio-pci/new_id
done
```

# find driver
`lspci -v`
- add before launching qemu

`/usr/bin/vfio-bind 0000:6c:00.0`

- add after qemu returns

`echo 0000:6c:00.0 > /sys/bus/pci/drivers/vfio-pci/unbind`
`echo 0000:6c:00.0 > /sys/bus/pci/drivers/xhci-hcd/bind`


