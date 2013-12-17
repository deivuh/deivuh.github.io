---
layout: post
title: "Bypassing a NAT/Firewall by Reverse SSH Tunneling"
date: 2013-12-15 23:07
comments: true
categories: sysadmin notes how-to
---




Not until last week, my ISP finally decided to change my neighbourhood / home connection from IPoATM to PPPoE, and giving me a direct access to the internet, thus using my router’s NAT configuration.

## The Problem ##

Before they made the changes, I had to struggle with server applications or having to figure out how to access my home’s pc while I was at somewhere else. The only benefit I had from this was having a somewhat more secure network, only those connected to the same ATM bridge would have access to my home network. Asking my ISP to setup a port forwarding to my modem wasn’t a choice, since they would just want to try to sell me a _public static IP service_.

{% img /images/posts/ssh-reverse-tunneling/problem.png  'The problem' 'All incoming connections are blocked' %}

## Using SSH Reverse Tunneling ##

Reading on the web I found out about reverse SSH tunnelling, which consists on creating a SSH connection from the destination machine behind the firewall to the source machine. Since an SSH connection was already open from the destination to the source, all it’s left is to connect through the same SSH connection. 

<!-- more -->

The solution would be as shown below.

{% img right /images/posts/ssh-reverse-tunneling/reverse-ssh.png  'Reverse SSH Tunnel' 'An incoming connection is made by forwarding it to the established outcoming connection '%}


First, the destination has to create an SSH connection to the source 

	$ ssh -R 1234:localhost:22 user@sourceHost

- The `-R` option specifies port forwarding 
- `1234` is the port to listen to, it can be any unused port 
- `localhost` is the host in which the port will be forwarded to
- `22` is the port to which the listening port connection will be forwarded to, in this case the default port in which the SSH connection is made
- `user` is the username at the source machine
- `sourceHost` is the source machine’s domain or IP Address


So, here the port `1234` will be opened and listening at the remote machine, any connection made to this port will be forwarded to port `22` at the localhost through the SSH connection.


This works, but in this case, but both the source and destination machines would have to be on 24/7 in order to keep this connect, which wouldn’t be that convenient at all.


##My setup with Tomato and a VPS##

I have a VPS accesible from anywhere, and a WRT54GL router with Tomato firmware at my home network which can access to my HomePC. Since both hosts do stay on 24/7 is possible to use them to keep an open SSH connection between them. 

So, including those to the equation, the solution setup would be:


   * The router creates an SSH connection with port forwarding to the VPS

   * When needed, from my laptop I can connect from anywhere to my VPS
   * Then from the VPS, a reverse SSH tunnel is made from the listening port through the established connection
   * From the router I can just Wake on LAN my Home PC, and access through another ssh connection o any other way.


{% img right /images/posts/ssh-reverse-tunneling/final-setup.png  'Final setup' "My final setup using my home`s router and a VPS"%}


Because the router could be restarted once in a while, it would have to reconnect automatically after it’s up again, and that’s possible to do so with Tomato. 

First, I had to make sure that Tomato is able to create the SSH connection without authentication, to do so, the router’s rsa public key should be added to the VPS’s /etc/authorized_keys file. 

Keeping in mind that Tomato uses [Dropbear](https://matt.ucc.asn.au/dropbear/dropbear.html) SSH client/server, and not OpenSSH, the command to generate the key’s would be:

	$ dropbearkey -t rsa -f /jffs/rsa_key 

- The option `-t` specifies the type of encryption, and `rsa` is the encoding we want
- `-f /jffs/rsa_key` specifies the file in which the private key will be written to. Notice that I am creating it in the `/jffs` directory. [JFFS](http://tomatousb.org/doc:jffs) is a mounted filesystem part of the router’s flash internal memory; because everything else outside is on RAM, we want it to persist. 

Once the keys are generated, the private will be written to the specified file and the public will be printed on the screen. This public key has to be placed inside the `/.ssh/authorized_keys` file on the VPS.


After setting up the keys, we can test if it works by trying to connect from the router's command line:

	$ ssh -y -f -N -i /jffs/rsa_key -R 1234:0.0.0.0:22 user@VPSHost

- The `-y` option is to accept any remote host key. Since the .ssh/known_hosts file will always be lost after reboot, our VPS host would always be unknown
- The `-N` option to not specify a remote command (This is dropbear SSH, and by default it seems to be used mostly to run remote commands instead of keeping  an session) 
- The `-i` option to use our identity file, our private key to authenticate. /jffs/rsa_key is the private key file.
- The `-R` to specify port forwarding
- `1234` is the port to listen to, it can be any unused port 
- `0.0.0.0` is the host in which the port will be forwarded to, 0.0.0.0 is equal to _any_ host, we can also replace this with the routers external IP address (which would be the same as the home PC’s IP address)
- `22` is the port to which the 1234 port connection will be forwarded to
- `user` is the username to use to login to the VPS
- `VPSHost` is the VPS address


If a remote session in the VPS opens, then it worked. 

Inside the Tomato web GUI, paste the above command to _Administration->Scripts->WanUp section_. This will make Tomato run open an SSH connection to the VPS once the internet connection is established after restarting.

That’s it, now you can open a reverse SSH connection on the VPS by using the command

     $ ssh user@localhost  -p 1234

Where `user` would be your router’s SSH user and `1234` the port you set to listen and forward.

You can wake your PC inside your routers network with _ether-wake_

     $ ether-wake 00:11:22:33:44:55

Where `00:11:22:33:44:55` would be your PC's MAC address. For convenience, we can make Tomato to create a wake up script, paste the following line in _Administration->Scripts->Init_ with the correct MAC address

     $ echo "/usr/bin/ether-wake 00:1C:DF:32:AC:41" > /home/root/wakeDesktopPC.sh 

Viola! All it takes now is to run the script with `./wakeDesktopPC.sh` and access your PC once it finishes booting up.
















