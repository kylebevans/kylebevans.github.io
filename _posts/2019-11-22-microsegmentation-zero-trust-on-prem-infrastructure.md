---
layout: default
title:  "Microsegmentation/Zero Trust with On-Premises Infrastructure"
date:   2019-11-22 10:00:00 -0400
categories: networking datacenter bgp microsegmentation zerotrust onprem
permalink: /microsegmentation-zero-trust-on-prem-infrastructure/
commentIssueId: 4
---

<h1 class="entry-title">
{% if page.title %}
    <a href="{{ root_url }}{{ page.url }}">{{ page.title }}</a>
{% endif %}
{% if post.title %}
    <a href="{{ root_url }}{{ post.url }}">{{ post.title }}</a>
{% endif %}
</h1>

November 22, 2019

## Hardware and Software
* Cisco Nexus 9372PX-E switches
* Dell r740 servers
* VMware vSphere 6.7
* RedHat Enterprise Linux 7
* Windows Server 2012R2
* FRRouting 5.0.1

## Properties
* 


## Diagram

![Clos Network]({{ site.url }}/assets/clos-network.png)


## Fault tolerance

If a spine switch is lost, only 12.5% of the bandwidth of the fabric is lost.  In the classic architecture with two large distribution switches, a failure results in 50% of bandwidth loss.

## Scaling

The design above is technically a 3-stage clos network, but the two leaf stages are depicted as one big stage.  To scale up to handle more leaves, a 5-stage clos network can be implemented:


![5 Stage Clos Network]({{ site.url }}/assets/5-stage-clos-network.png)


### Credits:

Petr Lapukhov's RFC: 
<a href="https://tools.ietf.org/html/rfc7938">Use of BGP for Routing in Large-Scale Data Centers</a>
