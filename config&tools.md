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
[centos@ip-172-31-6-60 ~]$ sudo su     # switch to root account
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
## Initial EFS
EFS using NFS and only support Linux system, do not support Windows.
- Install the amazon-efs-utils package on all the instances
```shell
sudo yum install -y amazon-efs-utils
```
> Refer: https://docs.aws.amazon.com/efs/latest/ug/installing-amazon-efs-utils.html
- Config EFS security group to allow ingress/inbound from instance secuity group
```shell
Add EFS security group, type is NFS, source is from instance security group
```
- Attach the EFS file system to instance
```shell
mkdir /efs
sudo mount -t efs -o tls fs-12345678:/ /efs
```
> Refer: https://docs.aws.amazon.com/efs/latest/ug/mounting-fs-mount-helper.html

## Tools
### AWS Policy Generator
The AWS Policy Generator is a tool that enables you to create policies that control access to Amazon Web Services (AWS) products and resources.         
- Link: https://awspolicygen.s3.amazonaws.com/policygen.html      
- Note: when you use aws-cn arn in the Policy Generator tool, please change the **aws-cn** to **aws**, if not the tool will get error and the Generate Policy button is gray. You can generate the policy first then change it back to aws-cn manually. 
```json
{
  "Id": "Policy1642062401473",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1642062399939",
      "Action": [
        "sqs:SendMessage"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws-cn:sqs:cn-northwest-1:056301980852:Demo-S3-Notification",
      "Principal": "*"
    }
  ]
}

# The default policy, it is only allow owner(root) to access SQS resource.  
{
  "Version": "2008-10-17",
  "Id": "__default_policy_ID",
  "Statement": [
    {
      "Sid": "__owner_statement",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws-cn:iam::056301980852:root"
      },
      "Action": "SQS:*",
      "Resource": "arn:aws-cn:sqs:cn-northwest-1:056301980852:Demo-S3-Notification"
    }
  ]
}
```

### IAM Policy Simulator
Testing your IAM policies with the IAM policy simulator
- Refer: https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_testing-policies.html
- Portal: https://policysim.aws.amazon.com
### EC2 Metadata & Userdata
Metadata url only working on EC2 instance to get metadata on EC2, and userdata is to get bootstrap script
- URL: http://169.254.169.254/latest/meta-data
- URL: http://169.254.169.254/latest/user-data
```shell
[centos@ip-172-31-6-60 ~]$ curl http://169.254.169.254/latest/meta-data
ami-id
ami-launch-index
ami-manifest-path
block-device-mapping/
events/
hibernation/
hostname
identity-credentials/
instance-action
instance-id
instance-life-cycle
instance-type
local-hostname
local-ipv4
mac
metrics/
network/
placement/
profile
public-hostname
public-ipv4
public-keys/
reservation-id
security-groups

services/[centos@ip-172-31-6-60 ~]$ curl http://169.254.169.254/latest/user-data
#!/bin/bash
yum install nginx
systemctl start nginx
systemctl enable nginx
```
