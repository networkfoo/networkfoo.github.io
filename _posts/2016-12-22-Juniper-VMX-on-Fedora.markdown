---
layout: post
title:  "Juniper vMX on Fedora"
date:   2016-12-22 09:30:00 +0100
categories: juniper vMX kvm
---

![Book Cover](/assets/images/2016-12-22-01.png){:style="float: left;margin-right: 10px;margin-top: 7px;"} This past weekend I received access to download the Trial Version for Juniper's vMX router. I have had to change a bunch of stuff to get it up and running on Fedora, which is the os I run KVM from. The trial software can be downloaded from [here](http://www.juniper.net/us/en/dm/free-vmx-trial/), however you will need to set up a relevant account and then contact Juniper Customer Services. This can take some time.

The setup is pretty complex without a guide and you will want to get hold of [Matt Dinham](https://twitter.com/mattdinham)'s excellent Day One Book "[vMX Up & Running](http://www.juniper.net/us/en/training/jnbooks/day-one/automation-series/vmx-up-running/)". You will want ro read, at least chapters 1 & 2 before starting here. 

As the `vMX` orchestration scripts are written to run with `ubuntu`, there was a lot of breakage to deal with so I will attempt to clear the path for any other Fedora users. I read somewhere on the Juniper site that vMX should run under Red Hat, so this may even run on Red Hat or CentOS, out of the box. Please let me know if it does.


### vMX Pre-requisites ###

I will only list the Fedora specifics here. The rest is in "[vMX Up & Running](http://www.juniper.net/us/en/training/jnbooks/day-one/automation-series/vmx-up-running/)" . 

Fedora packages needed:
<pre>
    dnf install	PyYAML 				\
	glib2-devel 				\
	libnl3-devel 				\
	libvirt-daemon-config-network 		\
	libvirt-daemon-kvm 			\
	libxml2-devel 				\
	libxslt-devel 				\
	libyaml-devel 				\
	numactl 				\
	numactl-devel 				\
	python-devel 				\
	python-netifaces			\ 
	qemu-kvm 				\
	redhat-lsb-core 			\
	virt-install 				\
	yajl-devel
</pre>

When that is installed and you have extracted the vMX .tgz bundle, it is a simple matter of applying the patch I have provided [here](https://github.com/networkfoo/juniper/tree/master/vmx-fedroa) to make all the required changes.

    cd /path/to/unbundled/vmx-16.1R1.7
    patch -p1 -i /path/to/vmx-fedora-16.1R1.7.patch 

From here, you need to follow "[vMX Up & Running](http://www.juniper.net/us/en/training/jnbooks/day-one/automation-series/vmx-up-running/)" to sort out your `vmx.conf` file.


### Fedora changes to vMX scripts ###

I haven't done much of anything fancy. All of it can be garnered from reading the patch file, however, here are quick notes on "why I did what":



* `env/fedora_virtio.env` - added file to provide environment variables from Fedora, to vMX.

* `scripts/common/vmx_common_utils.sh` - add `-e` to `echo` to colorize text as necessary.

* `scripts/kvm/common/vmx_kvm_bringup.sh`- make interfaces provide details we can work with.

* `scripts/kvm/common/vmx_kvm_cleanup.sh` - same again, it's all about grepping interfaces.


* `scripts/kvm/common/vmx_kvm_system_setup.sh` - this is going to take a little more explaining:

   * First. `/proc/sys/vm/nr_hugepages` - from what I heve researched this seems to be the location to allocate huge page memory for Fedora. If you have specific needs for Huge Pages you will need to research this more, and please leave a comment if you do. Running `grep Huge /proc/meminfo` will show you details of usage.

    &nbsp;
    <pre>
    [root@localhost vmx-16.1R1.7]# grep Huge /proc/meminfo 
    AnonHugePages:   2844672 kB
    ShmemHugePages:        0 kB
    HugePages_Total:    4096
    HugePages_Free:      189
    HugePages_Rsvd:        0
    HugePages_Surp:        0
    Hugepagesize:       2048 kB
    </pre>
    
  * next we skip the "ubuntu apparmor" foo, as Huge Pages is handle correctly via SELinux.

  * and finally 2 entries to start and stop libvirt.

 
* `scripts/templates/_vPFE-ref.xml` - cutting out `machine='pc-i440fx-trusty'` lets KVM choose the default machine type for your hypervisor.

* `vmx.sh` - just a few missing quotations of variables that made the script die.


### Conclusion ###

So that is just the start of the journey. From here, continue with "[vMX Up & Running](http://www.juniper.net/us/en/training/jnbooks/day-one/automation-series/vmx-up-running/)" to join the growing legion of vMX [mercs](http://www.urbandictionary.com/define.php?term=merc&defid=7321).

{% if site.disqus_shortname %}
  {% include disqus_comments.html %}
{% endif %}


