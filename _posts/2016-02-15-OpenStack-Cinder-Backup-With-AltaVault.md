---
layout: post
title: Efficiently protect OpenStack Cinder backup to public/private clouds using AltaVault
author: Kapil Arora
---
It's been observed that more and more organizations are turning to cloud for backup and archival needs. Actually that is many times one of the first workloads that brings cloud into the picture. Cloud Storage is ideal for backup because storage in the cloud is cheap e.g. Glacier pricing for AWS EU (Frankfurt) region is $0.012 per GB / month and EU (Ireland) region is even cheaper at $0.007 per GB / month (Feb 2016).
 
 
### Cloud Storage backup use cases 
 
* **#1** Cold Data storage
Cloud storage makes perfect sense for archival data which must be protected for legal or obligatory reasons. The data may not be accessed ever or for a very long time but still needs to be secured.
 
* **#2** Backup and Restore
Moreover, in backup and restore use cases: cloud storage is a perfect destination for secondary copy of data, away from the original on premise data. This backup data will only be used in most cases when the original copy is lost.
 
* **#3** A combination of the #1 and #2
Both of the above use cases can also be combined together to have a complete backup, restore and archive lifecycle where data older than x days/months/years becomes cold data.
 
### Cinder
Cinder is the OpenStack block storage program and it provides a standard API to manage Block Storage in OpenStack environments. 
Cinder also provides specific APIs for backup and restore functionality for block storage.
 
### Cinder Backup and Cinder Snapshot
A cinder snapshot is a PIT copy on the same storage pool and is dependent on the original copy many times depending on the storage system. On the other hand cinder backup writes a copy of the backup to a different media.
 
### Cinder Backup destination
Cinder can be configured to use either Swift or an NFS share as a backed or destination for storing backups.
 
### AltaVault
NetApp AltaVault is a cloud integrated Storage system which enables you to deliver enterprise class data protection and storage at upto 90% less cost that on premise solution. 
Learn more about the game-changing capabilities of the NetApp AltaVault cloud-integrated storage appliance
 
### Why AltaVault for cinder backup?
There are advantages to cloud backups but it also raises some questions and AltaVault helps us answer some important ones.
 
*  How simple is it to integrate cloud backups into my backup workflow?

 AltaVault acts as a gateway into the cloud, it seamlessly integrates into the OpenStack environment. It simply provides an NFS  front end to your cinder backup and can be installed and setup in around 30 minutes to an hour.
 
*  How secure is my data?

 AltaVault provides FIPS 140-2 encryption all the way through the process, inline, in transit and at rest ensuring that your data is protected at all times.
 
*  Am I locked-in with this cloud vendor? Can I switch cloud vendors if another one offers a better price?

 There is also a cloud migration feature that allows you to migrate your data from one cloud to another should you decide there is a compelling reason to switch.
 
* Speed of data retrieval and restores?
 
 With AltaVault you can keep local copy of recent backups for rapid restore but immediately replicate them to the cloud for off-site storage which helps you design for and meet your recovery SLAs.
 

![Image](/images/cinder-backup-altavault.png)

 With AltaVault you can start as small as 8 TB and scale unto 57 PB and you have multiple deployment options: physical, virtual or cloud based in AWS  or Azure.
Try AltaVault for free @ http://www.netapponcloud.com/try-it-now 
 
### Configuration: AltaVault & cinder-backup
Follow the below simple steps:
1. Create a new NFS export in AltaVault for Cinder backup using AltaVault Web GUI.
2. Configure Cloud provider setting in AltaVault GUI. e.g. Amazon S3 settings.
3. Open **cinder.conf** file usually found under /etc/cinder/ for editing
4. Set _**backup_driver=cinder.backup.drivers.nfs**_
5. Set NFS options for backup. refer: [http://docs.openstack.org/kilo/config-reference/content/nfs-backup-driver.html](http://docs.openstack.org/kilo/config-reference/content/nfs-backup-driver.html)
Example:

 >backup_share=10.65.56.16:/rfs/cinder-backup 
 >backup_mount_options=rw,nolock,hard,intr,nfsvers=3,tcp,rsize=131072,wsize=131072,bg 

6.  Save updated cinder.conf
7.  Restart cinder backup service

 >sudo service openstack-cinder-backup restart
