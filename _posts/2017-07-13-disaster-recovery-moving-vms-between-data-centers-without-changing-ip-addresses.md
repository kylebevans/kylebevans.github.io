---
layout: default
title:  "Disaster Recovery: Moving VMs Between Data Centers Without Changing IP Addresses"
date:   2017-07-13 02:15:01 -0400
categories: datacenter bgp disaster-recovery
permalink: /disaster-recovery-moving-vms-between-data-centers-without-changing-ip-addresses/
commentIssueId: 2
---

<h1 class="entry-title">
{% if page.title %}
    <a href="{{ root_url }}{{ page.url }}">{{ page.title }}</a>
{% endif %}
{% if post.title %}
    <a href="{{ root_url }}{{ post.url }}">{{ post.title }}</a>
{% endif %}
</h1>

July 13, 2017

## Problem
Let's assume we have some type of backup software that will backup VMs for us and copy them to our disaster recovery data center.  If we restore those VMs as is in at our DR site, they will have the IP addresses from the offline primary site.  Some software such as Veeam or VMware SRA can be configured to change the IP address for us, but how can it be done without changing the IP Address?

## Solution
* Do not use a Data Center Interconnect (DCI) like OTV.  Shared State = Shared Fate!
* Do not have any dependencies at all between Data Centers.  Shared State = Shared Fate!
* Use BGP to announce your IP Address blocks to your ISPs.
* Dedicate specific IP blocks to specific Data Centers: DC1: 10.0.0.0/24; DC2: 10.0.1.0/24.
* Announce a supernet of the two blocks from both Data Centers: 10.0.0.0/23
* In Internet Routing, the most specific routes are always selected, so as long as both DCs are online, the /23 route will never be used.
* If one DC goes offline, the Internet will start using the /23 route and send traffic to the other DC.  Caveat: BGP must go down at the offline Data Center.
* Restore your VMs at the Data Center that is still online and service will be restored.
* Use BGP on the host so that a VM can advertise its IP to the network instead of needing a default gateway. I will make a future blog post about this.

![Data Center Internet Edge]({{ site.url }}/assets/Data Center Internet Edge.png)

Note: If you are using two stateful firewalls to peer with ISPs, be sure to use AS-Path prepending or have your ISP use a local preference to make sure all traffic flows through one of the firewalls.  If you don't, then return traffic might not go back through the firewall it originated through, and the other firewall will drop it since it's not in the state table.  However, if you are using stateless firewalls or ACLs then don't worry about it.


### Credits:

I learned about this trick from "The Godfather" Ivan Pepelnjak on his blog post
<a href="http://blog.ipspace.net/2012/10/is-layer-3-dci-safe.html">Is Layer-3 DCI Safe?</a>


