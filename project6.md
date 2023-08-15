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

7. Use `pvcreate` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

`sudo pvcreate /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`

![Physical Volumes](./images/physical-volumes.jpg)

8. Verify that your Physical volume has been created successfully by running  

`sudo pvs`

![Physical Volumes Status](./images/physical-volumes-status.jpg)

9. Use `vgcreate` utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

`sudo vgcreate webdata-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`

![Volume Group](./images/volume-group.jpg)

10. Verify that your VG has been created successfully by running sudo vgs

`sudo vgs`

![Volume Group Status](./images/volume-group-status.jpg)

11. Use `lvcreate` utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs

`sudo lvcreate -L 14G -n apps-lv webdata-vg`

`sudo lvcreate -L 14G -n logs-lv webdata-vg`

![Logical Volumes](./images/logical-volumes.jpg)

12. Verify that your Logical Volume has been created successfully by running 

`sudo lvs`

![Logical Volumes Status](./images/logical-volumes-status.jpg)

13. Verify the entire setup

`sudo vgdisplay -v #view complete setup - VG, PV, and LV`

![Complete Setup Status1](./images/complete-setup1.jpg)

![Complete Setup Status2](./images/complete-setup2.jpg)

![Complete Setup Status3](./images/complete-setup3.jpg)

`sudo lsblk`

![lsblk status](./images/lsblk%20status.jpg)










