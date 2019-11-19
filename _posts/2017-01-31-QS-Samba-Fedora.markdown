---
layout: post
title:  "Juniper vQFX on KVM - Fedora"
date:   2017-01-10 09:30:00 +0100
categories: juniper vQFX kvm
---

![Book Cover](/assets/images/2016-12-22-01.png){:style="float: left;margin-right: 10px;margin-top: 7px;"} The last week I have been running vQFX10000 on KVM to get through some lab exercises. At the moment it is shipped as a Vagrant install with VMWare vmdk disks to be installed on VirtualBox. None of these products I run or have any interest in running, let alone combobulating all three together. Here are a quick few notes on getting it up and running under KVM.


### Convert vmdk disks to raw images ###

Use the *qemu-img* program to convert the vmdk disk images to a raw format. These will later be mounted as an *ide* drive.

<pre>
qemu-img convert -f vmdk /path/to/vqfx10k-re-15.1X53-D60.vmdk  -O raw /path/to/vqfx10k-re-15.1X53-D60.raw
qemu-img convert -f vmdk /path/to/vqfx10k-pfe-20160609-2.vmdk  -O raw /path/to/qfx2/vqfx10k-pfe-20160609-2.raw
</pre>

### Setup internal RE-PFE Bridge ###

This will be the bridge that is configured on the second interface of each *RE* & *PFE* vm. Quicksmart!

<pre>
brctl addbr qfx-int
ifconfig qfx-int up
</pre>

Create another bridge for your first dataplane interface *xe-0/0/0*.

<pre>
brctl addbr dataplane
ifconfig dataplane up
</pre>

**Note:** Dataplane interfaces are installed on the Route Engine vm (not the Packet Forward Engine vm). Also dataplane interfaces will be configured **starting at the fourth interface slot**. The third interface slot is saved for an unused *management* interface.

### Install Routing Engine ###

The only tricky thing about this install is the networking interfaces. I have *openvswitch* running which I connect all my lab equipment to a *mgmt vlan* as you can see from the install below. You will need to change this according to your own situation. The other interface cards are configured as bridges, which we just created.

<pre>
virt-install \
    --name re-hostname \
    --memory 1024 \
    --vcpus=1 \
    --import \
    --disk /path/to/vqfx10k-re-15.1X53-D60.raw,bus=ide,format=raw \
    --network network=openvswitch-net,portgroup=mgmt,model=e1000 \
    --network bridge=qfx-int,model=e1000 \
    --network network=openvswitch-net,portgroup=mgmt,model=e1000 \
    --network bridge=dataplane,model=e1000 \
    --graphics none
</pre>

Login with username *root* and password *Juniper*.

### Install Packet Forward Engine ###

The install on the PFE is a little more simple as there are only two interface cards to be concerned with and we have covered these already.

<pre>
virt-install \
    --name pfe-hostname \
    --memory 2048 \
    --vcpus=1 \
    --import \
    --disk /path/to/vqfx10k-pfe-20160609-2.raw,bus=ide,format=raw \
    --network network=openvswitch-net,portgroup=mgmt,model=e1000 \
    --network bridge=qfx-int,model=e1000 \
    --graphics none
</pre>


**Note:** There is *one big gotcha* with the installation, as it starts. There will be no console output after the *grub menu*. So, when you see the *grub menu*, **press *e* for "edit"** and insert into the kernel line **console=ttyS0** to continue. At this stage there is no reason to log into the PFE via console as it will become available via *ssh* once the management interface grabs an ip number from *dhcp*.

Login with username *root* and password *no*.

### Conclusion ###

So when they have both had time to settle, you can check out out various commands to verify connectivity.  Pretty neat huh!

<pre>
{master:0}
root@vqfx-re> show chassis pic fpc-slot 0 pic-slot 0 
FPC slot 0, PIC slot 0 information:
  Type                             48x 10G-SFP+
  State                            Online    
  PIC version                  2.9

  Uptime			 35 seconds

PIC port information:
                         Fiber                    Xcvr vendor       Wave-    Xcvr
  Port Cable type        type  Xcvr vendor        part number       length   Firmware
  0    10GBASE SR        MM    SumitomoElectric   SPP5101SR-J3      850 nm   0.0   
  1    10GBASE SR        MM    SumitomoElectric   SPP5101SR-J3      850 nm   0.0   
  2    10GBASE SR        MM    SumitomoElectric   SPP5101SR-J3      850 nm   0.0   
  3    10GBASE SR        MM    SumitomoElectric   SPP5101SR-J3      850 nm   0.0   
  4    10GBASE SR        MM    SumitomoElectric   SPP5101SR-J3      850 nm   0.0   
  5    10GBASE SR        MM    SumitomoElectric   SPP5101SR-J3      850 nm   0.0   
  6    10GBASE SR        MM    SumitomoElectric   SPP5101SR-J3      850 nm   0.0   
  7    10GBASE SR        MM    SumitomoElectric   SPP5101SR-J3      850 nm   0.0   
  8    10GBASE SR        MM    SumitomoElectric   SPP5101SR-J3      850 nm   0.0   
  9    10GBASE SR        MM    SumitomoElectric   SPP5101SR-J3      850 nm   0.0   
  10   10GBASE SR        MM    SumitomoElectric   SPP5101SR-J3      850 nm   0.0   
  11   10GBASE SR        MM    SumitomoElectric   SPP5101SR-J3      850 nm   0.0 
</pre>


{% if site.disqus_shortname %}
  {% include disqus_comments.html %}
{% endif %}


