# devops.tooling.website.solution
Build a comprehensive devops tooling website solution.

You will implement a tooling website solutio which makes access to DevOps tools within the corporate infrastructure easily accesisble. 

This project implements the following components:

  - Infrastructure: AWS
  - Webserver Linux: Red Hat ENterprise Linux 8
  - Database Server: Ubuntu 20.04 + MySQL
  - Storage Server: Red Hat Enterprise Linux 8 + NFS Server
  - Programming Language
  - Code Repository: GitHub

A 3-tier architecture.
Webservers treat the the storage and databse units as local systems to retrieve data from:
![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/1.three-tierArchitecture.png?raw=true)

Storage types:
NFS, Block and Object storage.

How do you know which storage system to use depends on:
  - Data type
  - Format type
  - Data access
  - Who/what will access the data
  - From where will the data be accessed
  - How frequently will the data be accessed

## Implementing a business website using NFS for the backend file

Create a webserver + 3 EBS volumes: See LVM project for instrcutions

Create 3 logical volumes, names lv-opt, lv-apps and lv-logs

    sudo lvcreate -n lv-apps -L 9G webdata-vg
    sudo lvcreate -n lv-opt -L 9G webdata-vg
    sudo lvcreate -n lv-logs -L 9G webdata-vg


![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/vgdisplayofAll3webseversAFTERlvcreate.png?raw=true)

Validate setup of VG, PG and LV:

    sudo vgdisplay -v
    sudo lsblk

Format the LVs to xfs:

    sudo mkfs -t xfs /dev/webdata-vg/lv-apps
    sudo mkfs -t xfs /dev/webdata-vg/lv-opt
    sudo mkfs -t xfs /dev/webdata-vg/lv-logs

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/sudomkfs.xfs.png?raw=true)

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/formatXFSsuccess.png?raw=true)

After formatting, you can mount the XFS-formatted logical volume to a directory to start using it. 

    sudo mkdir /mnt/apps
    sudo mkdir /mnt/opt
    sudo mkdir /mnt/logs

Create mount points on /mnt directory for the logical volumes as follows: 
- Mount lv-apps on /mnt/apps - To be used by webservers 
- Mount Iv-logs on /mnt/logs - To be used by webserver logs
- Mount Iv-opt on /mnt/opt - To be used by Jenkins server in Project 8

      sudo mount /dev/webdata-vg/lv-apps /mnt/apps
      sudo mount /dev/webdata-vg/lv-opt /mnt/opt
      sudo mount /dev/webdata-vg/lv-logs /mnt/logs

### Create recovery logs backups:

Create /home/recovery/logs to store backup of log data:

    $ sudo mkdir -p /home/recovery/logs

The mount command tells us what storage devices are mounted on our system:

    $ mount  | grep xvd

We can use grep to narrow down the output for our devices

Backup all the files in the /var/log directory into the /home/recovery/logs

    $ sudo rsync -av /var/log/. /home/recovery/logs/

Mount the /mnt/logs directory on the lv-logs logical volume:

    $ sudo mount /dev/webdata-vg/lv-logs /mnt/logs
    $ sudo systemctl daemon-reload

Restore log files back into /var/log directory:

    $ sudo rsync -av /home/recovery/logs/. /mnt/logs

Update /etc/fstab. This persists the mount configuration even after restart.

    $ sudo blkid 

Copy the UUIDs of the mapper files.

Edit /etc/fstab as follows in this format:

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/fstab.png?raw=true)

Run this to validate changes:

    df -h 

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/df-h_AFTERmounting_all_3.png?raw=true)

Run this to test the configuration. Nothing returned = success:

    $ sudo mount -a

Updated listed blockl devices configuration:

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/lsblk2.png?raw=true)

### Setup NFS Server

    sudo yum -y update
    sudo yum install nfs-utils -y
    sudo systemctl start nfs-server.service
    sudo systemctl enable nfs-server.service
    sudo systemctl status nfs-server.service


