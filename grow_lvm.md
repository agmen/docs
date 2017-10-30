Grow disk

# rescan disks
echo 1 > /sys/block/sda/device/rescan

Using the whole secondary hard disk for LVM partition: 
`fdisk /dev/hdb`
```
At the Linux fdisk command prompt,
press n to create a new disk partition,
press p to create a primary disk partition,
press 1 to denote it as 1st disk partition,
press ENTER twice to accept the default of 1st and last cylinder – to convert the whole secondary hard disk to a single disk partition,
press t (will automatically select the only partition – partition 1) to change the default Linux partition type (0x83) to LVM partition type (0x8e),
press L to list all the currently supported partition type,
press 8e (as per the L listing) to change partition 1 to 8e, i.e. Linux LVM partition type,
press p to display the secondary hard disk partition setup. Please take note that the first partition is denoted as /dev/hdb1 in Linux,
press w to write the partition table and exit fdisk upon completion.
```

# rescan partitions
`partprobe`

# create new volume
`pvcreate /dev/sda3`
# extend it into the volume group
`vgextend rootvg /dev/sda3`
# extend the logical volume in to the new space
`lvextend -L +10G /dev/rootvg/rootlv`
or
`lvextend -l +100%FREE  /dev/rootvg/rootlv`
# resize the filesystem in the LV
`resize2fs -p /dev/rootvg/rootlv`
