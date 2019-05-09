---
layout: default
title:  "Scalable Clos Layer3 Data Center Fabric"
date:   2019-05-07 15:45:00 -0400
categories: networking datacenter bgp
permalink: /scalable-clos-L3-DC-fabric/
commentIssueId: 3
---

<h1 class="entry-title">
{% if page.title %}
    <a href="{{ root_url }}{{ page.url }}">{{ page.title }}</a>
{% endif %}
{% if post.title %}
    <a href="{{ root_url }}{{ post.url }}">{{ post.title }}</a>
{% endif %}
</h1>

May 7, 2019

## Properties
* Switches are organized into layers or stages, such as a leaf layer and a spine layer.
* Switches only connect to switches in other layers.  For example, leaves only connect to spines and spines only connect to leaves.
* All links between switches are layer 3 links, never layer 2.  Layer 2/VLANs are confined to a single switch.
* The fabric is non-blocking.  This means that any server connected to the fabric can utilize its entire link bandwidth to communicate with other endpoints anywhere in the fabric.  For example, a file transfer between servers A and Z will not be throttled by a file transfer between servers B and Y due to all four servers sharing an oversubsribed link somewhere in the fabric.
* Non-blocking is achieved by every stage having the same number of links or bandwidth connecting to the other stages on either side.
* The bandwidth impact of a loss of a single switch can be minimized by increasing the number of switches in the fabric.


## Diagram

![Clos Network]({{ site.url }}/assets/clos-network.png)