Fix the permissions. This will allow the WebSErver to read, write, execute files on NFS:

    sudo chown -R nobody: /mnt/apps
    sudo chown -R nobody: /mnt/logs
    sudo chown -R nobody: /mnt/opt
    
    sudo chmod -R 777 /mnt/apps
    sudo chmod -R 777 /mnt/logs
    sudo chmod -R 777 /mnt/opt
    
    sudo systemctl restart nfs-server.service

Edit the /ect/exports text file to configure the files systems by adding the /32 subnet to each mount point as below:

    sudo vi /etc/exports

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/etc.Exports.png?raw=true)

Export the the list of file system (/etc/exports) to allow webserver clients to access NFS.  
![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/exportfs*.png?raw=true)

Then 

    sudo exportfs -arv

Check NFS ports and configure Security Groups by adding a new Inbound Rule:

    rpcinfo -p | grep nfs

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/rpcinfo-p.png?raw=true)

Inbound rules:
![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/inboundRules.png?raw=true)



DB Server storage setup:

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/DBserverBlockSetup.png?raw=true)

Setup of MySQL on DB server:

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/MySQLstatus.png?raw=true)

MySQL database setup on DB server:

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/mysqlToolingSetup.png?raw=true)


DB security group. The SG allows inbound access from the WebServers private IP. They are both on the same VPC so can connect to each others private IP:

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/DBsecuritygroup.png?raw=true)

Mount NFS

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/mountNFS_ws2.png?raw=true)

Edit /etc/fstab to persist NFS server info in the Webserver even after rebott:

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/WS_persistNFSIP_fstab.png?raw=true)

## Setup the Additional Webservers (3 in total, not including NFS server)

Create another EC2 instance with RHEL.

Install the NFS client:

    sudo yum install nfs-utils nfs4-acl-tools -y

Mount /var/www/ and target the NFS server's export for apps

    sudo mkdir /var/www
    sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www

Verifu successful NFS mount:

    df -h

  df -h on all 3 webservers showing mounted files:

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/all3webservers.df-h.png?raw=true)

To ensure this persists after reboot, edit /ect/fstab as follows:

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/fstabWebserver2.png?raw=true)

Enter the following into the WebServer2:

    sudo yum install httpd -y
    sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
    sudo yum install -y yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
    sudo yum module reset php
    sudo yum module enable php:remi-7.4
    sudo yum install php php-opcache php-gd php-curl php-mysqlnd
    sudo systemctl start php-fpm
    sudo systemctl enable php-fpm
    sudo setsebool -P httpd_execmem 1

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/php-enable-systemctl.png?raw=true)

All 3 webserver with php-fpm enabled:

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/php-fpm_onall3servers.png?raw=true)

All three webservers should now be in sync with the NFS server. To test this, create a est file in one of the 3 webservers and then check the the same file location in the NFS server as well as the other webservers:

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/sudoTouchtest.png?raw=true)

Success!

In each webserver, install git:

    sudo yum install -y git-all
    
Now, fork this repo into one of the Webservers:

    https://github.com/darey-io/tooling

Copy the html file in it to /var/www/html

    cd dareytooling
    sudo cp ./html /var/www/html/

Check apache is running:

    sudo systemctl status httpd

If it is not, restart it:

    sudo systemctl enable httpd
    sudo systemctl start httpd
    sudo systemctl status httpd

If you are getting errors disable SElinux:

    sudo setenforce 0
    sudo vi /etc/sysconfig/selinux

    # set SELINUX=disabled

    sudo systemctl restart httpd
    sudo systemctl status httpd

Then check on the browser via the public IP:

    wget -qO- ifconfig.me

Browser to show httpd is running:

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/httpdBrpwser.png?raw=true)

Login page for one of the 3 webservers:

![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/Screenshot%202023-12-07%20at%2015.17.53.png?raw=true)

