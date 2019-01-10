
KVM Virtualization
===

# System

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.5 LTS
Release:	16.04
Codename:	xenial
<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ uname -r
4.4.0-131-generic
</pre>

Use the Advanced Packaging Tool (APT) to update the list of available packages and their versions (update) and install newer versions of the packages installed (upgrade).

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo apt update &amp;&amp; sudo apt upgrade -y
</pre>

## Physical CPU

An Intel or AMD 64-bit CPU that has virtualization extensions, VT-x for Intel and AMD-V for AMD. Determine if your CPU supports the virtualiztion extensions and check for the `svm`, `kvm`, and `lm` flags. The `vmx` flag means that the CPU has VT-x (`svm` is for AMD) and `lm` is for 64-bit support.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ grep --color -Ew &apos;svm|vmx|lm&apos; /proc/cpuinfo
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp <font color="#EF2929"><b>lm</b></font> constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl <font color="#EF2929"><b>vmx</b></font> smx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid dca sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch epb invpcid_single intel_pt ibrs ibpb stibp kaiser tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm cqm rdseed adx smap xsaveopt cqm_llc cqm_occup_llc cqm_mbm_total cqm_mbm_local dtherm ida arat pln pts
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp <font color="#EF2929"><b>lm</b></font> constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl <font color="#EF2929"><b>vmx</b></font> smx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid dca sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch epb invpcid_single intel_pt ibrs ibpb stibp kaiser tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm cqm rdseed adx smap xsaveopt cqm_llc cqm_occup_llc cqm_mbm_total cqm_mbm_local dtherm ida arat pln pts
...
</pre>

Verify KVM modules are loaded.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ lsmod | grep kvm
<font color="#EF2929"><b>kvm</b></font>_intel             172032  0
<font color="#EF2929"><b>kvm</b></font>                   548864  1 <font color="#EF2929"><b>kvm</b></font>_intel
irqbypass              16384  1 <font color="#EF2929"><b>kvm</b></font>
</pre>

> In addition to virtualization extensions, you may need to enable Intel VT-d or AMD IOMMU (AMD-Vi) in the BIOS. These are required for direct PCI device assignment to VMs, for example, to assign a physical NIC from the hypervisor to the VM.

### CPU Cores

If running server-class VMs, then one core per vCPU is recommended. When counting cores, do not count the hyperthreaded cores on the Intel CPUs, just actual cores. However, if running desktop-class VMs or less CPU-intensive VMs, then it is safe to overcommit the CPU since the performance takes a back seat and priority changes to VM density per hypervisor more than performance.

Run the `lscpu` command to see the CPU topology.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                40
On-line CPU(s) list:   0-39
Thread(s) per core:    2
Core(s) per socket:    10
Socket(s):             2
NUMA node(s):          2
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 79
Model name:            Intel(R) Xeon(R) CPU E5-2630 v4 @ 2.20GHz
Stepping:              1
CPU MHz:               1200.203
CPU max MHz:           3100.0000
CPU min MHz:           1200.0000
BogoMIPS:              4401.33
Virtualization:        VT-x
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              25600K
NUMA node0 CPU(s):     0-9,20-29
NUMA node1 CPU(s):     10-19,30-39
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid dca sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch epb invpcid_single intel_pt ibrs ibpb stibp kaiser tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm cqm rdseed adx smap xsaveopt cqm_llc cqm_occup_llc cqm_mbm_total cqm_mbm_local dtherm ida arat pln pts</pre>

## Physical Memory

Simple rule of thumb to decide how much memory is need for a physical node is to add up all the memory that is planned to be assigned to VMs and add an additional 2 GB of RAM for the hypervisor to use. 

Just like with physical CPUs, memory can be overcommitted for desktop-class VMs or test testing of VMs. View the available memory with the `free` command and the `-g` flag for GB.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ free -g
              total        used        free      shared  buff/cache   available
Mem:            251           0         250           0           0         250
Swap:             0           0           0
</pre>

## Storage

When considering storage space for the hypervisor, factor in the space required for the OS installation, Swap, and VMs disk usage. 

If not planning to do any memory overcommit then 16 GB of swap space can be used for systems between 64 and 256 GB of RAM. If planning to do overcommit then additional swap space should be added. For example if the memory overcommit ration is 50% (.5) more than the available physical RAM  then use the following formula: (RAM * 0.5) + Swap for OS = Swap space required for overcommitting. For example, with a system that has 256 GB RAM and planning to use a .5 overcommit ration, then Swap space required is (256 * .5) + 8 = 136 GB.

> Swap space in Linux is used when the amount of physical memory (RAM) is full. If the system needs more memory resources and the RAM is full, inactive pages in memory are moved to the swap space.

Verify the configured Swap space and how much is available.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ swapon --show
NAME      TYPE      SIZE USED PRIO
/dev/sda5 partition 976M   0B   -1
<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ free -m
              total        used        free      shared  buff/cache   available
Mem:         257848         599      255591           9        1658      256335
Swap:           975           0         975
</pre>

## libvirtd Service

The `libvirtd` daemon can be stoppped with `sudo systemctl stop libvirtd` and started with `sudo systemctl start libvirtd` commands. The libvirt service exposes an API to interact with `qemu-kvm` binary. Clients such as `virsh` and `virt-manager` use this API to communicate with `qemu-kvm` for VM life cycle management.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ libvirtd --version
libvirtd (libvirt) 1.3.1
</pre>

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo systemctl status libvirtd
<font color="#8AE234"><b>●</b></font> libvirt-bin.service - Virtualization daemon
   Loaded: loaded (/lib/systemd/system/libvirt-bin.service; enabled; vendor preset: enabled)
   Active: <font color="#8AE234"><b>active (running)</b></font> since Tue 2018-12-11 12:44:59 MST; 2h 30min ago
     Docs: man:libvirtd(8)
           http://libvirt.org
 Main PID: 1536 (libvirtd)
    Tasks: 18
   Memory: 50.5M
      CPU: 10.876s
   CGroup: /system.slice/libvirt-bin.service
           ├─1536 /usr/sbin/libvirtd
           ├─1782 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/lib/libvirt/libvirt_leaseshelper
           └─1783 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/lib/libvirt/libvirt_leaseshelper

Dec 11 12:45:10 Supermicro-813MFTQC dnsmasq[1782]: started, version 2.75 cachesize 150
Dec 11 12:45:10 Supermicro-813MFTQC dnsmasq[1782]: compile time options: IPv6 GNU-getopt DBus i18n IDN DHCP DHCPv6 no-Lua TFTP conntrack ipset auth DNSSEC loop-detect inot
Dec 11 12:45:10 Supermicro-813MFTQC dnsmasq-dhcp[1782]: DHCP, IP range 192.168.122.2 -- 192.168.122.254, lease time 1h
Dec 11 12:45:10 Supermicro-813MFTQC dnsmasq-dhcp[1782]: DHCP, sockets bound exclusively to interface virbr0
Dec 11 12:45:10 Supermicro-813MFTQC dnsmasq[1782]: reading /etc/resolv.conf
Dec 11 12:45:10 Supermicro-813MFTQC dnsmasq[1782]: using nameserver 192.168.188.1#53
Dec 11 12:45:10 Supermicro-813MFTQC dnsmasq[1782]: read /etc/hosts - 5 addresses
Dec 11 12:45:10 Supermicro-813MFTQC dnsmasq[1782]: read /var/lib/libvirt/dnsmasq/default.addnhosts - 0 addresses
Dec 11 12:45:10 Supermicro-813MFTQC dnsmasq-dhcp[1782]: read /var/lib/libvirt/dnsmasq/default.hostsfile
Dec 11 15:15:43 Supermicro-813MFTQC systemd[1]: Started Virtualization daemon.
</pre>

## Validate System Capabilities

Perform sanity checks on KVM capabilities to validate that the host is conifugred in a suitable way to run the libvirt hypervisor drivers using KVM virtualization.

