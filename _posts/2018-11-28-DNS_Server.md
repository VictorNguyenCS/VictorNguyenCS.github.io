---
title: "DNS Server implemented in Python"
date: 2018-11-07
tags: [Socket Programming]
exceprt: "DNS Server implemented in Python, using socket programming concepts."
---

I created a DNS Server that utilizes Transmission control protocol (TCP) to provide connection from the DNS Servers to Client Servers. Implementation of the DNS Servers include
both recursive and iterative queries. Recursive queries ask for the DNS server to do all the job fetching the final IP address for your request and return it directly to you.
In the process, the queried DNS server may also query other DNS servers to get the final result. On the other hand, iterative queries means the DNS server will only resolve part of your request and refer you to other DNS servers for your final result.


<img src="images\DNS_Server\Model.png" alt="Model">