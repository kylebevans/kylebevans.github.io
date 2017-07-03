---
layout: default
title:  "Ansible Dynamic Inventory from SharePoint"
date:   2017-06-29 02:46:01 -0400
categories: ansible sharepoint
permalink: /ansible-dynamic-inventory-from-sharepoint/
commentIssueId: 1
---

<h1 class="entry-title">
{% if page.title %}
    <a href="{{ root_url }}{{ page.url }}">{{ page.title }}</a>
{% endif %}
{% if post.title %}
    <a href="{{ root_url }}{{ post.url }}">{{ post.title }}</a>
{% endif %}
</h1>

{{ page.commentIssueId }}

June 29, 2017

Out in Enterprise-land, SharePoint is a fact of life.  If you are using a SharePoint list as the source of truth for your equipment documentation, then you can use an Ansible Dynamic Inventory Python script to generate your Ansible inventory.  Basically, instead of trying to keep a file like /etc/ansible/hosts up to date, you can specify a python script that will generate your inventory dynamically from your source of truth, in this case SharePoint.

SharePoint 2013 includes a REST API that you can use to grab data in JSON or ATOM formats, but we will use JSON since the output from the dynamic inventory script must also be JSON.  In order to interact with SharePoint from Python, we will use the "requests" Python module along with "requests_ntlm" to handle NTLM authentication to SharePoint.  We will download the data and store it in a dictionary object. Also, for the sake of simplicity, let's say you have a SharePoint list called of devices called "Devices" with the columns "Title" and "Model."  The Title column for each device is also a valid hostname for the device so that Ansible can use it to connect to the device.

First of all, the dynamic inventory script must support being called in two ways:

```
ansible_dynamic_inventory.py --list

ansible_dynamic_inventory.py --host <hostname>
```

Using --list should print out the entire inventory in JSON, and using --host <hostname> should print out either the JSON for a single host or an empty JSON block.  For simplicity we are only going to print out an empty block and focus on getting --list to print out the full inventory in JSON.  We will also add support for a '-h' flag to print out usage.  To handle command line options, we use the getopt module, so the beginning of our script looks like this:

```python
#!/bin/python

import os, sys, getopt
import json
import requests
from requests_ntlm import HttpNtlmAuth

username = 'sharepointuser'
password = os.environ['SHAREPOINTPASSWORD']

try:
  opts, args = getopt.getopt(sys.argv[1:],"h",["list","host"])
except getopt.GetoptError:
  print "%s [--list]" % __file__
  print "%s [--host <hostname>]" % __file__
  sys.exit(2)

if len(sys.argv) < 2:
  print "%s [--list]" % __file__
  print "%s [--host <hostname>]" % __file__
  sys.exit(2)

for opt, arg in opts:
  if opt == '-h':
    print "%s [--list]" % __file__
    print "%s [--host <hostname>]" % __file__
    sys.exit()
  elif opt == '--host':
    print "{"
    print "}"
    sys.exit()
  # else must be --list, so go on with script
```

Notice above that there is a username and password variable that you must set.  The password variable as it is is set from the SHAREPOINTPASSWORD environment variable.  After that, we are ready to connect to SharePoint and get our data.


```python
headers = {'Accept': 'application/json;odata=verbose'}
response = requests.get("https://sharepoint.example.com/department/it/_api/web/lists/GetByTitle('Devices')/Items",auth=HttpNtlmAuth("ad\\" + username, password), headers=headers)
data = response.json()
```

The code above connects to SharePoint, authenticates, asks for the list data in JSON format, and saves it in a JSON object called "data."  Make sure you use your actual AD domain if it's not "AD."

Next, let's use a Python dictionary to prepare our output.  We will create groups by model and set a "netos" var for each group.  Setting up the dictionary is a little verbose since we have to initialize each level of the hierarchy as a dict or a list as appropriate:

```python
inventory = {}
inventory['c6500s'] = {}
inventory['c4500s'] = {}
inventory['c3850s'] = {}
inventory['c3750Xs'] = {}
inventory['c3560s'] = {}
inventory['c5516Xs'] = {}
inventory['c9372s'] = {}

inventory['c6500s']['hosts'] = []
inventory['c4500s']['hosts'] = []
inventory['c3850s']['hosts'] = []
inventory['c3750Xs']['hosts'] = []
inventory['c3560s']['hosts'] = []
inventory['c5516Xs']['hosts'] = []
inventory['c9372s']['hosts'] = []

inventory['c6500s']['vars'] = {'netos': 'ios'}
inventory['c4500s']['vars'] = {'netos': 'ios'}
inventory['c3850s']['vars'] = {'netos': 'ios'}
inventory['c3750Xs']['vars'] = {'netos': 'ios'}
inventory['c3560s']['vars'] = {'netos': 'ios'}
inventory['c5516Xs']['vars'] = {'netos': 'asa'}
inventory['c9372s']['vars'] = {'netos': 'nxos'}

inventory['c6500s']['children'] = []
inventory['c4500s']['children'] = []
inventory['c3850s']['children'] = []
inventory['c3750Xs']['children'] = []
inventory['c3560s']['children'] = []
inventory['c5516Xs']['children'] = []
inventory['c9372s']['children'] = []
```

Next we use the data from SharePoint to fill out the dictionary we just set up.  The JSON data from SharePoint is in data['d']['results'], don't ask me what the significance of the "d" is.

