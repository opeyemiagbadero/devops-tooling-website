
## Requirements
spun 5 EC2 servers, (1 for NFS, 3 for the webservers and 1 for the database server)


## Step 1 – NFS Server configuration.

Launched a new EC2 instance with RHEL Linux 8 Operating System, created 3 volumes in the same AZ (us-east-1b) as the NFS server each of 10GiB.

![1](https://user-images.githubusercontent.com/79456052/178137683-ccaa5c91-7d42-4f7b-966a-777d866021af.png)

Attached all three volumes one by one to the NFS server EC2 instance

![2](https://user-images.githubusercontent.com/79456052/178137684-a25f7037-f289-4e7b-8f47-27bbf6138a9f.png)

Used lsblk command to inspect the block devices attached to the NFS server. 

![3](https://user-images.githubusercontent.com/79456052/178137814-5037dccd-dc24-4c4b-bf77-4bacd447f71b.png)

Installed lvm2 package on the server using sudo yum install lvm2. Ran a sudo lvmdiskscan command to check for available partitions.Used pvcreate utility to mark each of 
the 3 disks as physical volumes (PVs) to be used by LVM.Verified that the physical volume had been created successfully by running sudo pvs.
Used vgcreate utility to add all 3 PVs to a volume group (VG) and named the VG nfsdata-vg.Verified that VG had been created successfully by running *sudo vgs*
Used lvcreate utility to create 3 logical volumes. lv-opt lv-apps, and lv-logs Verified that the logical volumes had been created successfully by running *sudo lvs*

![4](https://user-images.githubusercontent.com/79456052/178141126-795a09ff-95a7-4886-9c48-39635db92761.png)

![5  7-12](https://user-images.githubusercontent.com/79456052/178141097-d4226584-2353-4fcb-b21d-f0fd6d4c842b.png)

Verified the entire setup and used mkfs.xfs to format the logical volumes with xfs filesystem

![6  format logical volumes](https://user-images.githubusercontent.com/79456052/178138771-beec657f-af68-4601-8012-6d131e017173.png)

Created mount points on /mnt directory for the logical volumes. 

![7  mount the logical volumes and install nfs](https://user-images.githubusercontent.com/79456052/178140504-3a25530c-4762-4488-9a08-5e10e102dbf2.png)

Installed NFS server, configured it to start on reboot and made sure it was up and running.

![8  Install nfs contd](https://user-images.githubusercontent.com/79456052/178140599-be50ceaf-df6b-44cb-a76d-4d76b231104a.png)


Set up permission to allow the webservers to read, write and execute files on NFS, configured access to NFS for clients within the same subnet i.e 172.31.80.0/20,  
checked the port used by NFS. 


![9  steps 5 and 6](https://user-images.githubusercontent.com/79456052/178140689-67436a6a-7f0d-4ceb-87fe-05cf2c36c694.png)

Opened the port used by NFS using the security group and the ports: TCP 111, UDP 111, UDP 2049 for NFS server to gain access to the client.

![10  nfs security groups](https://user-images.githubusercontent.com/79456052/178141041-20baac57-13cb-473c-9e17-66c331e84664.png)

## Step 2 – database Server configuration.

Installed mysql on the database server. 

![1 installMySQL server database, tooling, created a database user and name ut webacces](https://user-images.githubusercontent.com/79456052/178141284-e2f6308c-0f87-4a71-a29c-63f59dc6011b.png)

Created a database and database user named tooling and webaccess respectively. Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr.

![2 Grant permission to webaccess user on tooling db to do anything only from the webservers subnet cidr](https://user-images.githubusercontent.com/79456052/178141287-bc9be9da-eb05-46e6-bc35-dac638f8be12.png)

## Step 3 — Web Servers configuration
Launched a new EC2 instance with RHEL 8 Operating System, installed an NFS client, mounted the /var/www/ and targeted the NFS server's export for apps.
Verified that NFS was mounted successfuly by running df -h. Made sure that the changes persisted on the web server after reboot

![1  1-4](https://user-images.githubusercontent.com/79456052/178492958-6cefd1aa-75ed-4921-b2b6-fc27e6570623.png)

Installed Remi's repository, apache and PHP. Verified that apache files and directories are available on the web server in /var/www 
and on the NFS server in /mnt/apps. Located the log folder for Apache on the web server and mounted it to the NFS server's export for logs i.e /var/log/httpd and confirmed 
that the NFS was successfully mounted and the changes persisted on the web server.
![2  Install Apache and PHP](https://user-images.githubusercontent.com/79456052/178493906-abdc3317-7f5d-4dbb-898c-98e207e31deb.png)

Installed git, initialized and forked the tooling website's code to the webserver.

![3  Install git , init and fork the darey-io tooling git repo](https://user-images.githubusercontent.com/79456052/178497629-4f93ba8c-85ed-44e0-849a-59c1feafbafa.png)

Deployed the tooling webiste's code to the webserver, and ensured that the html folder from the repository was deployed to /var/www/html. Disabled SELinux using the 
*sudo setenforce 0* command. To make the changes permanent, I opened the /etc/sysconfig/selinux config file and set SELINUX=disabled and then restarted httpd


![5  item no 9 on the documentation](https://user-images.githubusercontent.com/79456052/178499314-eacf6048-2f30-43c5-8933-62f39dbd213c.png)


Updated the webiste's configuration to connect to the database, and applied tooling-db.sql script to the database using the 
command *mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql*
                                                                                         
![6  item 10 in documentation](https://user-images.githubusercontent.com/79456052/178501320-07da67c5-58ea-479e-955d-f42801e6fc34.png)

Installed mysql on my webserver

![7  my sql instation on the webserver](https://user-images.githubusercontent.com/79456052/178502415-42539822-2818-47b1-a42b-1f885fb1a5af.png)

opened the TCP port 80 on the web server
                                                                                         
![8  confiure the security group on my sql to allow traffic](https://user-images.githubusercontent.com/79456052/178502582-76b40e16-539d-48e6-8e17-acd748437313.png)


Opened the webiste in my browser http://18.208.184.78/login.php

![12  last page](https://user-images.githubusercontent.com/79456052/178502705-e2ec506e-1cc7-4cd9-ab5b-37f7a6a666df.png)

![13  last page after 12](https://user-images.githubusercontent.com/79456052/178502722-91978267-d4c4-4842-8c62-c785550ec586.png)