It is the hardware virtualization support that helps KVM (qemu-kvm) VMs to have direct access to the physical CPU and helps it reach nearly native performance.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo virt-host-validate 
  QEMU: Checking for hardware virtualization                                 : <font color="#4E9A06">PASS</font>
  QEMU: Checking if device /dev/kvm exists                                   : <font color="#4E9A06">PASS</font>
  QEMU: Checking if device /dev/kvm is accessible                            : <font color="#4E9A06">PASS</font>
  QEMU: Checking if device /dev/vhost-net exists                             : <font color="#4E9A06">PASS</font>
  QEMU: Checking if device /dev/net/tun exists                               : <font color="#4E9A06">PASS</font>
  QEMU: Checking for cgroup &apos;memory&apos; controller support                      : <font color="#4E9A06">PASS</font>
  QEMU: Checking for cgroup &apos;memory&apos; controller mount-point                  : <font color="#4E9A06">PASS</font>
  QEMU: Checking for cgroup &apos;cpu&apos; controller support                         : <font color="#4E9A06">PASS</font>
  QEMU: Checking for cgroup &apos;cpu&apos; controller mount-point                     : <font color="#4E9A06">PASS</font>
  QEMU: Checking for cgroup &apos;cpuacct&apos; controller support                     : <font color="#4E9A06">PASS</font>
  QEMU: Checking for cgroup &apos;cpuacct&apos; controller mount-point                 : <font color="#4E9A06">PASS</font>
  QEMU: Checking for cgroup &apos;devices&apos; controller support                     : <font color="#4E9A06">PASS</font>
  QEMU: Checking for cgroup &apos;devices&apos; controller mount-point                 : <font color="#4E9A06">PASS</font>
  QEMU: Checking for cgroup &apos;net_cls&apos; controller support                     : <font color="#4E9A06">PASS</font>
  QEMU: Checking for cgroup &apos;net_cls&apos; controller mount-point                 : <font color="#4E9A06">PASS</font>
  QEMU: Checking for cgroup &apos;blkio&apos; controller support                       : <font color="#4E9A06">PASS</font>
  QEMU: Checking for cgroup &apos;blkio&apos; controller mount-point                   : <font color="#4E9A06">PASS</font>
  QEMU: Checking for device assignment IOMMU support                         : <font color="#4E9A06">PASS</font>
  QEMU: Checking if IOMMU is enabled by kernel                               : <font color="#C4A000">WARN</font> (IOMMU appears to be disabled in kernel. Add intel_iommu=on to kernel cmdline arguments)
   LXC: Checking for Linux &gt;= 2.6.26                                         : <font color="#4E9A06">PASS</font>
   LXC: Checking for namespace ipc                                           : <font color="#4E9A06">PASS</font>
   LXC: Checking for namespace mnt                                           : <font color="#4E9A06">PASS</font>
   LXC: Checking for namespace pid                                           : <font color="#4E9A06">PASS</font>
   LXC: Checking for namespace uts                                           : <font color="#4E9A06">PASS</font>
   LXC: Checking for namespace net                                           : <font color="#4E9A06">PASS</font>
   LXC: Checking for namespace user                                          : <font color="#4E9A06">PASS</font>
   LXC: Checking for cgroup &apos;memory&apos; controller support                      : <font color="#4E9A06">PASS</font>
   LXC: Checking for cgroup &apos;memory&apos; controller mount-point                  : <font color="#4E9A06">PASS</font>
   LXC: Checking for cgroup &apos;cpu&apos; controller support                         : <font color="#4E9A06">PASS</font>
   LXC: Checking for cgroup &apos;cpu&apos; controller mount-point                     : <font color="#4E9A06">PASS</font>
   LXC: Checking for cgroup &apos;cpuacct&apos; controller support                     : <font color="#4E9A06">PASS</font>
   LXC: Checking for cgroup &apos;cpuacct&apos; controller mount-point                 : <font color="#4E9A06">PASS</font>
   LXC: Checking for cgroup &apos;devices&apos; controller support                     : <font color="#4E9A06">PASS</font>
   LXC: Checking for cgroup &apos;devices&apos; controller mount-point                 : <font color="#4E9A06">PASS</font>
   LXC: Checking for cgroup &apos;net_cls&apos; controller support                     : <font color="#4E9A06">PASS</font>
   LXC: Checking for cgroup &apos;net_cls&apos; controller mount-point                 : <font color="#4E9A06">PASS</font>
   LXC: Checking for cgroup &apos;freezer&apos; controller support                     : <font color="#4E9A06">PASS</font>
   LXC: Checking for cgroup &apos;freezer&apos; controller mount-point                 : <font color="#4E9A06">PASS</font>
</pre>

`virsh` (virtualization shell) is a CLI for managing the VM and the hypervisor on a Linux system. It uses the libvirt management API and operates as an alternative to the GUI virt-manager.

`virsh` command classifications:
- Guest management (e.g. `start`, `stop`)
- Guest monitoring (e.g. `memstat`, `cpustat`)
- Host & hypervisor (e.g. `capabilities`, `nodeinfo`)
- Virtual networking (e.g. `net-list`, `net-define`)
- Storage management (e.g. `pool-list`, `pool-define`)
- Snapshot (e.g. `create-snapshot-as`)

See the system hardware architecture, CPU topology, memory size, and so on.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo virsh nodeinfo
CPU model:           x86_64
CPU(s):              40
CPU frequency:       1200 MHz
CPU socket(s):       1
Core(s) per socket:  10
Thread(s) per core:  2
NUMA cell(s):        2
Memory size:         264037316 KiB
</pre>

The `virsh domcapabilities` command displays an XML document describing the capabilities of qemu-kvm with respect to host and libvirt versions.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh domcapabilities
&lt;domainCapabilities&gt;
  &lt;path&gt;/usr/bin/kvm-spice&lt;/path&gt;
  &lt;domain&gt;qemu&lt;/domain&gt;
  &lt;machine&gt;pc-i440fx-xenial&lt;/machine&gt;
  &lt;arch&gt;x86_64&lt;/arch&gt;
  &lt;vcpu max=&apos;255&apos;/&gt;
  &lt;os supported=&apos;yes&apos;&gt;
    &lt;loader supported=&apos;yes&apos;&gt;
      &lt;enum name=&apos;type&apos;&gt;
        &lt;value&gt;rom&lt;/value&gt;
        &lt;value&gt;pflash&lt;/value&gt;
      &lt;/enum&gt;
      &lt;enum name=&apos;readonly&apos;&gt;
        &lt;value&gt;yes&lt;/value&gt;
        &lt;value&gt;no&lt;/value&gt;
      &lt;/enum&gt;
    &lt;/loader&gt;
  &lt;/os&gt;
  &lt;devices&gt;
    &lt;disk supported=&apos;yes&apos;&gt;
      &lt;enum name=&apos;diskDevice&apos;&gt;
        &lt;value&gt;disk&lt;/value&gt;
        &lt;value&gt;cdrom&lt;/value&gt;
        &lt;value&gt;floppy&lt;/value&gt;
        &lt;value&gt;lun&lt;/value&gt;
      &lt;/enum&gt;
      &lt;enum name=&apos;bus&apos;&gt;
        &lt;value&gt;ide&lt;/value&gt;
        &lt;value&gt;fdc&lt;/value&gt;
        &lt;value&gt;scsi&lt;/value&gt;
        &lt;value&gt;virtio&lt;/value&gt;
        &lt;value&gt;usb&lt;/value&gt;
      &lt;/enum&gt;
    &lt;/disk&gt;
    &lt;hostdev supported=&apos;yes&apos;&gt;
      &lt;enum name=&apos;mode&apos;&gt;
        &lt;value&gt;subsystem&lt;/value&gt;
      &lt;/enum&gt;
      &lt;enum name=&apos;startupPolicy&apos;&gt;
        &lt;value&gt;default&lt;/value&gt;
        &lt;value&gt;mandatory&lt;/value&gt;
        &lt;value&gt;requisite&lt;/value&gt;
        &lt;value&gt;optional&lt;/value&gt;
      &lt;/enum&gt;
      &lt;enum name=&apos;subsysType&apos;&gt;
        &lt;value&gt;usb&lt;/value&gt;
        &lt;value&gt;pci&lt;/value&gt;
        &lt;value&gt;scsi&lt;/value&gt;
      &lt;/enum&gt;
      &lt;enum name=&apos;capsType&apos;/&gt;
      &lt;enum name=&apos;pciBackend&apos;/&gt;
    &lt;/hostdev&gt;
  &lt;/devices&gt;
  &lt;features&gt;
    &lt;gic supported=&apos;no&apos;/&gt;
  &lt;/features&gt;
