---
layout: default
title:  "Scalable Clos Layer3 Data Center Fabric"
date:   2019-05-09 20:03:00 -0400
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

May 9, 2019

## Properties
* Switches are organized into layers or stages, such as a leaf layer and a spine layer.
* Switches can be all the same model or a couple models.  Switches are typically lower cost 1RU fixed-configuration switches instead of large chassis switches.
* Switches only connect to switches in other layers.  For example, leaves only connect to spines and spines only connect to leaves.
* All links between switches are layer 3 links, never layer 2.  Layer 2/VLANs/Spanning-tree are confined to a single switch. All links will be able to use ECMP (Equal Cost Multi-Path), and switches typically support 16 links in ECMP.
* The fabric is non-blocking.  This means that any server connected to the fabric can utilize its entire link bandwidth to communicate with other endpoints anywhere in the fabric.  For example, a file transfer between servers A and Z will not be throttled by a file transfer between servers B and Y due to all four servers sharing an oversubsribed link somewhere in the fabric.
* Non-blocking is achieved by every stage having the same number of links or bandwidth connecting to the other stages on either side.
* The bandwidth impact of a loss of a single switch can be minimized by increasing the number of switches in the fabric.
* Network can be scaled by adding additional stages.


## Diagram

![Clos Network]({{ site.url }}/assets/clos-network.png)


## Fault tolerance

If a spine switch is lost, only 12.5% of the bandwidth of the fabric is lost.  In the classic architecture with two large distribution switches, a failure results in 50% of bandwidth loss.

## Scaling

The design above is technically a 3-stage clos network, but the two leaf stages are depicted as one big stage.  To scale up to handle more leaves, a 5-stage clos network can be implemented:


![5 Stage Clos Network]({{ site.url }}/assets/5-stage-clos-network.png)