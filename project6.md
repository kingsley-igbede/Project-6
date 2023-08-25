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

14. Use mkfs.ext4 to format the logical volumes with ext4 filesystem

`sudo mkfs.ext4 /dev/webdata-vg/apps-lv`

![filesystem apps-lv](./images/filesystem-apps-lv.jpg)

`sudo mkfs.ext4 /dev/webdata-vg/logs-lv`

![filesystem logs-lv](./images/filesystem-logs-lv.jpg)

15. Create /var/www/html directory to store website files

`sudo mkdir -p /var/www/html`

16. Create /home/recovery/logs to store backup of log data

`sudo mkdir -p /home/recovery/logs`

17. Mount apps-lv logical volume on /var/www/html  

`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

![Mount Status](./images/mount-status.jpg)

18. Use `rsync` utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

`sudo rsync -av /var/log/. /home/recovery/logs/`

19. Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 18 above is very important)

`sudo mount /dev/webdata-vg/logs-lv /var/log`

`sudo ls -l /var/log`

![Logs-lv Mount Status](./images/logs-lv-mount-status.jpg)

20. Restore log files back into /var/log directory

`sudo rsync -av /home/recovery/logs/ /var/log`

*temporary mount update*

![Temp Mount Status](./images/temp-mount-status.jpg)

21. Update /etc/fstab file so that the mount configuration will persist after restart of the server.

*The UUID of the device will be used to update the /etc/fstab file*

`sudo blkid`

![BLKID Status](./images/sudo-blkid-status.jpg)

`sudo vi /etc/fstab`

*Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes*

![FSTAB Edit](./images/fstab-edits.jpg)

22. Test the configuration and reload the daemon

`sudo mount -a`

`sudo systemctl daemon-reload`

23. Verify your setup by running `df -h`, output must look like this:

![Verify Status](./images/verify-status.jpg)


### Step 2 - Prepare the Database Server

1.  Launch an EC2 instance that will serve as “Database Server”. Create 3 volumes in the same AZ as your Project6-DBserver EC2, each of 10 GB

![Database Instance](./images/db-instance.jpg)

![DB Volumes](./images/db-volumes.jpg)

2. Attach all three volumes one by one to Project6-DBserver EC2 instance

3. Use `lsblk` command to inspect what block devices are attached to the server.

![DB Volumes Attached](./images/db-volumes-attached.jpg)

4.  Use `df -h` command to see all mounts and free space on your server

5. Use gdisk utility to create a single partition on each of the 3 disks

`sudo gdisk /dev/nvme1n1`

5. Use `lsblk` utility to view the newly configured partition on each of the 3 disks.

![DB Partition Status](./images/db-partition-status.jpg)

6. Install lvm2 package using `sudo yum install lvm2` . Run `sudo lvmdiskscan` command to check for available partitions.

![DB LVM2 Package Install Status](./images/DB-lvm2-status.jpg)

`sudo lvmdiskscan`

![DB LVM Disk Status](./images/db-lvm-disk-status.jpg)

7. Use `pvcreate` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

`sudo pvcreate /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`

![Db Physical Volumes](./images/db-physical-volumes.jpg)

8. Verify that your Physical volume has been created successfully by running  

`sudo pvs`

![Db PVS Status](./images/db-physical-volume-status.jpg)

9. Use `vgcreate` utility to add all 3 PVs to a volume group (VG). Name the VG database-vg

`sudo vgcreate database-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`

![DB Volume Group](./images/db-volume-group.jpg)

10. Verify that your VG has been created successfully by running sudo vgs

`sudo vgs`

![DB Volume Group Status](./images/db-volume-group-status.jpg)

11. Use `lvcreate` utility to create a db-lv logical volume. 

`sudo lvcreate -n db-lv -L 20G database-vg`

12. Verify that your Logical Volume has been created successfully by running

`sudo lvs`

![DB Logical Volumes Status](./images/db-logical-volume-status.jpg)

13. Verify the entire setup

`sudo vgdisplay -v #view complete setup - VG, PV, and LV`

![DB Complete Setup Status1](./images/db-complete-setup1.jpg)

![DB Complete Setup Status2](./images/db-complete-setup2.jpg)

`sudo lsblk`

![DB lsblk status](./images/db-lsblk-status.jpg)

14. Use mkfs.ext4 to format the logical volume with ext4 filesystem

`sudo mkfs.ext4 /dev/database-vg/db-lv`

![filesystem db-lv](./images/db-filesystem.jpg)

15. Create /db directory 

`sudo mkdir /db`

16. Mount /db on db-lv logical volume

`sudo mount /dev/database-vg/db-lv /db`

17. Check status using the code

`df -h`

![DB Mount Status](./images/db-mount-status.jpg)