&lt;/domainCapabilities&gt;
</pre>

# Setup SSH

Generate your key using the `ssh-keygen` command. Next you are asked for the directory in which to save your key files, defaulting to `/home/${USER}/.ssh`. Next you are asked for a passphrase, which is optional. If you need to change your password the command `ssh-keygen -p` can be used.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>X1-Carbon-6th-Gen:</b></font><font color="#FCE94F"><b>~</b></font>$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/sean/.ssh/id_rsa): /home/sean/.ssh/mykey
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/sean/.ssh/mykey.
Your public key has been saved in /home/sean/.ssh/mykey.pub.
The key fingerprint is:
SHA256:VYDv6V/iDWcGPIwSxM075PcIVSee8erekHjScvHuh80 sean@X1-Carbon-6th-Gen
The key&apos;s randomart image is:
+---[RSA 2048]----+
|       ..+....+ .|
|       .o +... * |
|        .+.o  o .|
|         o*+.  . |
|        S..==oo  |
|         .o .*.+ |
|         .  * %+.|
|          .. #.+E|
|           .o o.=|
+----[SHA256]-----+</pre>

Two files are created `mykey` and `mykey.pub`. The `mykey` file is the private key and should never leave the machine, be given to another user, or be stored on any external media. If the private key leaks out, then your keys can no longer be trusted. By default, the private key is owned by the user that created it, with rw permissions given only to its owner. The public key `mykey.pub` has more lenient permissions, being readable by everyone and writable by the owner.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>X1-Carbon-6th-Gen:</b></font><font color="#FCE94F"><b>~</b></font>$ ls -l .ssh/
total 12K
-rw-r--r-- 1 sean  444 Dec 11 12:50 known_hosts
-rw------- 1 sean 1.8K Dec 11 17:24 mykey
-rw-r--r-- 1 sean  404 Dec 11 17:24 mykey.pub
</pre>

The public key is what gets copied to other servers to facilitate logging in via the SSH key-pair. When you log in to a server that has your public key, it checks that it’s a mathematical match to the private key on you machine, and if so then you are logged in. You’ll also be asked for your passphrase, if one was set. Use the ssh-copy-id command to transmit the public key to a target server.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>X1-Carbon-6th-Gen:</b></font><font color="#FCE94F"><b>~</b></font>$ ssh-copy-id -i ~/.ssh/mykey.pub 192.168.188.4
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: &quot;/home/sean/.ssh/mykey.pub&quot;
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
sean@192.168.188.4&apos;s password: 

Number of key(s) added: 1

Now try logging into the machine, with:   &quot;ssh &apos;192.168.188.4&apos;&quot;
and check to make sure that only the key(s) you wanted were added.
</pre>

The contents of `~/.ssh/mykey.pub` on your machine is copied into the `~/.ssh/authorized_keys` file on the target server and SSH checks the contents of it when connecting, looking for a key that matches the private key (`~/.ssh/mykey`) on your local machine. If the two keys are a mathematical match, you are allowed access. If you set up a passphrase, you'll be asked to enter it in order to open your public key. With an SSH agent, you can actually cache your passphrase the first time you use it, so you won't be asked for it with every connection. To start the SSH agent, type `eval $(ssh-agent)`.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>X1-Carbon-6th-Gen:</b></font><font color="#FCE94F"><b>~</b></font>$ eval $(ssh-agent)
Agent pid 20690
</pre>

This command will start an SSH agent, that will continue to run in the background of your shell. Then, you can unlock your key for your agent to use with the `ssh-add` command.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>X1-Carbon-6th-Gen:</b></font><font color="#FCE94F"><b>~</b></font>$ ssh-add ~/.ssh/mykey
Enter passphrase for /home/sean/.ssh/mykey: 
Identity added: /home/sean/.ssh/mykey (/home/sean/.ssh/mykey)
</pre>

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>X1-Carbon-6th-Gen:</b></font><font color="#FCE94F"><b>~</b></font>$ ssh 192.168.188.4
Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.4.0-131-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 packages can be updated.
0 updates are security updates.

New release &apos;18.04.1 LTS&apos; available.
Run &apos;do-release-upgrade&apos; to upgrade to it.


*** System restart required ***
Last login: Tue Dec 11 16:44:51 2018 from 192.168.188.223
<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ </pre>

# Virtual Machine Manager

The `virt-manager` utility provides the ability to manage both local and remote KVM servers. To create a new connection to a remote external server, click on **File** and select **Add Connection**. A new screen will appear, where the connection details can be filled out. Check the **Connect to remote host** if not connecting to the local host (in which case you can leave the defaults). For the remote connection select the **Method** as **SSH** and fill out the **Username**, **Hostname** of the server where KVM was installed on and if **Autoconnect** if desired. For a remote connection to work the username included here will need to be able to access the server via SSH, have permissions to the hypervisor and the `libvirtd` unit must be running on the server. Press **Connect** to initiate the SSH connection and you may be prompted for the username password to your KVM server; type that in and press **OK**.

