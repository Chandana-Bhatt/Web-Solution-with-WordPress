# Web-Solution-with-WordPress

This project consists of two parts:
1.  Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience of working with disks, partitions and volumes in Linux.
2.  Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skills of deploying Web and DB tiers of Web solution.

This project employs a three tier architecture while also ensuring that the disks used to store files on the Linux servers are adequately partitioned and managed through programs such as gdisk and LVM respectively.

Side note : 
Three tier architecture -
  Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers.
  The 3 layers are:
  1. Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.
  2. Business Layer (BL): This is the backend program that implements business logic. Application or Webserver
  3. Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS Server.
  
  
# Initial setup

1. We start by creating two EC2 instances of the following specification:
   RedHat Enterprise Linux 9 (HVM)
   
2. Rename the two instances as web-server and db-server respectively.
   
3. We then configure the security groups to allow all the traffic(for both the servers) as shown :
 
    ![IMG-9320](https://user-images.githubusercontent.com/119781770/210654032-e740f521-9067-4ca2-9f06-3596ca2fa300.jpg)
    
 4. Create 6 volume groups of 10 GB each, same availability zone as that of web-server and db-server       instances and named it as web1, web2, web3, db1, db2, db3 respectively. Attach 3 volume groups         named web1, web2, web3 to the web-server instance and db1,db2,db3 to the db-server instance.
 
 5. SSH into the web and database server instances via the terminal using SSH key-value pair(.pem).
 
 # Project
 
 1. Starting by typing in lsblk command to inspect what block devices are attached to the server.
 2. Using df -h command to see all mounts and free space on your server.
 3. Using gdisk utility to create a single partition on each of the 3 disks-xvdf,xvdh,xvdg
 
      ````````
      sudo gdisk /dev/xvdf
      ``````
     Answering the prompts and then saving it
      
  4. Use lsblk utility to view the newly configured partition on each of the 3 disks.
  
      ![IMG-9304](https://user-images.githubusercontent.com/119781770/210658450-06a9ca8b-19cf-4bdf-8ca4-7d842139227f.jpg)

  5. Install lvm2 package using sudo yum install lvm2. Run 'which lvm' command to check the lvm path.
       
  6. Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM.
       
       ``````````
       sudo pvcreate /dev/xvdf1
       sudo pvcreate /dev/xvdg1
       sudo pvcreate /dev/xvdh1
       
       ```````````
   7. Verify that your Physical volume has been created successfully by running 'sudo pvs'
   
   Step 5,6,7 is shown below:
   
   ![IMG-9308](https://user-images.githubusercontent.com/119781770/211180795-4fff49ed-cf9c-4d3f-8021-4b0055e4f34c.jpg)


   8. Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
   
        ```````
        sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
        ```````
   9. Verify that your VG has been created successfully using the command:
       
       ``````
       sudo vgs
       ``````
       
   Step 8,9,10,11 is shown below:
    
   ![IMG-9309](https://user-images.githubusercontent.com/119781770/210659750-b493cdad-e835-49d8-8433-ac05c56d2427.jpg)

   10. Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-        lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the            Website while, logs-lv will be used to store data for logs.
   
            ````````
            sudo lvcreate -n apps-lv -L 14G webdata-vg
            sudo lvcreate -n logs-lv -L 14G webdata-vg
            ````````
   11. Check that your Logical Volume has been created successfully by running the command 
        'sudo lvs'
        
   12. Using mkfs.ext4 to format the logical volumes with ext4 filesystem (commands shown below)
           
           `````
           sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
           sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
           `````
           
   13. Create /var/www/html directory to store website files
           
           `````
           sudo mkdir -p /var/www/html
           `````
           
   14. Create /home/recovery/logs to store backup of log data
     
           ``````
           sudo mkdir -p /home/recovery/logs
           ``````
           
   15. Mount /var/www/html on apps-lv logical volume
           
           `````
           sudo mount /dev/webdata-vg/apps-lv /var/www/html/
           `````
           
   ![IMG-9311](https://user-images.githubusercontent.com/119781770/210663320-b7a0a7e7-cb05-43a5-a970-d580ab4fc670.jpg)

           
   16. Use rsync utility to backup all the files in the log directory /var/log into                         /home/recovery/logs (This is required before mounting the file system)
            
            `````
            sudo rsync -av /var/log/. /home/recovery/logs/
            `````
   ![IMG-9316](https://user-images.githubusercontent.com/119781770/210663813-c84d6c88-aea6-49bd-b3a1-55c341a55c7f.jpg)

   17. Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be        deleted. That is why step 15 above is very important)
             
             ```
             sudo mount /dev/webdata-vg/logs-lv /var/log
             ````
   18. Restore log files back into /var/log directory
             
             ````
             sudo rsync -av /home/recovery/logs/. /var/log
             `````
   19. The UUID of the device will be used to update the /etc/fstab file; command:
             ````
             sudo blkid
             ````
             

   20. Update /etc/fstab file so that the mount configuration will persist after restart of the              server. 
            
   ![IMG-9318](https://user-images.githubusercontent.com/119781770/210665177-09296948-1ec5-48d7-9c65-8ff9057a1be4.jpg)
   
   

   ![IMG-9317](https://user-images.githubusercontent.com/119781770/210665226-99ef2152-f592-4be6-bb5c-b908ff8bd86d.jpg)
