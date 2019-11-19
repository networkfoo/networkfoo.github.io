---
layout: post
title:  "Sendmail With SMTO AUTH - FreeBSD"
date:   2017-02-28 09:30:00 +0100
categories: freebsd sendmail
---

![Book Cover](/assets/images/2017-02-28-01.jpg){:style="float: left;margin-right: 10px;margin-top: 7px;"} Samba is a free software re-implementation of the SMB networking protocol, and was originally developed by Andrew Tridgell. Samba provides file and print services for various Microsoft Windows clients and can integrate with a Microsoft Windows Server domain, either as a Domain Controller (DC) or as a domain member. As of version 4, it supports Active Directory and Microsoft Windows NT domains.

## Sendmail DNS Configuration

This is really '''important''', hostname has to be set. Make sure it is set in `/etc/rc.conf` then update on dns zone to add an '''`MX Record`'''. Most likely you can do this with your DNS provider as you will not be running a DNS server..


update '''`/etc/rc.conf`'''.

{{{
echo 'sendmail_enable="YES"' >> /etc/rc.conf
}}}

## Compile Sendmail to support SMTP Auth

First fetch and unpack source:
{{{
fetch ftp://ftp.freebsd.org/pub/FreeBSD/releases/amd64/10.3-RELEASE/src.txz && tar -C / -xzvf src.txz
}}}

Next, add flags for sendmail in '''`/etc/make.conf`'''

{{{
SENDMAIL_CFLAGS=-I/usr/local/include/sasl -DSASL
SENDMAIL_LDFLAGS=-L/usr/local/lib
SENDMAIL_LDADD=-lsasl2
}}}

then recompile sendmail:

{{{
cd /usr/src/lib/libsmutil
make cleandir && make obj && make
cd /usr/src/lib/libsm
make cleandir && make obj && make
cd /usr/src/usr.sbin/sendmail
make cleandir && make obj && make && make install
}}}

After Sendmail has been compiled and reinstalled, edit /etc/mail/freebsd.mc or the local .mc. Many administrators choose to use the output from hostname(1) as the name of .mc for uniqueness. Add these lines:

{{{
dnl set SASL options
TRUST_AUTH_MECH(`GSSAPI DIGEST-MD5 CRAM-MD5 LOGIN')dnl
define(`confAUTH_MECHANISMS', `GSSAPI DIGEST-MD5 CRAM-MD5 LOGIN')dnl
}}}

These options configure the different methods available to Sendmail for authenticating users. To use a method other than pwcheck, refer to the Sendmail documentation.

Finally, run make(1) while in '''`/etc/mail`'''. That will run the new .mc and create a .cf named either freebsd.cf or the name used for the local .mc. Then, run make install restart, which will copy the file to sendmail.cf, and properly restart Sendmail. For more information about this process, refer to /etc/mail/Makefile.

{{{
make
make all install restart
}}}

== Setup SASL Authentication ==

Two steps are required, first creating a user in the '''`saslpasswddb`''', then  indicating in the '''`Sendmail.conf`''' file that we will be using the '''`saslpasswddb`'''.

{{{
saslpasswd2 philip
echo "pwcheck_method: auxprop" > /usr/local/lib/sasl2/Sendmail.conf
}}}

you can then check if SASL Auth is working in Sendmail by running telent and verifying '''`250-AUTH DIGEST-MD5 CRAM-MD5 LOGIN`''' exists:

{{{
telnet localhost 25
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
220 muscleman.thepark ESMTP Sendmail 8.15.2/8.15.2; Sat, 5 Mar 2016 09:53:49 +0100 (CET)
ehlo localhost
250-muscleman.thepark Hello localhost [127.0.0.1], pleased to meet you
250-ENHANCEDSTATUSCODES
250-PIPELINING
250-8BITMIME
250-SIZE
250-DSN
250-ETRN
250-AUTH DIGEST-MD5 CRAM-MD5 LOGIN
250-STARTTLS
250-DELIVERBY
250 HELP
^]
telnet> quit
Connection closed.
}}}

== Permitting Sendmail to accept emails coming from the SASL Authed User ==

We need to allow our domain to send email from the SASL Authed User. This is accomplished by editing the '''`/etc/mail/access`''' file. Whenever this file is updated, update its database with '''`makemap hash`''' and rtestart sendmail.

{{{
cd /etc/mail
echo "From:thepark            OK" >> access
makemap hash /etc/mail/access < /etc/mail/access
service sendmail restart
}}}

Now we can send an email from a MUA, for example Thunderbird via SMTP Auth and sendmail will deliver this to any mail server on the internet. 

Note: as we do not have a registered domain name, the recieving mail server may block this email.


== A note on Key permissions for STARTTLS ==

You will need to add the following to your host.mc file to allow use of a group readable private key.

{{{
echo "O DontBlameSendmail=GroupReadableKeyFile" >> hostname.mc
make
make all install restart
}}}


{% if site.disqus_shortname %}
  {% include disqus_comments.html %}
{% endif %}



