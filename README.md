# How to mount the Amazon Elastic Filesystem and share the filesystem with Multiple Windows EC2 instances using SAMBA?

To share an AWS EFS (Elastic File System) between a Windows EC2 instance and a Linux EC2 instance using Samba, we can follow these step-by-step instructions. Since EFS natively supports NFS and not SMB (which Windows uses), we'll set up a Linux instance as a bridge using Samba to make the EFS accessible to Windows.

## Prerequisites:

An AWS EFS file system created in your VPC.

A Linux EC2 instance and a Windows EC2 instance running in the same VPC.

Proper security group settings allowing NFS, SMB, and SSH traffic between the instances.

![image](https://github.com/user-attachments/assets/367ad309-2b16-4abf-b0be-aa355f111500)

## Install Botocore to Resolve Mount Target

```
sudo yum install -y python3-pip
sudo pip3 install botocore
python3 -m pip show botocore
```

## Mount EFS on the Linux EC2 Instance

```
sudo yum install -y amazon-efs-utils
sudo mkdir /mnt/efs
sudo mount -t efs fs-0123456789.efs.ap-south-1.amazonaws.com:/ /mnt/efs
df -h
```

## Install Samba on the Linux Instance

```
sudo yum update -y
sudo yum install -y samba samba-common samba-client
sudo mkdir -p /mnt/efs/shared
sudo chmod 777 /mnt/efs/shared
```

## Configure Samba

_Edit the Samba configuration file (/etc/samba/smb.conf)_

```
sudo vi /etc/samba/smb.conf

_//Add the following configuration at the bottom of the file_

[shared]
path = /mnt/efs/shared
available = yes
valid users = smbuser
read only = no
browsable = yes
public = yes
writable = yes
create mask = 0777
directory mask = 0777

_Create a Samba user:_

sudo useradd smbuser
sudo smbpasswd -a smbuser

_Restart the Samba service:_

sudo systemctl restart smb
sudo systemctl enable smb
```

## Automate EFS Mount on Boot

If you want to ensure that the EFS mounts automatically on boot, add an entry to /etc/fstab on the Linux EC2 instance

```
sudo vi /etc/fstab

fs-0123456789.efs.ap-south-1.amazonaws.com:/ /mnt/efs efs _netdev 0 0

sudo umount /mnt/efs
sudo mount -a
```

![image](https://github.com/user-attachments/assets/7a3b4447-9024-4e86-98b1-174fe6edb858)

## Access the Shared Folder from the Windows EC2 Instance

Log into your Windows EC2 instance.

Open File Explorer and navigate to the Samba share by entering the Linux instance's private IP and the shared folder name

![image](https://github.com/user-attachments/assets/61d3df93-aa57-40b4-bfed-252bb12abb2a)

```
\\<Linux-EC2-Private-IP>\shared
```

![image](https://github.com/user-attachments/assets/0cf98d69-2348-43e4-9cc9-7a2c3c36ef61)









