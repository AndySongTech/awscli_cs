## Initial EBS
- Check the ebs volume
```shell
[centos@ip-172-31-6-60 ~]$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk
`-xvda1 202:1    0   8G  0 part /
xvdb    202:16   0   2G  0 disk
```
- Format the volume and mount to instance
```shell
[root@ip-172-31-6-60 centos]# mkfs -t xfs /dev/xvdb      # format the volume to xfs type
meta-data=/dev/xvdb              isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

[root@ip-172-31-6-60 centos]# mkdir /data         # create mount point
[root@ip-172-31-6-60 centos]# mount /dev/xvdb /data    # mount the volume
[root@ip-172-31-6-60 data]# df -h    # validate the config
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        485M     0  485M   0% /dev
tmpfs           495M     0  495M   0% /dev/shm
tmpfs           495M  6.7M  489M   2% /run
tmpfs           495M     0  495M   0% /sys/fs/cgroup
/dev/xvda1      8.0G  1.6G  6.5G  19% /
tmpfs            99M     0   99M   0% /run/user/1000
/dev/xvdb       2.0G   33M  2.0G   2% /data
```
- Add the mount to /etc/fstab to mount automatically when the machine reboot
```shell
cat <<EOF >>/etc/fstab
/dev/xvdb /data xfs defaults 0 0
EOF
```


