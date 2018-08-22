---
layout: post
title: "802.1x Wi-Fi Authentication with Python, FreeRADIUS and Pi"
author: Richard
---

## Background
I moved into an apartment in the middle of town earlier this year, and I wanted to be able to closely monitor activity on my wireless network. I also wanted to be able to quickly give visitors access, without having to give them my invariably long and annoying WPA key. I also love ridiculously over-engineering my home network (my family's home has a Samba4 AD domain, used for Wi-Fi, ownCloud, Subsonic and Windows machine authentication).

I had purchased a Raspberry Pi for use as a NAS (automated backups and media streaming, mostly), so I figured I could use it as a RADIUS server. This didn't, however, solve the issue of storing and managing user information, and providing authentication services to FreeRADIUS (my RADIUS server of choice).

## Technical Overview
The final setup I settled on was an AirPort Express access point connected to my Thomson TG585v7 modem/router/switch (internal AP disabled), with the Raspberry Pi running FreeRADIUS with a custom authentication backend, and nginx hosting the web UI.

### FreeRADIUS
The server version in use as of this writing is v2.1.12. The authentication scheme in use is PEAP-MSCHAPv2.

### Python Backend
The backend was written with mild/low load use cases in mind, and as such does not concern itself with larger database systems, detailed statistics collection, or any fancy "enterprise-grade" features. Python's built in "shelve" module is used to store a copy of user data on-disk, and the backend is specifically built to work with MSCHAPv2 authentication - user passwords are stored as NT hashes. The script also implements the ability to add, remove, enable and disable users.

Because it is running on a machine with relatively low compute capacity, which is doing quite a few other things (web server, DLNA/UPnP server, dnsmasq, Samba3, and FreeRADIUS itself), I chose not to use a daemon-based approach. Instead, FreeRADIUS is configured to execute a script, handing it the necessary arguments (such as User-Name and NT-Password), every time a user authenticates. This makes for a less-scalable solution, but one with zero overhead when it's not needed.

### Web Interface
I created an extremely simple web interface for adding users, as well as enabling, disabling and deleting them. The UI uses a PHP script to handle the requests, and a simple webpage with JavaScript to process and represent JSON-formatted user data produced by the Python script.

### RPi, Access Point and Modem/Router
This is a fairly standard configuration. The access point is configured to authenticate users against the Raspberry Pi's FreeRADIUS server. The Pi and Thomson router have static IP addresses on the LAN (as you might expect). Dnsmasq on the Pi handles DNS and DHCP on the local network, with the Thomson doing firewall/NAT, and not much else.

## Code and Configuration Specifics
I am still working on the code (and there are some fairly glaring database race conditions that require fixing), but will publish it to GitHub and link here when it's ready. It is a very hacky solution though, so if you intend on using it, I'd advise some familiarity with Python!
