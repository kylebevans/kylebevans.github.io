---
layout: default
title:  "Ansible Dynamic Inventory from SharePoint"
date:   2017-06-29 02:46:01 -0400
categories: ansible sharepoint
---

Out in Enterprise-land, SharePoint is a fact of life.  If you are using a SharePoint list as the source of truth for your equipment documentation, then you can use an Ansible Dynamic Inventory Python script to generate your Ansible inventory.  Basically, instead of trying to keep a file like /etc/ansible/hosts up to date, you can specify a python script that will generate your inventory dynamically from your source of truth, in this case SharePoint.

SharePoint 2013 includes a REST API that you can use to grab data in JSON or ATOM formats, but we will use JSON since the output from the dynamic inventory script must also be JSON.  In order to interact with SharePoint from Python, we will use the "requests" Python module along with "requests_ntlm" to handle NTLM authentication to SharePoint.  We will download the data and store it in a dictionary object. Also, for the sake of simplicity, let's say you have a SharePoint list of devices with the columns "Title" and "Model."  The Title column for each device is also a valid hostname for the device so that Ansible can use it to connect to the device.

First of all, the dynamic inventory script must support being called in two ways:

```
ansible_dynamic_inventory.py --list

ansible_dynamic_inventory.py --host <hostname>
```

