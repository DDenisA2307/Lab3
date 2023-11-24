```
[root@Lab3mdadmin ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Thu Nov 16 07:51:44 2023
        Raid Level : raid6
        Array Size : 761856 (744.00 MiB 780.14 MB)
     Used Dev Size : 253952 (248.00 MiB 260.05 MB)
      Raid Devices : 5
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Thu Nov 16 11:40:18 2023
             State : clean 
    Active Devices : 5
   Working Devices : 5
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : Lab3mdadmin:0  (local to host Lab3mdadmin)
              UUID : 1832a96e:5905f635:c9bd96a7:c00f9916
            Events : 39

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       5       8       64        3      active sync   /dev/sde
       4       8       80        4      active sync   /dev/sdf


[root@Lab3mdadmin ~]# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid6 sdc[1] sdb[0] sdd[2] sde[5] sdf[4]
      761856 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/5] [UUUUU]
      
unused devices: <none>

[root@Lab3mdadmin ~]# mdadm -S /dev/md0
mdadm: stopped /dev/md0
[root@Lab3mdadmin ~]# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] 
unused devices: <none>

[root@Lab3mdadmin ~]# mdadm --zero-superblock /dev/sdb
[root@Lab3mdadmin ~]# mdadm --zero-superblock /dev/sdc
[root@Lab3mdadmin ~]# mdadm --zero-superblock /dev/sdd
[root@Lab3mdadmin ~]# mdadm --zero-superblock /dev/sde
[root@Lab3mdadmin ~]# mdadm --zero-superblock /dev/sdf

[root@Lab3mdadmin ~]# mdadm --create --verbose /dev/md0 -l 1 -n 5 /dev/sd{b,c,d,e,f}
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: size set to 254976K
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.


[root@Lab3mdadmin ~]# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] [raid1] 
md0 : active raid1 sdf[4] sde[3] sdd[2] sdc[1] sdb[0]
      254976 blocks super 1.2 [5/5] [UUUUU]
      
unused devices: <none>


[root@Lab3mdadmin ~]# mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Nov 22 04:36:46 2023
        Raid Level : raid1
        Array Size : 254976 (249.00 MiB 261.10 MB)
     Used Dev Size : 254976 (249.00 MiB 261.10 MB)
      Raid Devices : 5
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Wed Nov 22 04:36:59 2023
             State : clean 
    Active Devices : 5
   Working Devices : 5
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : resync

              Name : Lab3mdadmin:0  (local to host Lab3mdadmin)
              UUID : f6c6d846:8b4cdd4a:ccc3e7e3:b925ddc6
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde
       4       8       80        4      active sync   /dev/sdf


[root@Lab3mdadmin ~]# echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
[root@Lab3mdadmin ~]# mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >>  /etc/mdadm/mdadm.conf
[root@Lab3mdadmin ~]# cat /etc/mdadm/mdadm.conf
DEVICE partitions
ARRAY /dev/md0 level=raid1 num-devices=5 metadata=1.2 name=Lab3mdadmin:0 UUID=f6c6d846:8b4cdd4a:ccc3e7e3:b925ddc6


[root@Lab3mdadmin ~]# parted -s /dev/md0 mklabel gpt
[root@Lab3mdadmin ~]# parted /dev/md0 mkpart primary ext4 0% 20%
Information: You may need to update /etc/fstab.

[root@Lab3mdadmin ~]# parted /dev/md0 mkpart primary ext4 20% 40%         
Information: You may need to update /etc/fstab.

[root@Lab3mdadmin ~]# parted /dev/md0 mkpart primary ext4 40% 60%         
Information: You may need to update /etc/fstab.

[root@Lab3mdadmin ~]# parted /dev/md0 mkpart primary ext4 60% 80%         
Information: You may need to update /etc/fstab.

[root@Lab3mdadmin ~]# parted /dev/md0 mkpart primary ext4 80% 100%        
Information: You may need to update /etc/fstab.


[root@Lab3mdadmin ~]# for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
Stride=0 blocks, Stripe width=0 blocks
12544 inodes, 50176 blocks
2508 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=33685504
7 block groups
8192 blocks per group, 8192 fragments per group
1792 inodes per group
Superblock backups stored on blocks: 
	8193, 24577, 40961

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
Stride=0 blocks, Stripe width=0 blocks
12824 inodes, 51200 blocks
2560 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=33685504
7 block groups
8192 blocks per group, 8192 fragments per group
1832 inodes per group
Superblock backups stored on blocks: 
	8193, 24577, 40961

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
Stride=0 blocks, Stripe width=0 blocks
12544 inodes, 50176 blocks
2508 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=33685504
7 block groups
8192 blocks per group, 8192 fragments per group
1792 inodes per group
Superblock backups stored on blocks: 
	8193, 24577, 40961

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
Stride=0 blocks, Stripe width=0 blocks
12824 inodes, 51200 blocks
2560 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=33685504
7 block groups
8192 blocks per group, 8192 fragments per group
1832 inodes per group
Superblock backups stored on blocks: 
	8193, 24577, 40961

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
Stride=0 blocks, Stripe width=0 blocks
12544 inodes, 50176 blocks
2508 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=33685504
7 block groups
8192 blocks per group, 8192 fragments per group
1792 inodes per group
Superblock backups stored on blocks: 
	8193, 24577, 40961

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

[root@Lab3mdadmin ~]# mkdir -p /raid/part{1,2,3,4,5}
[root@Lab3mdadmin ~]# for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done
[root@Lab3mdadmin ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        489M     0  489M   0% /dev
tmpfs           496M     0  496M   0% /dev/shm
tmpfs           496M  6.7M  489M   2% /run
tmpfs           496M     0  496M   0% /sys/fs/cgroup
/dev/sda1        40G  4.4G   36G  11% /
tmpfs           100M     0  100M   0% /run/user/1000
/dev/md0p1       44M  1.1M   40M   3% /raid/part1
/dev/md0p2       45M  1.1M   40M   3% /raid/part2
/dev/md0p3       44M  1.1M   40M   3% /raid/part3
/dev/md0p4       45M  1.1M   40M   3% /raid/part4
/dev/md0p5       44M  1.1M   40M   3% /raid/part5
```
