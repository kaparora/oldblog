---
layout: post
title: Cinder Availability Zones with NetApp ONTAP
author: Kapil Arora
---

![Image](/images/cinder_az_ontap.png)
In this post we will look at how you can define and configure Cinder Availability Zones with NetApp Clustered Data ONTAP.
Assuming that you have 2 Nova Availability Zones **nova1** and **nova2** and you have NetApp ONTAP **Cluster1** physically located in nova1 availability zone and NetApp ONTAP **Cluster2**  located in nova2 availability zone.

In this scenario you will need to run 2 separate cinder-volume processes and you will need different cinder.conf files for each **cinder-volume** process.

Availability zone can be defined in _cinder.conf_ with the following params:
* storage_availability_zone
* default_availability_zone

Configure and start  **cinder-volume1** and **cinder-volume2** **services** using _cinder1.conf_ and _cinder2.conf_ respectively.
Where cinder1.conf has availability zone set to nova1 and cinder2.conf to nova2.

Define **backends** from cluster1 in cinder1.conf and cluster2 in cinder2.conf.

Once cinder services are configured restart all cinder services and execute the following:
> cinder availability-zone-list

Now you can create new cinder volumes like:
> cinder create --availability-zone nova2 1



