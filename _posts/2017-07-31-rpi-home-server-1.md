---
layout: post
title: "Raspberry Pi Home Server"
date: 2017-07-31
---

## Secure access, downloads, automation, and files
I've had a Raspberry Pi 1 running as a pihole (DNS server that has blocklists of ad domains) and doing fairly well deflecting ads from devices that don't have built in adblock. Even without ad blocking, it gives very easy to use analytics of how many DNS lookups are being made and to where, which can catch compromised computers.

Since it did so well as an always-on home server of sorts, I decided to fully turn it into a server that will run for pennies a month.

I thought about what I wanted it to do, and decided on these requirements:
1. Act as a network attached storage. It won't be the fastest, but it would allow me to move things between devices or to be accessible from all devices. The main advantage is being able to upload and download files without having to get them on Dropbox/Drive/OneDrive/MEGA.
2. Act as a download control server for regular downloads, youtube downloads (cursed Canadian internet often buffers), and torrent downloads (it's about time I seed my Ubuntu/Raspbian/CentOS ISOs 24/7)
3. Provide the capability to have my own web server and mail server
4. Provide internet SSH access in a hardened way that won't get pwned by automated brute forcers, script kiddies, or elite international hackers of mystery

I swapped out the SD card and started from scratch on Raspbian minimal about 2 weeks ago. As of 2017-08-05, items 1 and 4 are going quite well, and items 2 and 3 got about halfway done before refusing to cooperate.

External (connect to RPi on home network from anywhere) SSH was easy to set up with a number of great guides available. I port forwarded an obscure port to 22 on the Pi, and installed fail2ban to deal with brute force attempts. I made fail2ban block-happy by changing the ban to a 24hr ban after 5 wrong attempts within the span of 24h.

The storage is currently two 4GB USB sticks being used to test how various file systems play with automount. Currently the nicest has been FAT32/FAT16 (after dealing with the default permissions being root ownership), and I think ext2/3/4 will also be OK. NTFS/exFAT were not cooperative. When I need more space I will attach an external hard drive through a USB hub.

Using youtube-dl and wget to download files has worked quite well. The Pi doesn't have amazing IO but it is directly connected to the router by ethernet and running a download on it is fire and forget. For torrent daemons deluge and transmission installed fine, but lead into the wormhole time sink of the web server.

## Web server issues
The first thing I tried to set up on the Pi after doing the OS setup was installing the wiki software MoinMoin. After some minor hiccups with the dependencies it installed and ran without issue. The nginx webserver software it brought in could be accessed over the network with no issues. I could access MoinMoin through Lynx browser on the RPi, but no matter what ports I opened or which daemons I restarted it simply refused to serve the wiki pages to non-localhost clients.

Not wanting to get bogged down, I moved on to setting up the torrent software. Since I do not want to SSH in every time I want to start or check the torrents, I installed Deluge, which can be accessed remotely by other Deluge clients, or by a (nginx delivered) web interface. Both options, of course, would not work for unknown reasons. So I switched to Transmission which also has a (nginx) web interface; that also would not work for unknown reasons. It appeared the connection would come to the Pi but would never make it to the daemon listening for the connections. No amount of permisiveness in IPTables changed this, nor did disabling fail2ban.

My guess is that the issue lies somewhere between IPTables, the configuration of nginx, and the application daemons. None of the guides mentioned needing to fiddle with IPTables or nginx, and I made sure to properly configure the applications. As a last resort I fully removed Deluge, Transmission, MoinMoin, and nginx to make ready for a second attempt at setting up a torrent interface and personal wiki.


## Notes:

Here are some useful links that I used:

https://www.pestmeester.nl/index.html

http://kamilslab.com/2016/12/11/how-to-install-fail2ban-on-the-raspberry-pi/

