---
title: Piggybacking Off An Insecure Shared LAN, Part 1
author: Richard
date: 11/09/2015
layout: post
---

Since moving to Christchurch at the start of this year, I have been in the unfortunate position of having to share one ADSL2+ WAN connection with a fairly large 
number of other people. My living arrangement (fairly common in Christchurch) is a "shared house" -- basically a slightly upmarket flat -- and one of the 
difficulties I've had to contend with is no longer having a dedicated WAN connection to myself.

I am rather a stickler for privacy of my communications, and I don't particularly like the idea of all of my browsing traffic, etc. traversing a poorly secured 
network, managed by my landlord, and shared by a large number of hosts (twenty unique MACs in the last week, as of now). It wasn't until I had some suspicious SSL 
certificate warnings from my browser (that I couldn't reproduce with the same DNS server over my mobile data connection) that I decided to take some action.

Before implementing my solution, my current network topology is as follows:

``<WAN GATEWAY> --- <SHARED ROUTER> (192.168.1.1) ----// WLAN //---- (192.168.1.20) <MY ROUTER -- NAT and SPI firewall> (192.168.3.1) -- <SWITCH / WLAN> -- {my 
devices}``

The setup is already limiting due to the double-NAT configuration, although as we are allowed access to the shared 
router's port forwarding options, this is not a major 
issue for me (in fact I host my TLS'ed Stash instance and SSH server on unprivileged ports, forwarded through the shared router).

How can we limit it some more?! We add another NAT gateway, with another double-NAT configuration... That's how! Of 
course, the difference here is that the second NAT 
gateway's "WAN interface" is actually an OpenVPN tun adapter.


## (Some) Technical Details

Currently my dd-wrt router's ``/proc/net/ip_conntrack`` looks like this:

    udp      17 42 src=192.168.3.163 dst=192.168.3.255 sport=137 dport=137 packets=9 bytes=702 [UNREPLIED] src=192.168.3.255 dst=192.168.3.163 sport=137 dport=137 packets=0 bytes=0 mark=0 use=2
    udp      17 58 src=192.168.3.163 dst=255.255.255.255 sport=17500 dport=17500 packets=872 bytes=149112 [UNREPLIED] src=255.255.255.255 dst=192.168.3.163 sport=17500 dport=17500 packets=0 bytes=0 mark=0 use=2
    udp      17 119 src=192.168.3.24 dst=VPN.PROVIDER.I.P sport=46730 dport=443 packets=63726 bytes=11946902 src=49.50.252.195 dst=192.168.1.10 sport=443 dport=46730 packets=70644 bytes=75235786 [ASSURED] mark=0 use=3
    udp      17 111 src=192.168.3.183 dst=83.140.21.66 sport=37772 dport=8099 packets=7974 bytes=833228 src=83.140.21.66 dst=192.168.1.10 sport=8099 dport=37772 packets=601 bytes=28848 [ASSURED] mark=0 use=2
    udp      17 58 src=192.168.3.163 dst=192.168.3.255 sport=17500 dport=17500 packets=218 bytes=37278 [UNREPLIED] src=192.168.3.255 dst=192.168.3.163 sport=17500 dport=17500 packets=0 bytes=0 mark=0 use=2
    tcp      6 3600 ESTABLISHED src=192.168.3.163 dst=192.168.3.1 sport=58850 dport=22 packets=128 bytes=11457 src=192.168.3.1 dst=192.168.3.163 sport=22 dport=58850 packets=77 bytes=9382 [ASSURED] mark=0 use=3

... In other words, there are only two connections to public IPs - one is from my OpenVPN client/gateway (192.168.3.24) and the other is from my RTL-SDR 
Flightradar24 box, which has a static interface configuration that I have not yet updated. Note the relatively small amount of data transferred on the VPN 
connection - this is because the VPN service has just been automatically restarted (the NAT forwarding on my VPN provider's end seems to sporadically die).

A traceroute from my Windows machine (a Microsoft Surface 3 [not-a-Pro] - a brilliant little machine, may I add?!) yields the following:

    Richard@Richard-Surface ~
    $ tracert www.google.com

    Tracing route to www.google.com [119.161.83.30]
    over a maximum of 30 hops:
    
      1     5 ms     2 ms     1 ms  vpngateway.richardhofman.home [192.168.3.24]
      2    66 ms    41 ms    38 ms  172.20.32.1
      3    49 ms   246 ms   206 ms  VPN.PROVIDER.GW.IP
      4   230 ms    43 ms    64 ms  NAT-1-XXXXXX.xx.xx.nz [x.x.x.x]
      5   166 ms    53 ms    70 ms  ge-x-x-x-xxx.xxx.telstraclear.net [x.x.x.x]
      6   117 ms    49 ms    46 ms  g1-0-0-906.u12.telstraclear.net [203.98.18.66]
      7    56 ms   144 ms    51 ms  ae9-43.tkcr5.global-gateway.net.nz [122.56.127.62]
      8    47 ms    97 ms    54 ms  vocus-dom.tkcr5.global-gateway.net.nz [122.56.118.114]
      9    54 ms    48 ms    52 ms  119.161.83.30
    
    Trace complete.


As you can see, no traffic is passing directly out of my network through the shared house WLAN. Instead, it is going through the OpenVPN client, which is NAT'ing
(netfilter masquerading) it through the encrypted oVPN tunnel kindly provided by my DNS service! As of now, I haven't blocked clients on my LAN from going through 
the standard gateway, but the default handed out via DHCP is the VPN gateway.

## Performance

Obviously, the throughput of my WAN connection has dropped due to the overhead of SSL-VPN'ing everything that comes in and goes out, plus the overhead of passing 
through four NAT configurations, each way, between an internal machine and the destination server. That being said, the throughput I am getting in day-to-day use is 
actually *very* usable... I can happily stream 1080p YouTube videos and Netflix (was secretly hoping it would break this, so I would spend more time doing useful 
stuff...) without any lag whatsoever!

Speedtest.net gives me the following stats (on a fairly slow evening):

![Speedtest.net results](http://www.speedtest.net/result/4654701969.png "Speedtest results")


## Implementation details

The meaty stuff... I will detail the specifics of my setup in a future post. In the meantime, if you're interested in this sort of stuff, try reading the [IPTables 
packet filtering documentation](http://www.netfilter.org/documentation/HOWTO/packet-filtering-HOWTO-7.html) - it's actually a deceptively fun little tool, that I 
had neglected to learn properly until this project!

Until next time...