18. Update /etc/fstab file so that the mount configuration will persist after restart of the server.

*The UUID of the device will be used to update the /etc/fstab file*

`sudo blkid`

![DB BLKID Status](./images/db-blkid-status.jpg)

`sudo vi /etc/fstab`

![DB FSTAB Edit](./images/db-fstab-edit.jpg)

19. Test the configuration and reload the daemon

`sudo mount -a`

`sudo systemctl daemon-reload`

20. Verify your setup by running `df -h`, output must look like this:

![DB Verify Status](./images/db-verify-setup.jpg)


### Step 3 — Install WordPress on Web Server EC2

1. Update the repository

`sudo yum update -y`

2. Install wget, Apache and it’s dependencies

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

![Apache and Dependencies](./images/apache-dependencies.jpg)

3. Start Apache

`sudo systemctl enable httpd`

`sudo systemctl start httpd`

![Start Apache Status](./images/start-apache-status.jpg)

4. To install PHP and it’s depemdencies

`sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

![EPEL Status](./images/epel-status.jpg)

`sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

![Remirepo Status](./images/remirepo-status.jpg)

`sudo yum module list php`

![Module List PHP](./images/module-list-php.jpg)

`sudo yum module reset php`

![Module Reset PHP](./images/module-reset-php.jpg)

`sudo yum module enable php:remi-7.4`

![Module Enable PHP](./images/module-enable-php.jpg)

`sudo yum install php php-opcache php-gd php-curl php-mysqlnd`

![PHP Modules](./images/php-modules.jpg)

`php -v`

![PHP Version](./images/php-version.jpg)

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

`sudo setsebool -P httpd_execmem 1`

`sudo systemctl status php-fpm`

![PHP FPM Status](./images/php-fpm-status.jpg)

5. Restart Apache

`sudo systemctl restart httpd`

6. Download wordpress and copy wordpress to var/www/html

`mkdir wordpress`

`cd wordpress`

`sudo wget http://wordpress.org/latest.tar.gz`

![Wordpress Download](./images/wordpress-download.jpg)

*Extract Wordpress*
`sudo tar xzvf latest.tar.gz`

![Wordpress Folder](./images/extra-wordpress-folder.jpg)

`sudo rm -rf latest.tar.gz`

`cp wordpress/wp-config-sample.php wordpress/wp-config.php`

`ls -l`

`cd wordpress`

`ls -l`

![Wordpress Folder Content](./images/wordpress-content.jpg)

`cp -R wordpress/. /var/www/html/`

![html status](./images/html-status.jpg)

7. Configure SELinux Policies

`sudo chown -R apache:apache /var/www/html/wordpress`

`sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R`

`sudo setsebool -P httpd_can_network_connect=1`

### Step 4 — Install MySQL on your DB Server EC2

`sudo yum update`

`sudo yum install mysql-server`

`sudo systemctl status mysqld`

![mysqld status](./images/mysqld-status.jpg)

`sudo systemctl restart mysqld`

`sudo systemctl enable mysqld`

`sudo systemctl status mysqld`

![mysqld status active](./images/mysqld-status-active.jpg)

### Step 5 — Configure DB to work with WordPress

`sudo mysql`

`CREATE DATABASE wordpress;`

`show databases;`

![databases status](./images/dbserver-databases-status.jpg)

`CREATE USER `myuser`@`172.31.47.102` IDENTIFIED BY 'mypass';`

`GRANT ALL ON wordpress.* TO 'myuser'@'172.31.47.102';`

`FLUSH PRIVILEGES;`

`SHOW DATABASES;`

`select user, host from mysql.user;`

![dbserver user-host sttus](./images/dbserver-user-host-status.jpg)

`exit`

### Step 6 — Configure WordPress to connect to remote database.

*Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32*

![dbserver security group](./images/dbserver-security-group.jpg)

1. Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client

`sudo yum install mysql`

`sudo mysql -u myuser -p -h 172.31.45.110`

![webserver mysql status](./images/webserver-mysql-status.jpg)

2. Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.

`show databases;`

![webserver dbcommand status](./images/webserver-dbcommand-status.jpg)

`exit`

3. Change permissions and configuration so Apache could use WordPress:

`sudo chown -R apache:apache /var/www/html/wordpress`

`sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R`

`sudo setsebool -P httpd_can_network_connect=1`

`sudo vi wp-config.php`

![edit apache](./images/edit-apache.jpg)

`sudo systemctl restart httpd`

4. Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

![webserver security policy status](./images/webserver-port80-security-policy.jpg)

5. Try to access from your browser the link to your WordPress

[wordpress site](http://13.51.172.159/wordpress/)


![wordpress site](./images/wordpress-site.jpg)

*Fill out your DB credentials:*

![wordpress db status](./images/wordpress-db-status.jpg)










