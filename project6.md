# Kingsley Documentation of Project 6

## Web Solution With WordPress

*In this project i am tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress*

### Step 1

1. Launch an EC2 instance that will serve as “Web Server”. Create 3 volumes in the same AZ as your Project6-Webserver EC2, each of 10 GB

![EC2 Instances](./images/ec2-instances.jpg)

![Volumes Created](./images/volumes-created.jpg)

2. Attach all three volumes one by one to Project6-Webserver EC2 instance

3. Use `lsblk` command to inspect what block devices are attached to the server.

![Volumes Attached](./images/block-volumes-attached.jpg)

4. Use `df -h` command to see all mounts and free space on your server

![Server Free Space And Mounts](./images/server-mounts-freespace.jpg)

5. Use gdisk utility to create a single partition on each of the 3 disks

`sudo gdisk /dev/nvme1n1`

![Disk 1 Partition](./images/disk-partition1a.jpg)

![Disk 1 Partition](./images/disk-partition1b.jpg)

`sudo gdisk /dev/nvme2n1`

![Disk 2 Partition](./images/disk-partition2a.jpg)

![Disk 2 Partition](./images/disk-partition2b.jpg)

`sudo gdisk /dev/nvme3n1`

![Disk 3 Partition](./images/disk-partition3a.jpg)

![Disk 3 Partition](./images/disk-partition3b.jpg)

5. Use `lsblk` utility to view the newly configured partition on each of the 3 disks.

![Partition Status](./images/partition-status.jpg)

6. Install lvm2 package using `sudo yum install lvm2` . Run `sudo lvmdiskscan` command to check for available partitions.

![LVM2 Package Install Status](./images/lvm2-package-status.jpg)

`sudo lvmdiskscan`

![Available Partitions](./images/partition-scan.jpg)




