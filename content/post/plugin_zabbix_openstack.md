---
title: "Write your own check to monitor resources in OpenStack"

date: 2018-10-11T20:35:31+07:00

categories:
- plugin
# - category
# - subcategory

tags:
- zabbix
- openstack
# - tag1
# - tag2

keywords:
# Define keywords for search engines. you can also define global keywords 
# in Hugo configuration file.
# - tech

thumbnailImage: /img/zabbix.jpg
# thumbnailImage: //example.com/image.jpg
# Image displayed in index view.

thumbnailImagePosition : left
# Display thumbnail image at the right of title 
# in index pages (right, left or bottom). thumbnailImagePosition 
# overwrite the setting thumbnailImagePosition in the theme configuration file.

coverImage:
# Image displayed in full size at the top of your post in post view. If thumbnail image is not configured, cover image is also used as thumbnail image.

coverSize:
# partial: cover image take a part of the screen height (60%), full: cover image take the entire screen height.

coverCaption:
# Add a caption under the cover image
    
gallery:
# - image 1
# - image 2
comments: true
summary: Montoring resources OpenStack 
# Custom excerpt text to show on the homepage.
---

# Montoring resources OpenStack 

Write your own check to monitor resources in OpenStack

<img src="https://i.imgur.com/h1fQnGV.png">

## 1. Prepare

- Zabbix server:
    + Version: 3.0
    + OS: CentOS7
    + IP: 10.10.10.174
- OpenStack Queens
    + OS: CentOS7
    + IP controller: 10.10.10.175
    + Two node compute and One node cinder 

You can install via links reference:

- <a href="https://github.com/congto/openstack-tools/tree/master/scripts/OpenStack-Queens-No-HA/CentOS7">Install OpenStack Queens</a>

- <a href="https://www.zabbix.com/documentation/3.0/manual/installation/install_from_packages/server_installation_with_mysql">Install Zabbix server</a>

**Describe**: The plugin installed in your controller node. It get metric from OpenStack and send to zabbix server via api. 

## 2. Install

### 2.1. In dashboard zabbix, You create items:

- Download templates openstack and import to zabbix server

### 2.2. In controller node:

- Setup python3:

    ```
    yum -y install https://centos7.iuscommunity.org/ius-release.rpm
    yum -y install python35u
    yum -y install python35u-pip
    yum install python35u-devel -y
    ```

#### Install from source:

- Clone source code:

    ```
    https://github.com/hocchudong/plugin_zabbix_openstack.git
    ```
- Update configuration in `etc/config.cfg` of source code and copy it to `/etc/openstack_monitoring/config.cfg`

- Go to openstack_monitoring and install package in file `requirements.txt`

    `pip3.5 install -r requirements.txt`

- Go to openstack_monitoring and run command:

    `python3.5 main.py start|stop|restart`

***or***

#### Install from file setup.py

- Edit `config.cfg` in `etc/config.cfg`

- Go to source code:

    `python3.5 setup.py install`

- Start

    `openstack_monitoring start|stop|restart`

## 3. Features

- TO DO
