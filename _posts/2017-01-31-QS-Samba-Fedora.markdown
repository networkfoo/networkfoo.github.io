---
layout: post
title:  "Quick Start Samba - Fedora"
date:   2017-01-31 09:30:00 +0100
categories: fedora samba
---

![Book Cover](/assets/images/2016-12-22-01.png){:style="float: left;margin-right: 10px;margin-top: 7px;"} Samba is a free software re-implementation of the SMB networking protocol, and was originally developed by Andrew Tridgell. Samba provides file and print services for various Microsoft Windows clients and can integrate with a Microsoft Windows Server domain, either as a Domain Controller (DC) or as a domain member. As of version 4, it supports Active Directory and Microsoft Windows NT domains.

Samba runs on most Unix, OpenVMS and Unix-like systems, such as Linux, Solaris, AIX and the BSD variants, including Apple's macOS Server, and macOS client (Mac OS X 10.2 and greater). Samba is standard on nearly all distributions of Linux and is commonly included as a basic system service on other Unix-based operating systems as well. Samba is released under the terms of the GNU General Public License. The name Samba comes from SMB (Server Message Block), the name of the standard protocol used by the Microsoft Windows network file system. 

## Installation & Configuration
```console
yum install samba samba-common samba-client
mkdir -p /var/samba/{data,media}
useradd philip
chown philip:philip /var/samba/{data,media}
smbpasswd -a philip
    New SMB password:
    Retype new SMB password:
    Added user philip.
cp /etc/samba/smb.conf{,.orig} 
vi /etc/samba/smb.conf
setenforce 0
vi /etc/sysconfig/selinux
getenforce
mount /dev/vdb /var/samba/data/
mount /dev/vdc /var/samba/media/
systemctl enable smb.service
    Created symlink from /etc/systemd/system/multi-user.target.wants/smb.service to /usr/lib/systemd/system/smb.service.
systemctl enable nmb.service
    Created symlink from /etc/systemd/system/multi-user.target.wants/nmb.service to /usr/lib/systemd/system/nmb.service.
systemctl start nmb.service
systemctl start smb.service
vi /etc/fstab
```


{% if site.disqus_shortname %}
  {% include disqus_comments.html %}
{% endif %}



