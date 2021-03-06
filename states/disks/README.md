
#### Disks
- Disk partition info:
  - `fdisk -l /dev/sda` 
  - `parted -l /dev/sda`  
- Disk info: `hdparm -iI /dev/sda`
- Disk speed measurement: `hdparm -tT /dev/sda`  
- DD write (cache): `dd if=/dev/zero of=/tmp/output.dat bs=1MB count=100`  
- DD write (synchronous IO): `dd if=/dev/zero of=/tmp/output.dat bs=1MB count=100 conv=fsync`  
- DD read:  `dd if=/tmp/output.dat of=/dev/null bs=4096k`  
- Linux md devices (aka RAID): `mdadm`
- Destroy data: `dd if=/dev/zero of=/dev/sdb bs=4096 count=512`

##### Block devices
```
lsblk
blkid
```

##### parted

```
parted /dev/sdb mklabel gpt -s
parted /dev/sdb mkpart primary 0% 100% -s
parted /dev/sdb align-check optimal 1

mke2fs -t ext4 -O ^has_journal /dev/sdb1
tune2fs -O ^has_journal /dev/sdb1
```

```
[root@workstation2 ~]# salt 'ring-a1' partition.mklabel /dev/sdb gpt
ring-a1:
[root@workstation2 ~]#  salt 'ring-a1' partition.mkpart /dev/sdb primary fs_type=ext2 start=2048s end=100%
ring-a1:
[root@workstation2 ~]#  salt 'ring-a1' extfs.mkfs /dev/sdb1 fs_type=ext4 
```

##### LVM:
```
pvdisplay
lvdisplay
lvmdiskscan
```

### dd

- Use fdatasync attribute to show the status rate only after the data is completely written to disk.
```
[root@workstation2 mnt]# dd if=/dev/zero of=test1.dat bs=1MB count=100
100+0 records in
100+0 records out
100000000 bytes (100 MB) copied, 0.0998435 s, 1.0 GB/s
[root@workstation2 mnt]# dd if=/dev/zero of=test2.dat bs=1MB count=100 conv=fdatasync
100+0 records in
100+0 records out
100000000 bytes (100 MB) copied, 11.1252 s, 9.0 MB/s
[root@workstation2 mnt]# 
```

##### exFAT

Mount a USB drive

- Add support for exfat FS: `yum install fuse-exfat exfat-utils`
- mount the partition as exfat: `mount -t exfat /dev/sdb1 /mnt`

- Example: Using parted to create an exFAT filesystem on a microSD card for a GoPro Hero3
```
dd if=/dev/zero of=/dev/sdb bs=4096 count=512

parted /dev/sdb mklabel msdos
parted /dev/sdb mkpart primary fat32 0% 100%
parted /dev/sdb print free
parted /dev/sdb align-check opt 1
/sbin/mkfs -t fat /dev/sdb1

mount /dev/sdb1 /mnt

```

- Example: Using parted to create an ext4 filesystem on a USB stick:
```
parted -l
dd if=/dev/zero of=/dev/sdb bs=4096 count=512

parted /dev/sdb mklabel gpt
parted /dev/sdb mkpart primary ext4 0% 100%
parted /dev/sdb print free
parted /dev/sdb align-check opt 1

cat /proc/partitions

/sbin/mkfs -t ext4 /dev/sdb1

mount -t ext4 /dev/sdb1 /mnt

ls -l /mnt
df

findmnt /dev/sdb1
```

Disk Performance:  `dd if=/dev/zero of=test2.dat bs=1MB count=100 conv=fdatasync`

| Device | Type | Speed | Interface |
|--------|------|-------|-----------|
| MicroSD c10 | Flash |  10 MB/s | USB 2.0|
| Hitachi HCC54755 | HDD | 80 MB/s | SATA |
| INTEL SSDSC2BB30 | SSD | 275 MB/s | SATA |

#### References
- [Disk Caching](http://www.linuxatemyram.com/index.html)
- [http://blackbird.si/tips-for-optimizing-disk-performance-on-linux](http://blackbird.si/tips-for-optimizing-disk-performance-on-linux/)
- [http://linuxmantra.com/2013/11/disk-read-ahead-in-linux.html](http://linuxmantra.com/2013/11/disk-read-ahead-in-linux.html)
