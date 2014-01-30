---
layout: post
title: "Why using Google DNS wasn't a great idea"
date: 2014-01-29 18:51:31 -0600
comments: true
categories: sysadmin notes networks
---



<!-- ##Why setting up Google DNS as my router's default DNS server was a bad idea# -->

During the last couple of week of last month, I noticed delays while browsing the web, especially Reddit, up to the point that started to bug me. Was it because of the [recent change made by my ISP](http://davidhsiehlo.com/blog/2013/12/20/finally/)? Or because I changed my Tomato router to manage my DSL connection? 


The first thing I did was a [speed test](http://www.speedtest.net/my-result/3268882393), but everything seemed to be normal, so it wasn't a speed issue. So the DNS server became my first suspect, since I setup my router to use Google DNS by default, both main and alternate server. But why would Google DNS server be slow? I thought. 

So I proceeded to make some tests:

First, I ran a ping test to reddit.com domain:

	➜  ~  ping www.reddit.com
	PING a659.b.akamai.net (165.254.57.144): 56 data bytes
	64 bytes from 65.120.60.171: icmp_seq=400 ttl=48 time=89.331 ms
	64 bytes from 65.120.60.171: icmp_seq=401 ttl=48 time=93.348 ms
	64 bytes from 65.120.60.171: icmp_seq=402 ttl=48 time=92.700 ms
	....
	64 bytes from 65.120.60.171: icmp_seq=755 ttl=48 time=360.952 ms
	64 bytes from 65.120.60.171: icmp_seq=756 ttl=48 time=284.577 ms
	64 bytes from 65.120.60.171: icmp_seq=757 ttl=48 time=201.729 ms

The average was about 90 ms, which is slow but acceptable; but there were times were it reached up to 360+ ms of latency! That's high enough to be noticeable when browsing.

<!-- more -->

So I did some ping tests to Google DNS server too:

	➜  ~  ping 8.8.8.8
	PING 8.8.8.8 (8.8.8.8): 56 data bytes
	64 bytes from 8.8.8.8: icmp_seq=0 ttl=38 time=57.962 ms
	64 bytes from 8.8.8.8: icmp_seq=1 ttl=38 time=58.151 ms
	64 bytes from 8.8.8.8: icmp_seq=2 ttl=38 time=60.308 ms
	...
	64 bytes from 8.8.8.8: icmp_seq=104 ttl=38 time=510.392 ms
	64 bytes from 8.8.8.8: icmp_seq=105 ttl=38 time=236.928 ms
	64 bytes from 8.8.8.8: icmp_seq=106 ttl=38 time=154.764 ms
	
Apparently, the speed problem was indeed originated from the DNS server connection.  

That's when the first possibility came to my mind. To confirm, the next thing I did was to run a traceroute to the Google DNS server:

	➜  ~  traceroute 8.8.8.8
	traceroute to 8.8.8.8 (8.8.8.8), 64 hops max, 52 byte packets
	 1  192.168.2.1 (192.168.2.1)  1.204 ms  1.048 ms  1.230 ms
	 2  10.192.34.115 (10.192.34.115)  6.730 ms  6.707 ms  6.607 ms
	 3  10.192.56.149 (10.192.56.149)  7.416 ms  7.702 ms  7.261 ms
	 4  * * *
	 5  10.192.37.213 (10.192.37.213)  12.911 ms  48.465 ms  89.934 ms
	 6  10.192.37.169 (10.192.37.169)  152.473 ms  137.849 ms  16.745 ms
	 7  10.192.55.150 (10.192.55.150)  18.766 ms  19.843 ms  192.897 ms
	 8  10.192.12.38 (10.192.12.38)  195.543 ms  171.874 ms  442.763 ms
	 9  inetin-ptobar-ptobarrios-6-tge1-0-0.uninet.net.mx (187.174.64.26)  	309.486 ms  309.899 ms  307.067 ms
	10  ideint-miami-americas-15-tge0-0-3-0.uninet.net.mx (201.125.198.10)  	58.064 ms  58.251 ms  57.935 ms
	11  customer-201-125-242-65.uninet.net.mx (201.125.242.65)  58.259 ms  	58.076 ms  58.378 ms
	12  209.85.253.74 (209.85.253.74)  60.183 ms  61.403 ms
    209.85.253.118 (209.85.253.118)  103.468 ms
	13  209.85.252.96 (209.85.252.96)  79.788 ms  111.421 ms
    209.85.252.98 (209.85.252.98)  100.892 ms
	14  209.85.243.254 (209.85.243.254)  70.847 ms
    209.85.248.31 (209.85.248.31)  75.368 ms
	    209.85.243.254 (209.85.243.254)  57.866 ms
	15  * * *
	16  google-public-dns-a.google.com (8.8.8.8)  77.301 ms  76.182 ms  73.930 ms

By using an IP location service, [the IP address of the last hop resulted from Georgia, USA](http://www.iplocation.net/index.php?query=209.85.243.254). 

Well, just as I thought, the DNS server reached was actually from the US, as I checked thereafter, [there are actually no Google DNS servers closer to Guatemala](http://blog.archit.in/2012/02/google-public-dns-server-locations-list/). 

Well, so I setup the DNS servers that were previously provided by my ISP, and problem solved. 

	➜  ~  ping www.reddit.com
	PING a659.b.akamai.net (186.151.236.160): 56 data bytes
	64 bytes from 186.151.236.160: icmp_seq=0 ttl=59 time=7.843 ms
	64 bytes from 186.151.236.160: icmp_seq=1 ttl=59 time=7.932 ms
	64 bytes from 186.151.236.160: icmp_seq=2 ttl=59 time=8.010 ms
	
###Conclusion###

The location of the DNS server does affect speed significantly when browsing, or in whatever activity in which domain name resolutions are involved.  Make sure that the DNS server you're using is actually close to you. Unless there are any restrictions by your ISP's DNS server, stick just with it.



	
		
	
		