```python
for device in data['d']['results']:
  if device['Model'] == 'Catalyst 6509-E':
    inventory['c6500s']['hosts'].append(device['Title'])
  elif device['Model'] == 'Catalyst 4507R+E':
    inventory['c4500s']['hosts'].append(device['Title'])
  elif device['Model'] == 'Catalyst 3850':
    inventory['c3850s']['hosts'].append(device['Title'])
  elif device['Model'] == 'Catalyst 3750X':
    inventory['c3750Xs']['hosts'].append(device['Title'])
  elif device['Model'] == 'Catalyst 3560' or device['Model'] == 'Catalyst 3560G':
    inventory['c3560s']['hosts'].append(device['Title'])
  elif device['Model'] == 'ASA 5516-X':
    inventory['c5516Xs']['hosts'].append(device['Title'])
  elif device['Model'] == 'Nexus 9372':
    inventory['c9372s']['hosts'].append(device['Title'])
```


Finally, we output our inventory in JSON:

```python
print json.dumps(inventory, indent=4, separators=(',', ': '))
```


Sample output:

```json
{
    "c4500s": {
        "hosts": [
            "dist-switch1",
            "dist-switch2",
            "dist-switch3"
        ],
        "children": [],
        "vars": {
            "netos": "ios"
        }
    },
    "c3750Xs": {
        "hosts": [
            "room10-switch1",
            "room20-switch1",
            "room30-switch2"
        ],
        "children": [],
        "vars": {
            "netos": "ios"
        }
    },
    "c5516Xs": {
        "hosts": [
            "firewall1",
            "firewall2",
            "firewall3"
        ],
        "children": [],
        "vars": {
            "netos": "asa"
        }
    }
}
```

Usage with ansible-playbook:

```
export SHAREPOINTPASSWORD='hithere'

ansible-playbook main.yml -vvvv -f 10 -i /etc/ansible/ansible_dyn_inventory.py --extra-vars '{"cli": {"username": "thisguy","password": "SneakyPassword11","host": "{{ inventory_hostname }}","timeout": "60"}, "asacli": {"username": "thisguy","password": "SneakyPassword11","host": "{{ inventory_hostname }}","timeout": "60","authorize": "yes","auth_pass": "SneakyPassword11"}}'
```


Full script:

```
#!/bin/python

import os, sys, getopt
import json
import requests
from requests_ntlm import HttpNtlmAuth

username = 'sharepointuser'
password = os.environ['SHAREPOINTPASSWORD']

try:
  opts, args = getopt.getopt(sys.argv[1:],"h",["list","host"])
except getopt.GetoptError:
  print "%s [--list]" % __file__
  print "%s [--host <hostname>]" % __file__
  sys.exit(2)

if len(sys.argv) < 2:
  print "%s [--list]" % __file__
  print "%s [--host <hostname>]" % __file__
  sys.exit(2)

for opt, arg in opts:
  if opt == '-h':
    print "%s [--list]" % __file__
    print "%s [--host <hostname>]" % __file__
    sys.exit()
  elif opt == '--host':
    print "{"
    print "}"
    sys.exit()
  # else must be --list, so go on with script

headers = {'Accept': 'application/json;odata=verbose'}
response = requests.get("https://sharepoint.example.com/department/it/_api/web/lists/GetByTitle('Devices')/Items",auth=HttpNtlmAuth("ad\\" + username, password), headers=headers)
data = response.json()


inventory = {}
inventory['c6500s'] = {}
inventory['c4500s'] = {}
inventory['c3850s'] = {}
inventory['c3750Xs'] = {}
inventory['c3560s'] = {}
inventory['c5516Xs'] = {}
inventory['c9372s'] = {}

inventory['c6500s']['hosts'] = []
inventory['c4500s']['hosts'] = []
inventory['c3850s']['hosts'] = []
inventory['c3750Xs']['hosts'] = []
inventory['c3560s']['hosts'] = []
inventory['c5516Xs']['hosts'] = []
inventory['c9372s']['hosts'] = []

inventory['c6500s']['vars'] = {'netos': 'ios'}
inventory['c4500s']['vars'] = {'netos': 'ios'}
inventory['c3850s']['vars'] = {'netos': 'ios'}
inventory['c3750Xs']['vars'] = {'netos': 'ios'}
inventory['c3560s']['vars'] = {'netos': 'ios'}
inventory['c5516Xs']['vars'] = {'netos': 'asa'}
inventory['c9372s']['vars'] = {'netos': 'nxos'}

inventory['c6500s']['children'] = []
inventory['c4500s']['children'] = []
inventory['c3850s']['children'] = []
inventory['c3750Xs']['children'] = []
inventory['c3560s']['children'] = []
inventory['c5516Xs']['children'] = []
inventory['c9372s']['children'] = []

for device in data['d']['results']:
  if device['Model'] == 'Catalyst 6509-E':
    inventory['c6500s']['hosts'].append(device['Title'])
  elif device['Model'] == 'Catalyst 4507R+E':
    inventory['c4500s']['hosts'].append(device['Title'])
  elif device['Model'] == 'Catalyst 3850':
    inventory['c3850s']['hosts'].append(device['Title'])
  elif device['Model'] == 'Catalyst 3750X':
    inventory['c3750Xs']['hosts'].append(device['Title'])
  elif device['Model'] == 'Catalyst 3560' or device['Model'] == 'Catalyst 3560G':
    inventory['c3560s']['hosts'].append(device['Title'])
  elif device['Model'] == 'ASA 5516-X':
    inventory['c5516Xs']['hosts'].append(device['Title'])
  elif device['Model'] == 'Nexus 9372':
    inventory['c9372s']['hosts'].append(device['Title'])



print json.dumps(inventory, indent=4, separators=(',', ': '))
```

Links:

<a href="http://docs.ansible.com/ansible/dev_guide/developing_inventory.html">Ansible Dynamic Inventory</a>
<br />
<a href="https://dev.office.com/sharepoint/docs/sp-add-ins/complete-basic-operations-using-sharepoint-rest-endpoints">SharePoint REST API</a>
