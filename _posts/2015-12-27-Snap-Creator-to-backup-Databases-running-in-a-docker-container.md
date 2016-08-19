---
layout: post
title: Use Snap Creator to backup Databases running in a Docker Container
author: Kapil Arora
---

### What is Snap Creator?
Snap Creator is a tool which provides a CLI and GUI and helps you take consistent backups of your applications and or databases.

 More information: [http://www.netapp.com/us/products/management-software/snapcreator-framework.aspx](http://www.netapp.com/us/products/management-software/snapcreator-framework.aspx)
 
 [Download](http://mysupport.netapp.com/NOW/cgi-bin/software?product=Snap+Creator+Framework&platform=All+Platforms) 
 
 We will use MySQL DB as a sample for this post.
 
### How do you make sure  mysql data is stored  in a NetApp volume?
Make sure you mount your NetApp Volume on the Docker Server @ _/mnt/mysql_data_ or at a location of your choice, e.g.
  >sudo mount -t nfs svm_ip:/mysql_data_vol /mnt/mysql_data
 
### How can I start a Container with MySQL?
You can simply run latest mysql database version in a container with following command:
 >docker run --name some-mysql -v /mnt/mysql_data&colon;/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:latest

 **Note:** the -v option and how we provide the volume.
 
 
 
### How do we enable MySQL backup?
To enable backup of MySQL Database we need to make sure Snap Creator Agent is running in the container.
 
### Steps to enable Snap Creator Agent within the MySQL Container:
1. Install Snap Creator server and agent on the Docker host. (I installed it at _/software/snapcreator/_)
2. Install docker-compose
>sudo curl -L https://github.com/docker/compose/releases/download/1.4.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
>sudo chmod +x /usr/local/bin/docker-compose
3. Find Snap Creator Agent under : /software/snapcreator/scAgent4.1.2
4. Create a new entrypoint file for our Docker Container:
```
File: new-entry-point.sh
#! /bin/bash
echo "running sc agent "
/etc/netapp/snapcreator/scAgent4.1.2/bin/scAgent start
echo "running old entrypoint with args"
/entrypoint.sh "$@"
``` 
5. Create a new DockerFile to build our MySQL container with SC Agent enabled
```
File: DockerFile: 
FROM mysql:latest
RUN apt-get update
RUN apt-get install -y openjdk-7-jre-headless
COPY new-entry-point.sh /new-entry-point.sh
ENTRYPOINT ["/new-entry-point.sh"]
CMD ["mysqld"]
```
6. Put both the files Dockerfile and new-entry-point.sh under a folder named **mysql-image** 
7. Create a new docker-compose file
```
File: docker-compose.yml
mysql_db:
build: mysql-image
ports:
- "9090:9090"
- "3306:3306"
volumes:
- /mnt/mysql_data_prod:/var/lib/mysql
- /software/snapcreator/scAgent4.1.2:/etc/netapp/snapcreator/scAgent4.1.2
environment:
MYSQL_ROOT_PASSWORD: netapp01
SCAGENT_HOME: /etc/netapp/snapcreator/scAgent4.1.2
``` 

8. Run the following:
 > docker-compose build
 > docker-compose up -d
 
9. Now your MySQL Container will start with SC Agent running on port 9090 which is exposed also on the docker host at 9090. (which is configurable in the docker-compose.yml file)
10. Now start Snap Creator Server and create a new config for your MySQl database as usual.
 
### Sample code available here:

[https://github.com/kapilarora/continuous-delivery-pipeline/tree/master/ruby-mysql-web-prod](https://github.com/kapilarora/continuous-delivery-pipeline/tree/master/ruby-mysql-web-prod)
