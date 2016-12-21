---
layout: post
title:  "Using Ansible and the Juniper Junos module with KVM"
date:   2016-12-20 00:15:00 +0100
categories: juniper ansible kvm
---

![Book Cover](/assets/images/2016-12-18-02.png){:style="float: left;margin-right: 10px;margin-top: 7px;"} I started reading [Automating Junos Administration](http://shop.oreilly.com/product/0636920041498.do), the Ansible chapter in specific to get a start on automation.
I will note a couple of things that were peculiar as I worked my way through. These are tips and failures that should affect anyone working through this chapter.

### Console Connections ###

In the chapter, when connecting to a console, this is done via a **Terminal Server** rather than a **Serial Conection**. By default KVM provides serial ports for the console. These are Psuedo Terminals and provisioned under `/dev/pts/*` with * being the number/instance of the `pts`. However, for the purposes of the chapter we will need to reasign our consoles to be available via a Terminal Server. We do this with some XML fiddling so our relevant sections appear as follows:

    <serial type='tcp'>
      <source mode='bind' host='127.0.0.1' service='4555'/>
      <protocol type='telnet'/>
      <target port='0'/>
    </serial>
    <console type='tcp'>
      <source mode='bind' host='127.0.0.1' service='4555'/>
      <protocol type='telnet'/>
      <target type='serial' port='0'/>
    </console>

### junos-netconify ###

If you choose to install via `pip install junos-netconify`, you will get a slightly out of date package with an error in the `facts.py` file. This throws a `"list index out of range"` error and basically means your console connections will not work with the `junos_get_facts` module. It should be changed to represent the following:

<pre>
        # extract the version
        # First try the  tag present in >= 15.1
	<b>swinfo = rsp.findtext('junos-version', default=None)</b>
        if not swinfo:
            # For < 15.1, get version from the "junos" package.
            pkginfo = rsp.xpath(
</pre>

Installing via the GitHub method should avoid this problem.


### Conclusion ###

Well, in hindsight that seems all like a no-brainer but if you come up against this for the first time then I hope these hints can point you in the right direction.

So now we posess a multitude of Ansible foo, what to do? "Automate All of the Things" or "[Automate Some of the Things](http://packetpushers.net/podcast/podcasts/datanauts-053-automate-things/)".

### Update ###

junos-netconify has now been merged into PyEZ so when you read through the chapter...

* you **do not** need to install `junos-netconify` 

* you need to adjust the code to reflect the PyEZ way of doing things.

So what was:
{% raw %}
      ...
      junos_get_facts:
        host: "{{ inventory_hostname }}"
        console: "--telnet={{ jaccess.console_ts }},{{ jaccess.console_port }}"
        user: "root"
        passwd: "{{ jaccess.root_password }}"
        savedir: tmp/
      register: jresult
      ...
{% endraw %}
should appear:
{% raw %}
      ...
      junos_get_facts:
        host: "{{ jaccess.console_ts }}"
        mode: "telnet"
        port: "{{ jaccess.console_port }}"
        user: "root"
        passwd: "{{ jaccess.root_password }}"
        savedir: tmp/
      register: jresult
      ...
{% endraw %}

{% if site.disqus_shortname %}
  {% include disqus_comments.html %}
{% endif %}