![](https://i.imgur.com/ce8dCtQ.png)

The **Overview** tab gives basic information on the libvirt connection URI, CPU, and memory usage pattern of the host system.

![](https://i.imgur.com/tBNqJ6f.png)

## Virtual Networks Tab

The **Virtual Networks** tab allows configuration of various types of virtual networks (VN) and monitor their status. There are three types of VNs:
1. NAT'd
2. Routed
3. Isolated

### NAT'd Mode

NAT'd mode provides outbound network connectivity to attached VMs. This means that VMs can communicate with the outside network based on the network connectivity with the host. But none of the outside entities are able to communicate with the VMs.

By default a VM will get an IP address in the 192.168.122.0/24 network (`virbr0` interface), within the range of .2 to .254.

![](https://i.imgur.com/c9Lyuei.png)

Use the `virsh net-list --all` to list both active and inactive VNs. Without `--all` only active VNs are listed.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-list --all
 Name                 State      Autostart     Persistent
&#45;---------------------------------------------------------
 default              active     yes           yes
</pre>

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-info default
Name:           default
UUID:           74c996f0-5929-49f4-8d02-aabecb729901
Active:         yes
Persistent:     yes
Autostart:      yes
Bridge:         virbr0
</pre>

Virtual network configuration files are stored in `/etc/libvirt/qemu/networks` as XML files. The `virsh net-destroy <VN>` command will stop a VN while `virsh net-start <VN>` will start the VN. 

:::danger
DO NOT use the the `net-destroy` when VMs are active using the VN as it will break network connectivity for the VM.
:::

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-dumpxml default
&lt;network&gt;
  &lt;name&gt;default&lt;/name&gt;
  &lt;uuid&gt;74c996f0-5929-49f4-8d02-aabecb729901&lt;/uuid&gt;
  &lt;forward mode=&apos;nat&apos;&gt;
    &lt;nat&gt;
      &lt;port start=&apos;1024&apos; end=&apos;65535&apos;/&gt;
    &lt;/nat&gt;
  &lt;/forward&gt;
  &lt;bridge name=&apos;virbr0&apos; stp=&apos;on&apos; delay=&apos;0&apos;/&gt;
  &lt;mac address=&apos;52:54:00:0c:0d:6f&apos;/&gt;
  &lt;ip address=&apos;192.168.122.1&apos; netmask=&apos;255.255.255.0&apos;&gt;
    &lt;dhcp&gt;
      &lt;range start=&apos;192.168.122.2&apos; end=&apos;192.168.122.254&apos;/&gt;
    &lt;/dhcp&gt;
  &lt;/ip&gt;
&lt;/network&gt;
</pre>

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ ifconfig virbr0 
virbr0    Link encap:Ethernet  HWaddr 52:54:00:0c:0d:6f  
          inet addr:192.168.122.1  Bcast:192.168.122.255  Mask:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
</pre>

### Routed Mode

With routed mode, the virtual switch is connected to the physical host LAN, passing guest network traffic back and forth without using NAT. The virtual switch sees the IP addresses in each packet, using that information when deciding what to do.

In this mode all VMs are in a subnet routed through the virtual switch. This on its own is not sufficient. because no other hosts on the physical network know this subnet exists or how to reach it. It is thus necessary to configure routers in the physical network (e.g. using a static route).

![](https://i.imgur.com/RpW6b99.png)

### Isolated Mode

In this mode, guests connected to the virtual switch can communicate with each other, and with the host. However, their traffic will not pass outside of the host, nor can they receive traffic from outside the host. 

![](https://i.imgur.com/7vjDhd8.png)

## Storage Tab

The **Storage Tab** provides details of the storage pools available and is just a store for save VM disk images. Different sources such as directory and LVM can be used.

The default storage pool is in `/var/lib/libvirt/images`

![](https://i.imgur.com/mQ2XcPs.png)

Stop the `libvirtd` service as additional configuraiton will be done.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo systemctl stop libvirtd
</pre>

Ensure a group named `kvm` exist that will be used to allow members of the group to manage VMs. If the group is not created then perform the `sudo groupadd kvm` command.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ more /etc/group | grep kvm
<font color="#EF2929"><b>kvm</b></font>:x:111:
</pre>

Make the `root` user and the `kvm` group the owner of the `/var/lib/libvirt/images` directory.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo chown root:kvm /var/lib/libvirt/images
</pre>

Set the permissions of `/var/lib/libvirt/images` so that anyone in the `kvm` group will be able to modify its contents. Remember that `d` stands for directory and the permissions for user, group, and other are grouped in three sets with `r` (read), `w` write, `x` execute permissions.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo chmod g+rw /var/lib/libvirt/images
</pre>

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo ls -la /var/lib/libvirt | grep images
drwxrwx--x  2 root         kvm  4096 May 23  2018 <font color="#EF2929"><b>images</b></font>
</pre>

Set the primary user account in use on the server to be a member of the `kvm` group in order to manage VMs without switching to `root` first. Log out and log in again after executing the command, so the changes take effect.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo usermod -aG kvm sean
</pre>

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ more /etc/group | grep kvm
<font color="#EF2929"><b>kvm</b></font>:x:111:sean</pre>

Start the `libvirtd` service and check the status for errors.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo systemctl start libvirtd
<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo systemctl status libvirtd
<font color="#8AE234"><b>●</b></font> libvirt-bin.service - Virtualization daemon
   Loaded: loaded (/lib/systemd/system/libvirt-bin.service; enabled; vendor preset: enabled)
   Active: <font color="#8AE234"><b>active (running)</b></font> since Tue 2018-12-11 18:36:27 MST; 14s ago
     Docs: man:libvirtd(8)
           http://libvirt.org
 Main PID: 1924 (libvirtd)
    Tasks: 19
   Memory: 42.0M
      CPU: 14.342s
   CGroup: /system.slice/libvirt-bin.service
           ├─1782 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/lib/libvirt/libvirt_leaseshelper
           ├─1783 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/lib/libvirt/libvirt_leaseshelper
           └─1924 /usr/sbin/libvirtd

Dec 11 18:36:27 Supermicro-813MFTQC systemd[1]: Starting Virtualization daemon...
Dec 11 18:36:27 Supermicro-813MFTQC systemd[1]: Started Virtualization daemon.
Dec 11 18:36:41 Supermicro-813MFTQC dnsmasq[1782]: read /etc/hosts - 5 addresses
Dec 11 18:36:41 Supermicro-813MFTQC dnsmasq[1782]: read /var/lib/libvirt/dnsmasq/default.addnhosts - 0 addresses
Dec 11 18:36:41 Supermicro-813MFTQC dnsmasq-dhcp[1782]: read /var/lib/libvirt/dnsmasq/default.hostsfile
</pre>

Copy the CentOS minimal ISO file to the /var/lib/libvirt/images/iso, which installs only a very small base set of packages and is the smallest download of CentOS available.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>X1-Carbon-6th-Gen:</b></font><font color="#FCE94F"><b>~/Downloads/Software</b></font>$ scp CentOS-7-x86_64-Minimal-1804.iso 192.168.188.4:/var/lib/libvirt/images/.
CentOS-7-x86_64-Minimal-1804.iso                                                                                                         100%  906MB   8.1MB/s   01:51 </pre>

![](https://i.imgur.com/CPRLf8v.png)

## Create a VM

In `virt-manager`, right-click your server connection and click on **New** to start the creation of a VM. The default selection will be on **Local install media (ISO image or CDROM)**; leave this selection and click on **Forward**.

![](https://i.imgur.com/JE9Kikn.png)

Click on **Browse** to open up another window where you can select an ISO image under the default filesystem directory. Click on **Choose Volume** and then **Forward**.

![](https://i.imgur.com/AEyxGb4.png)

![](https://i.imgur.com/QracCfn.png)

![](https://i.imgur.com/Ht2hwb2.png)

Next, allocate RAM and CPU resources to the VM. For most Linux distributions with no graphical user interface, 512 MB is plenty (unless your workload demands more). Click on **Forward**.

![](https://i.imgur.com/zSmTZr6.png)

Next, allocate free disk space for the VM's virtual hard disk. This space won't be used up all at once; by default, KVM utilizes thin provisioning that basically just fills up the virtual disk as the VM needs space. Click on **Forward**. 

![](https://i.imgur.com/MZtiy0d.png)

Lastly, name the VM. This won't be the hostname of the virtual machine; it's just the name you'll see when you see the VM listed in `virt-manager`. Selecting **Customize configuration before install** box will open another wizard that provides the ability to add, remove, and configure VM hardware settings. Click on **Finish**, the VM will start and it will automatically boot into the install ISO that was attached to the VM near the beginning of the process.

![](https://i.imgur.com/B1JfIVl.png)

![](https://i.imgur.com/fTueMkd.png)

When you click on the VM window, it may steal the keyboard and mouse and dedicate it to the window. Press Ctrl + Alt at the same time to release this control and regain full control of the keyboard and mouse.

![](https://i.imgur.com/QGQAc6r.png)

![](https://i.imgur.com/yQwTkz3.png)

> The default MAC address range used by libvirt is `52:54:00`.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-dhcp-leases default
 Expiry Time          MAC address        Protocol  IP address                Hostname        Client ID or DUID
&#45;------------------------------------------------------------------------------------------------------------------
 2018-12-11 20:13:26  52:54:00:72:38:e0  ipv4      192.168.122.17/24         centos7         -
</pre>

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ ssh 192.168.122.17
The authenticity of host &apos;192.168.122.17 (192.168.122.17)&apos; can&apos;t be established.
ECDSA key fingerprint is SHA256:ojUEdhup2GDnJ+hq4UiFzOEOkMZRgb+8M+lmeeXA8CU.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added &apos;192.168.122.17&apos; (ECDSA) to the list of known hosts.
sean@192.168.122.17&apos;s password: 
Last login: Tue Dec 11 19:14:14 2018
[sean@centos7 ~]$</pre>

<pre>[sean@centos7 ~]$ ip a
1: lo: &lt;LOOPBACK,UP,LOWER_UP&gt; mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:72:38:e0 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.17/24 brd 192.168.122.255 scope global noprefixroute dynamic eth0
       valid_lft 3171sec preferred_lft 3171sec
    inet6 fe80::f484:1b6:1edb:a0f2/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
</pre>

<pre>[sean@centos7 ~]$ ping google.com -c 3
PING google.com (172.217.11.238) 56(84) bytes of data.
64 bytes from den02s01-in-f14.1e100.net (172.217.11.238): icmp_seq=1 ttl=55 time=36.0 ms
64 bytes from den02s01-in-f14.1e100.net (172.217.11.238): icmp_seq=2 ttl=55 time=25.3 ms
64 bytes from den02s01-in-f14.1e100.net (172.217.11.238): icmp_seq=3 ttl=55 time=25.3 ms

--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 25.307/28.915/36.085/5.071 ms
</pre>

## Using virt-install (Desktop)

`virt-install` is an alternative CLI tool that is scripting-friendly and supports graphical installations that can be used to setup guest and then start the install process. 

Before starting the OS install using the `virt-install` command, it is necessary to create a virtual disk using the `qemu-img` command. 

:::info
Notice this is performed on a Ubuntu desktop.
:::

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>X1-Carbon-6th-Gen:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo qemu-img create -f raw -o size=9G /var/lib/libvirt/qemu/centos7.img 
Formatting &apos;/var/lib/libvirt/qemu/centos7.img&apos;, fmt=raw size=9663676416</pre>

Start the installation with the `virt-install` command and it will launch virt-viewer.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>X1-Carbon-6th-Gen:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo virt-install \
&gt; --name centos7.0 \
&gt; --ram 512 \
&gt; --disk path=/var/lib/libvirt/qemu/centos7.img \
&gt; --vcpus 1 \
&gt; --os-type Linux \
&gt; --os-variant CentOS7.0 \
&gt; --network bridge=virbr0 \
&gt; --graphics vnc,port=5999 \
&gt; --console pty,target_type=serial \
&gt; --cdrom /var/lib/libvirt/images/CentOS-7-x86_64-Minimal-1804.iso</pre>

# Virtual Networking

The main component of libvirt networking is the virtual switch, also known as the ++bridge++. Interfaces connecting VMs to a bridge are called ++TAP devices++. The TAP interface is part of the TUN/TAP implementation available in the Linux kernal. TAP (network tap) simulates a link layer device and operates at OSI L2 for Ethernet frames. TUN stands for "tunnel" and operates at OSI L3 for IP packets.

## Create Bridge & TAP Interface

The `brctl` command is provided by the package `bridge-utils`. Create a bridge called "test". The `brctl show` command will list all available bridges on the server along with the ID of the bridge, STP status, and interfaces attached to it.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo brctl addbr test
<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ brctl show
bridge name	bridge id		STP enabled	interfaces
test		8000.000000000000	no		
virbr0		8000.5254000c0d6f	yes		virbr0-nic
</pre>

A Linux bridge will show up as a network device using the `ip` command.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ ip -c link show test
9: <font color="#06989A">test</font>: &lt;BROADCAST,MULTICAST&gt; mtu 1500 qdisc noop state <font color="#CC0000">DOWN </font>mode DEFAULT group default qlen 1000
    link/ether <font color="#C4A000">56:7c:eb:34:d7:19</font> brd ff:ff:ff:ff:ff:ff
</pre>

Likewise, `ifconfig` can be used to check and configure network settings for a Linux bridge. `ifconfig` is relatively easy to read and understand but is not as feature-rich as the `ip` command.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ ifconfig test
test      Link encap:Ethernet  HWaddr 56:7c:eb:34:d7:19  
          BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

</pre>

Create a tap device named "vm-nic" using the TUN/TAP device module.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ ip tuntap add dev vm-vnic mode tap
<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ ip -c link show vm-vnic
10: <font color="#06989A">vm-vnic</font>: &lt;BROADCAST,MULTICAST&gt; mtu 1500 qdisc noop state <font color="#CC0000">DOWN </font>mode DEFAULT group default qlen 1000
    link/ether <font color="#C4A000">d6:4e:9a:24:99:ae</font> brd ff:ff:ff:ff:ff:ff
</pre>

Add the tap device "vm-vnic" to the bridge "test".

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo brctl addif test vm-vnic
<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ brctl show
bridge name	bridge id		STP enabled	interfaces
test		8000.d64e9a2499ae	no		vm-vnic
virbr0		8000.5254000c0d6f	yes		virbr0-nic
</pre>

## Delete TAP Interface & Bridge

Remove the "vm-nic" tap device from the "test" bridge.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo brctl delif test vm-vnic
<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ brctl show test
bridge name	bridge id		STP enabled	interfaces
test		8000.000000000000	no		</pre>

Once the "vm-vnic" is removed from the bridge, remote the tap device using the `ip` command.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ ip tuntap del dev vm-vnic mode tap
</pre>

Finally, remove the "test" bridge.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo brctl delbr test; echo $?
0
</pre>

## Create Isolated Bridge

Use the `virsh` command to build an isolated network by creating an XML file with the following contents and save it as "isolated.xml". Here:
- `<network>`: is used for defining the virtual net.
- `<name>`: is used for defining the name of the virtual net an is called "isolated".

```xml=
<network>
   <name>isolated</name>
</network>
```

To create the network using the XML file use the `virsh net-define` command followed by the path of the XML file. Here libvirt added additional parameters including:
- `<uuid>`: unique ID for a bridge.
- `<bridge>`: is used to define bridge details such as the name of the bridge is `virbr1`, with STP on, and a delay of 0. These are the same options that can be controlled with the `brctl` command options with STP set by `stp` and delay by `setfd`.
- `<mac>`: is the MAC address assigned to the bridge.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-define isolated.xml
Network isolated defined from isolated.xml
<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-list --all
 Name                 State      Autostart     Persistent
&#45;---------------------------------------------------------
 default              active     yes           yes
 isolated             inactive   no            yes
</pre>

View the XML file that libvirt created using the `virsh net-dumpxml` command.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-dumpxml isolated
&lt;network&gt;
  &lt;name&gt;isolated&lt;/name&gt;
  &lt;uuid&gt;2213fd59-2603-496a-9cf1-7f27d7fc220f&lt;/uuid&gt;
  &lt;bridge name=&apos;virbr1&apos; stp=&apos;on&apos; delay=&apos;0&apos;/&gt;
  &lt;mac address=&apos;52:54:00:67:27:65&apos;/&gt;
&lt;/network&gt;
</pre>

The configuration file is stored in `/etc/libvirt/qemu/networks/` as an XML file with the given name of the virtual network.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo cat /etc/libvirt/qemu/networks/isolated.xml 
&lt;!--
WARNING: THIS IS AN AUTO-GENERATED FILE. CHANGES TO IT ARE LIKELY TO BE
OVERWRITTEN AND LOST. Changes to this xml configuration should be made using:
  virsh net-edit isolated
or other application using the libvirt API.
--&gt;

&lt;network&gt;
  &lt;name&gt;isolated&lt;/name&gt;
  &lt;uuid&gt;2213fd59-2603-496a-9cf1-7f27d7fc220f&lt;/uuid&gt;
  &lt;bridge name=&apos;virbr1&apos; stp=&apos;on&apos; delay=&apos;0&apos;/&gt;
  &lt;mac address=&apos;52:54:00:67:27:65&apos;/&gt;
&lt;/network&gt;
</pre>

Activate the isolated network using the `virsh net-start` command and auto-start with the `virsh net-autostart` command.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-start isolated
Network isolated started

<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-autostart isolated
Network isolated marked as autostarted

<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-list --all
 Name                 State      Autostart     Persistent
&#45;---------------------------------------------------------
 default              active     yes           yes
 isolated             active     yes           yes
</pre>

### Add Virtio vNIC

A virtio vNIC can be added while a VM is running and will be ready to use inside the VM immediately.

Get the details of the current vNIC attached to VM "c1" using the `virsh domiflist` command.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh domiflist c2
Interface  Type       Source     Model       MAC
&#45;------------------------------------------------------
vnet1      network    default    virtio      52:54:00:46:1f:75</pre>

Attach a new vNIC to VM "c2" using the `virsh attach-interface` command. Two new options include:
- `--config`: makes the change persistent in the next startup of the VM.
- `--live`: informs libvirt a NIC is being attached to a live VM.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh attach-interface --domain c2 --source isolated --type network --model virtio --config --live
Interface attached successfully

<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh domiflist c2
Interface  Type       Source     Model       MAC
&#45;------------------------------------------------------
vnet1      network    default    virtio      52:54:00:46:1f:75
vnet2      network    isolated   virtio      52:54:00:11:d7:0b
</pre>

The `virbr1-nic` interface is created by libvirt when it starts `virbr1`. The purpose of this interface is to provide a consistent and reliable MAC address for the bridge.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ brctl show virbr1
bridge name	bridge id		STP enabled	interfaces
virbr1		8000.525400672765	yes		virbr1-nic
							vnet2
</pre>

### Test Isolated Bridge

Connect VM "c1" to VM "c2" over the isolated network and assign IPs to the newly added interfaces and verify that reachability is established with the `ping` command. 

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-dhcp-leases default
 Expiry Time          MAC address        Protocol  IP address                Hostname        Client ID or DUID
&#45;------------------------------------------------------------------------------------------------------------------
 2018-12-11 23:11:45  52:54:00:46:1f:75  ipv4      192.168.122.14/24         c2              -
 2018-12-11 23:16:07  52:54:00:8a:42:bb  ipv4      192.168.122.162/24        c1              -
</pre>

<pre>[sean@c1 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-eth1
TYPE=&quot;Ethernet&quot;
BOOTPROTO=&quot;static&quot;
IPV4_FAILURE_FATAL=&quot;no&quot;
NAME=&quot;eth1&quot;
DEVICE=&quot;eth1&quot;
ONBOOT=&quot;yes&quot;
IPADDR=&quot;192.168.1.100&quot;
PREFIX=&quot;24&quot;
</pre>

<pre>[sean@c1 ~]$ systemctl restart network
<font color="#EF2929"><b>==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===</b></font>
Authentication is required to manage system services or units.
Authenticating as: Sean (sean)
Password: 
<font color="#EF2929"><b>==== AUTHENTICATION COMPLETE ===</b></font>
</pre>

<pre>[sean@c1 ~]$ systemctl status network
<font color="#8AE234"><b>●</b></font> network.service - LSB: Bring up/down networking
   Loaded: loaded (/etc/rc.d/init.d/network; bad; vendor preset: disabled)
   Active: <font color="#8AE234"><b>active (exited)</b></font> since Tue 2018-12-11 22:47:20 MST; 3s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 9707 ExecStop=/etc/rc.d/init.d/network stop (code=exited, status=0/SUCCESS)
  Process: 9944 ExecStart=/etc/rc.d/init.d/network start (code=exited, status=0/SUCCESS)

Dec 11 22:47:20 c1 systemd[1]: Starting LSB: Bring up/down networking...
Dec 11 22:47:20 c1 network[9944]: Bringing up loopback interface:  [  OK  ]
Dec 11 22:47:20 c1 network[9944]: Bringing up interface eth0:  Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/21)
Dec 11 22:47:20 c1 network[9944]: [  OK  ]
Dec 11 22:47:20 c1 network[9944]: Bringing up interface eth1:  Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/22)
Dec 11 22:47:20 c1 network[9944]: [  OK  ]
Dec 11 22:47:20 c1 systemd[1]: Started LSB: Bring up/down networking.
</pre>

<pre>[sean@c1 ~]$ ip -c addr show dev eth1
3: <font color="#06989A">eth1</font>: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500 qdisc pfifo_fast state <font color="#4E9A06">UP </font>group default qlen 1000
    link/ether <font color="#C4A000">52:54:00:18:6b:4b</font> brd <font color="#C4A000">ff:ff:ff:ff:ff:ff</font>
    inet <font color="#75507B">192.168.1.100</font>/24 brd <font color="#75507B">192.168.1.255 </font>scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 <font color="#3465A4">fe80::5054:ff:fe18:6b4b</font>/64 scope link 
       valid_lft forever preferred_lft forever
</pre>

<pre>[sean@c1 ~]$ ping 192.168.1.200 -c 3
PING 192.168.1.200 (192.168.1.200) 56(84) bytes of data.
64 bytes from 192.168.1.200: icmp_seq=1 ttl=64 time=0.855 ms
64 bytes from 192.168.1.200: icmp_seq=2 ttl=64 time=0.400 ms
64 bytes from 192.168.1.200: icmp_seq=3 ttl=64 time=0.412 ms

--- 192.168.1.200 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2001ms
rtt min/avg/max/mdev = 0.400/0.555/0.855/0.213 ms
</pre>

<pre>[sean@c1 ~]$ ip n show dev eth1
192.168.1.200 lladdr 52:54:00:11:d7:0b DELAY</pre>

Log into the "c2" VM and verify the MAC address.

<pre>[sean@c2 ~]$ ip -c addr show dev eth1
3: <font color="#06989A">eth1</font>: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500 qdisc pfifo_fast state <font color="#4E9A06">UP </font>group default qlen 1000
    link/ether <font color="#C4A000">52:54:00:11:d7:0b</font> brd <font color="#C4A000">ff:ff:ff:ff:ff:ff</font>
    inet <font color="#75507B">192.168.1.200</font>/24 brd <font color="#75507B">192.168.1.255 </font>scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 <font color="#3465A4">fe80::5054:ff:fe11:d70b</font>/64 scope link 
       valid_lft forever preferred_lft forever
</pre>

## Create Routed Bridge

Create an XML configuration file called "routed.xml".

```xml=
<network>
  <name>routed</name>
  <forward dev='eno1' mode='route'>
    <interface dev='eno1'/>
  </forward>
  <ip address='192.168.2.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.2.128' end='192.168.2.254'/>
    </dhcp>
  </ip>
</network>
```

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-define routed.xml
Network routed defined from routed.xml

<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-list --all
 Name                 State      Autostart     Persistent
&#45;---------------------------------------------------------
 default              active     yes           yes
 isolated             active     yes           yes
 routed               inactive   no            yes
</pre>

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-start routed
Network routed started

<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-autostart routed
Network routed marked as autostarted

<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-list --all
 Name                 State      Autostart     Persistent
&#45;---------------------------------------------------------
 default              active     yes           yes
 isolated             active     yes           yes
 routed               active     yes           yes
</pre>

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-dumpxml routed
&lt;network connections=&apos;1&apos;&gt;
  &lt;name&gt;routed&lt;/name&gt;
  &lt;uuid&gt;eebe92aa-6e01-4559-8964-57b02d11395d&lt;/uuid&gt;
  &lt;forward dev=&apos;eno1&apos; mode=&apos;route&apos;&gt;
    &lt;interface dev=&apos;eno1&apos;/&gt;
  &lt;/forward&gt;
  &lt;bridge name=&apos;virbr2&apos; stp=&apos;on&apos; delay=&apos;0&apos;/&gt;
  &lt;mac address=&apos;52:54:00:e3:bd:c4&apos;/&gt;
  &lt;ip address=&apos;192.168.2.1&apos; netmask=&apos;255.255.255.0&apos;&gt;
    &lt;dhcp&gt;
      &lt;range start=&apos;192.168.2.128&apos; end=&apos;192.168.2.254&apos;/&gt;
    &lt;/dhcp&gt;
  &lt;/ip&gt;
&lt;/network&gt;
</pre>

Check that network is marked as persistent and autostart.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-info routed
Name:           routed
UUID:           eebe92aa-6e01-4559-8964-57b02d11395d
Active:         yes
Persistent:     yes
Autostart:      yes
Bridge:         virbr2
</pre>

### Test Routed Network

On the "c1" VM on the host server (Supermicro-813MFTQC) setup the eth1 interface to belong to the routed network (192.168.2/24) and set it for DHCP. Then turn down the eth0 interface that is attached to the default NAT'd network.

![](https://i.imgur.com/0U1Ku9N.png)

<pre>[sean@c1 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-eth1
TYPE=&quot;Ethernet&quot;
BOOTPROTO=&quot;dhcp&quot;
IPV4_FAILURE_FATAL=&quot;no&quot;
NAME=&quot;eth1&quot;
DEVICE=&quot;eth1&quot;
ONBOOT=&quot;yes&quot;
</pre>

<pre>[sean@c1 ~]$ systemctl restart network
<font color="#EF2929"><b>==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===</b></font>
Authentication is required to manage system services or units.
Authenticating as: Sean (sean)
Password: 
<font color="#EF2929"><b>==== AUTHENTICATION COMPLETE ===</b></font>
</pre>

<pre>[sean@c1 ~]$ sudo ifdown eth0
Device &apos;eth0&apos; successfully disconnected.
</pre>

<pre>[sean@c1 ~]$ ip -c address show dev eth1
3: <font color="#06989A">eth1</font>: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500 qdisc pfifo_fast state <font color="#4E9A06">UP </font>group default qlen 1000
    link/ether <font color="#C4A000">52:54:00:18:6b:4b</font> brd <font color="#C4A000">ff:ff:ff:ff:ff:ff</font>
    inet <font color="#75507B">192.168.2.133</font>/24 brd <font color="#75507B">192.168.2.255 </font>scope global noprefixroute dynamic eth1
       valid_lft 3493sec preferred_lft 3493sec
    inet6 <font color="#3465A4">fe80::5054:ff:fe18:6b4b</font>/64 scope link 
       valid_lft forever preferred_lft forever</pre>

<pre>[sean@c1 ~]$ ip route
default via 192.168.2.1 dev eth1 proto dhcp metric 101 
192.168.2.0/24 dev eth1 proto kernel scope link src 192.168.2.133 metric 101 
</pre>

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-dhcp-leases routed
 Expiry Time          MAC address        Protocol  IP address                Hostname        Client ID or DUID
&#45;------------------------------------------------------------------------------------------------------------------
 2018-12-12 21:12:16  52:54:00:18:6b:4b  ipv4      192.168.2.133/24          c1              -
</pre>

Setup a static route for the "routed" network 192.168.2/24 back to the host server (Supermicro-813MFTQC) on the LAN at 192.168.188.4 on the target LAN device (X1-Carbon-6th-Gen) that is in the 192.168.188/24 network.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>X1-Carbon-6th-Gen:</b></font><font color="#FCE94F"><b>~</b></font>$ ip -c address show dev wlp2s0
3: <font color="#06989A">wlp2s0</font>: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500 qdisc mq state <font color="#4E9A06">UP </font>group default qlen 1000
    link/ether <font color="#C4A000">74:e5:f9:65:08:79</font> brd <font color="#C4A000">ff:ff:ff:ff:ff:ff</font>
    inet <font color="#75507B">192.168.188.223</font>/24 brd <font color="#75507B">192.168.188.255 </font>scope global dynamic noprefixroute wlp2s0
       valid_lft 41881sec preferred_lft 41881sec
    inet6 <font color="#3465A4">fe80::2232:36a9:1595:7a59</font>/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
</pre>

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>X1-Carbon-6th-Gen:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo ip route add 192.168.2.0/24 via 192.168.188.4
</pre>

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>X1-Carbon-6th-Gen:</b></font><font color="#FCE94F"><b>~</b></font>$ ip route get 192.168.2.0/24
192.168.2.0 via 192.168.188.4 dev wlp2s0 src 192.168.188.223 uid 1000 
    cache 
</pre>

Perform a ping from VM "c1" on the host server (Supermicro-813MFTQC) to 192.168.188.223 (X1-Carbon-6th-Gen) in order to verify reachability. Perform a `tcpdump` on the target device (X1-Carbon-6th-Gen) to capture the ICMP pings.

<pre>[sean@c1 ~]$ ping 192.168.188.223 -c 3
PING 192.168.188.223 (192.168.188.223) 56(84) bytes of data.
64 bytes from 192.168.188.223: icmp_seq=1 ttl=63 time=1.06 ms
64 bytes from 192.168.188.223: icmp_seq=2 ttl=63 time=1.24 ms
64 bytes from 192.168.188.223: icmp_seq=3 ttl=63 time=1.37 ms

--- 192.168.188.223 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 1.064/1.226/1.375/0.130 ms</pre>

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>X1-Carbon-6th-Gen:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo tcpdump -nni wlp2s0 icmp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on wlp2s0, link-type EN10MB (Ethernet), capture size 262144 bytes
19:40:09.776932 IP 192.168.2.133 &gt; 192.168.188.223: ICMP echo request, id 3898, seq 1, length 64
19:40:09.777009 IP 192.168.188.223 &gt; 192.168.2.133: ICMP echo reply, id 3898, seq 1, length 64
19:40:10.778410 IP 192.168.2.133 &gt; 192.168.188.223: ICMP echo request, id 3898, seq 2, length 64
19:40:10.778492 IP 192.168.188.223 &gt; 192.168.2.133: ICMP echo reply, id 3898, seq 2, length 64
19:40:11.779847 IP 192.168.2.133 &gt; 192.168.188.223: ICMP echo request, id 3898, seq 3, length 64
19:40:11.779927 IP 192.168.188.223 &gt; 192.168.2.133: ICMP echo reply, id 3898, seq 3, length 64
^C
6 packets captured
6 packets received by filter
0 packets dropped by kernel</pre>

Likewise confirm reachability from the target (X1-Carbon-6th-Gen) device, which is on the 192.168.188/24 network back to VM "c1" (192.168.2.133) on the server host (Supermicro-813MFTQC).

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>X1-Carbon-6th-Gen:</b></font><font color="#FCE94F"><b>~</b></font>$ ping 192.168.2.133 -c 3
PING 192.168.2.133 (192.168.2.133) 56(84) bytes of data.
64 bytes from 192.168.2.133: icmp_seq=1 ttl=63 time=1.79 ms
64 bytes from 192.168.2.133: icmp_seq=2 ttl=63 time=3.65 ms
64 bytes from 192.168.2.133: icmp_seq=3 ttl=63 time=1.38 ms

--- 192.168.2.133 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 1.389/2.279/3.650/0.983 ms
</pre>

### Edit Virtual Network

Edit the routed virtual network so that packets from the VMs can be forwarded to any interface available on the host based on IP routes rules specified on the host.

The VM "c1" was stopped at this point in order to manipulate the "routed" network. Then it is okay to stop the VN using the `virsh net-destroy` command.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-destroy routed
Network routed destroyed
</pre>

Edit the network using `virsh net-edit` command. A temporary copy of the configuration file is placed in the `/tmp` directory. The `<forward>` tag will be modified. Verify the changes with the `virsh net-dumpxml` command.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-edit routed

Select an editor.  To change later, run &apos;select-editor&apos;.
  1. /bin/ed
  2. /bin/nano        &lt;---- easiest
  3. /usr/bin/vim.basic
  4. /usr/bin/vim.tiny

Choose 1-4 [2]: 2
Network routed XML configuration edited.
</pre>

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-dumpxml routed
&lt;network&gt;
  &lt;name&gt;routed&lt;/name&gt;
  &lt;uuid&gt;eebe92aa-6e01-4559-8964-57b02d11395d&lt;/uuid&gt;
  &lt;forward mode=&apos;route&apos;/&gt;
  &lt;bridge name=&apos;virbr2&apos; stp=&apos;on&apos; delay=&apos;0&apos;/&gt;
  &lt;mac address=&apos;52:54:00:e3:bd:c4&apos;/&gt;
  &lt;ip address=&apos;192.168.2.1&apos; netmask=&apos;255.255.255.0&apos;&gt;
    &lt;dhcp&gt;
      &lt;range start=&apos;192.168.2.128&apos; end=&apos;192.168.2.254&apos;/&gt;
    &lt;/dhcp&gt;
  &lt;/ip&gt;
&lt;/network&gt;
</pre>

Restart the VN using the `virh net-start` command.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-start routed
Network routed started
</pre>

![](https://i.imgur.com/DuGMC7v.png)

The "c1" VM was restarted and verification of reachability was performed again to the target device (X1-Carbon-6th-Gen).

<pre>[sean@c1 ~]$ sudo ifdown eth0
Device &apos;eth0&apos; successfully disconnected.
</pre>

<pre>[sean@c1 ~]$ ip route
default via 192.168.2.1 dev eth1 proto dhcp metric 101 
192.168.2.0/24 dev eth1 proto kernel scope link src 192.168.2.133 metric 101 
</pre>

<pre>[sean@c1 ~]$ ping 192.168.188.223 -c 3
PING 192.168.188.223 (192.168.188.223) 56(84) bytes of data.
64 bytes from 192.168.188.223: icmp_seq=1 ttl=63 time=1.35 ms
64 bytes from 192.168.188.223: icmp_seq=2 ttl=63 time=1.62 ms
64 bytes from 192.168.188.223: icmp_seq=3 ttl=63 time=1.24 ms

--- 192.168.188.223 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 1.246/1.410/1.629/0.161 ms
</pre>

Likewise from the target device (X1-Carbon-6th-Gen), verify reachability to the "c1" VM at 192.168.2.133 (routed network) on the host server (Supermicro-813MFTQC) that is on the LAN at 192.168.188.4.

## Bridge with Shared Physical Interface

Create the `br0` bridge with the `eno2` interface and setup so the `br0` can be used while creating VM network interfaces. This way VMs will acquire a DHCP address from the LAN that `eno2` is connected to.

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ cat /etc/network/interfaces
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eno1
iface eno1 inet dhcp # IP address = 192.168.188.4

# Setup bridge interface on eno2
auto eno2
iface eno2 inet manual

auto br0
iface br0 inet dhcp
  hwaddress ether ac:1f:6b:61:5e:dd # IP address = 192.168.188.5
  bridge_ports eno2
  bridge_stp on
  bridge_fd 0</pre>

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ sudo systemctl restart networking
</pre>

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ brctl show
bridge name	bridge id		STP enabled	interfaces
br0		8000.ac1f6b615edd	yes		eno2
virbr0		8000.5254000c0d6f	yes		virbr0-nic
</pre>

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ ip -c address show dev br0
41: <font color="#06989A">br0</font>: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500 qdisc noqueue state <font color="#4E9A06">UP </font>group default qlen 1000
    link/ether <font color="#C4A000">ac:1f:6b:61:5e:dd</font> brd ff:ff:ff:ff:ff:ff
    inet <font color="#75507B">192.168.188.5</font>/24 brd 192.168.188.255 scope global br0
       valid_lft forever preferred_lft forever
    inet6 <font color="#3465A4">fe80::ae1f:6bff:fe61:5edd</font>/64 scope link 
       valid_lft forever preferred_lft forever
</pre>

Create an XML configuration file called “br0.xml”.

```xml=
<network>
  <name>br0</name>
  <forward mode="bridge"/>
  <bridge name="br0"/>
</network>
```

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-define br0.xml
Network br0 defined from br0.xml

<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-list --all
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 br0                  inactive   no            yes
 default              active     yes           yes

<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-start br0
Network br0 started

<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-autostart br0
Network br0 marked as autostarted

<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-list --all
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 br0                  active     yes           yes
 default              active     yes           yes

<font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ virsh net-dumpxml br0
&lt;network&gt;
  &lt;name&gt;br0&lt;/name&gt;
  &lt;uuid&gt;b074bb07-13a2-4847-93bd-690be8c27c7c&lt;/uuid&gt;
  &lt;forward mode=&apos;bridge&apos;/&gt;
  &lt;bridge name=&apos;br0&apos;/&gt;
&lt;/network&gt;
</pre>

![](https://i.imgur.com/x5r4cHx.png)

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>Supermicro-813MFTQC:</b></font><font color="#FCE94F"><b>~</b></font>$ brctl show 
bridge name	bridge id		STP enabled	interfaces
br0		8000.ac1f6b615edd	yes		eno2
							vnet0
virbr0		8000.5254000c0d6f	yes		virbr0-nic
</pre>

<pre>[sean@c1 ~]$ ip -c address show dev eth0
2: <font color="#06989A">eth0</font>: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500 qdisc pfifo_fast state <font color="#4E9A06">UP </font>group default qlen 1000
    link/ether <font color="#C4A000">52:54:00:8a:42:bb</font> brd <font color="#C4A000">ff:ff:ff:ff:ff:ff</font>
    inet <font color="#75507B">192.168.188.201</font>/24 brd <font color="#75507B">192.168.188.255 </font>scope global noprefixroute dynamic eth0
       valid_lft 42882sec preferred_lft 42882sec
    inet6 <font color="#3465A4">fe80::ccf3:ee70:4db8:17f7</font>/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
</pre>

<pre>[sean@c1 ~]$ ping google.com -c 3
PING google.com (74.125.138.139) 56(84) bytes of data.
64 bytes from yi-in-f139.1e100.net (74.125.138.139): icmp_seq=1 ttl=46 time=104 ms
64 bytes from yi-in-f139.1e100.net (74.125.138.139): icmp_seq=2 ttl=46 time=93.3 ms
64 bytes from yi-in-f139.1e100.net (74.125.138.139): icmp_seq=3 ttl=46 time=96.4 ms

--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 93.397/98.078/104.394/4.635 ms</pre>

## MacVTap

MacVTap is used when users in the LAN want to access the VM, but you don't want to create a normal Linux bridge. This connection is not used in production systems and is mostly used on a workstation-type station.

![](https://i.imgur.com/rFUwwYm.png)

<pre><font color="#34E2E2"><b>sean</b></font>@<font color="#8AE234"><b>X1-Carbon-6th-Gen:</b></font><font color="#FCE94F"><b>~</b></font>$ ping 192.168.188.201 -c 3
PING 192.168.188.201 (192.168.188.201) 56(84) bytes of data.
64 bytes from 192.168.188.201: icmp_seq=1 ttl=64 time=124 ms
64 bytes from 192.168.188.201: icmp_seq=2 ttl=64 time=7.63 ms
64 bytes from 192.168.188.201: icmp_seq=3 ttl=64 time=5.29 ms

--- 192.168.188.201 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 5.293/45.693/124.156/55.489 ms
</pre>

<pre>[sean@c1 ~]$ ip -c address show dev eth0
2: <font color="#06989A">eth0</font>: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500 qdisc pfifo_fast state <font color="#4E9A06">UP </font>group default qlen 1000
    link/ether <font color="#C4A000">52:54:00:8a:42:bb</font> brd <font color="#C4A000">ff:ff:ff:ff:ff:ff</font>
    inet <font color="#75507B">192.168.188.201</font>/24 brd <font color="#75507B">192.168.188.255 </font>scope global noprefixroute dynamic eth0
       valid_lft 42523sec preferred_lft 42523sec
    inet6 <font color="#3465A4">fe80::ccf3:ee70:4db8:17f7</font>/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
</pre>

