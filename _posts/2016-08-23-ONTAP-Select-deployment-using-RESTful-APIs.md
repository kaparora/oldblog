---
layout: post
title: ONTAP Select deployment using RESTful APIs
author: Kapil Arora
---

## What is ONTAP Select?
ONTAP Select is a software-only version of ONTAP 9 implemented as a virtual appliance. It provides an enterprise class storage management solution running on commodity hardware. Because of the inherent flexibility of the virtualization technology, ONTAP Select offers several deployment models that can be used to address different business requirements and usage scenarios.

[Read More](http://www.netapp.com/us/media/ds-3769.pdf)
[Download](http://mysupport.netapp.com/NOW/cgi-bin/software?product=ONTAP+Select&platform=Appliance+Install)

## ONTAP Select Deploy administration utility
The Deploy administration utility is packaged and installed as a separate Linux virtual machine. You must use the utility to deploy the ONTAP Select clusters. Note that the Deploy utility includes a current version of the ONTAP Select node image.
![Image](/images/ontap-deploy.png)

Once you have deployed the "ONTAP select deploy administration utility" VM you can use it to add, configure and manage  Hosts and also create and destroy ONTAP clusters. All these operations can easily be performed using the RESTful APIs.
<p></p>
The Swagger UI can be accessed e.g. @ https://192.168.58.9/api/v1/ui where 192.168.58.9 is the IP of my Deploy VM. Let's look at how you can create a 1 node cluster in a VMware environment using the RESTful APIs.
### Steps to create a 1 node cluster:
1. Add host
2. Configure Host
3. Create Cluster

### Add Host
As a first step, you must provide your ESX host details. In this step we add an ESX Host which will be used for our ONTAP cluster deployment. For this API we need ESX host IP/hostname(esx-01.muccbc.hq.netapp.com in this example provided in the REST URL), username, password and the vcenter Ip/hostname. You can use the following CURL command to execute this step. To execute the CURL commands you will need to provide your ONTAP select deploy utility VMs username and password.
{% highlight sh %}
curl -X PUT --header 'Content-Type: application/json' --header 'Accept: application/json' -d '{
  "password": "Secret01;",
  "username": "restful@muccbc",
  "vcenter": "vc-demo01.muccbc.hq.netapp.com"
}' 'https://192.168.58.9/api/v1/hosts/esx-01.muccbc.hq.netapp.com' --user admin:Secret01 --insecure
{% endhighlight %}
Once the host is added, it will be authenticated. To check if the host was properly added and authenticated run the following curl command and check status of your host.
{% highlight sh %}
curl -X GET --header 'Accept: application/json' 'https://192.168.58.9/api/v1/hosts' --user admin:Secret01 --insecure
{% endhighlight %}
Expected Output:
{% highlight sh %}
[
  {
    "host": "cbc-esx-sdot.muccbc.hq.netapp.com",
    "status": "authenticated",
    "type": "ESX"
  }
]
{% endhighlight %}

### Configure Host
Once the Host is Authenticated, the next step is to configure our ESX host and prepare it for the ONTAP cluster deployment. For this step we need to provide the Network names for data, internal and management networks and a storage Pool.
{% highlight sh %}
curl -X PUT --header 'Content-Type: application/json' --header 'Accept: application/json' -d '{
  "data_network": {
    "name": "VM_STORAGE"
  },
  "internal_network": {
    "name": "VM_STORAGE"
  },
  "location": "MUCCBC",
  "mgmt_network": {
    "name": "VM_MGMT"
  },
  "storage_pool": "LocalDS"
}' 'https://192.168.58.9/api/v1/hosts/esx-01.muccbc.hq.netapp.com/configuration' --user admin:Secret01 --insecure
{% endhighlight %}

Run the following Curl command to check the status of your configuration request. 
{% highlight sh %}
curl -X GET --header 'Accept: application/json' 'https://192.168.58.9/api/v1/hosts' --user admin:Secret01 --insecure
{% endhighlight %}
You should wait until the configuration is finished and following output is seen.
{% highlight sh %}
[
  {
    "host": "cbc-esx-sdot.muccbc.hq.netapp.com",
    "status": "configured",
    "type": "ESX"
  }
]
{% endhighlight %}
### Create Cluster

Once the ESX host is added, authenticated and configured, we can go ahead and send a cluster create request. In this step we are actually going to deploy the ONTAP node on the ESX and create a cluster with the provided managent IP. At the end of this step, we should have a running NetApp ONTAP Select cluster.To execute this API you will need the cluster management IP, node management IP, DNS ips and domain info,gateway, ntp server IP, etc. as shown below in the CURL command. 
{% highlight sh %}
curl -X POST --header 'Content-Type: application/json' --header 'Accept: application/json' -d '{
  "admin_password": "Secret01",
  "cluster_mgmt_ip": "192.168.59.175",
  "dns_info": {
    "dns_ips": [
      "192.168.56.5"
    ],
    "domains": [
      "muccbc.hq.netapp.com"
    ]
  },
  "eval": true,
  "gateway": "192.168.59.1",
  "inhibit_rollback": true,
  "name": "select",
  "netmask": "255.255.255.0",
  "nodes": [
    {
      "host": "esx-01.muccbc.hq.netapp.com",
      "name": "select-01",
      "node_mgmt_ip": "192.168.59.176"
    }
  ],
  "ntp_servers": [
    "192.168.56.5"
  ]
}' 'https://192.168.58.9/api/v1/clusters' --user admin:Secret01 --insecure
{% endhighlight %}

Once the request is successfully sent, you can use the below CURL command to monitor the progress :
{% highlight sh %}
curl -X GET --header 'Accept: application/json' 'https://192.168.58.9/api/v1/clusters' --user admin:Secret01 --insecure
{% endhighlight %}
Once the Cluster creation is completed you should see the state of the Cluster as 'online'.
{% highlight sh %}
[
  {
    "cluster_mgmt_ip": "192.168.59.175",
    "name": "select",
    "num_nodes": 1,
    "state": "online"
  }
]
{% endhighlight %}
### That's it.
Our One node cluster is online and ready. We can now use the cluster management IP and login to the cluster using a console or System Manager.

