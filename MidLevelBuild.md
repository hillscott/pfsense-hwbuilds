# Mid-Level Build
If you need _under_ 70Mb/s Up/Down Throughout with OpenVPN, and want some Intrusion Prevention, even if it's with delayed blocking (instead of Inline), and you don't have $560 + tax, or just are scared by installing processors... or power usage, or fans / noise, this build may be for you. 

**Cost**: ~$275 + tax (and maybe shipping)

**Power Consumption**: 7-9watts (120v)

**Build Difficulty**:
* _Physical_ - Super easy, remove a few screws, drop in a stick of memory + a disk - and go. 
* _Technical_ - We have to build a kernel module. Or you have to trust mine. It's not hard if you take it step by step, but the words "kernel module" seem to scare some people. 

**Downsides**: It's not going to be good enough for a full 100Mb/s Up/Down pipe, unless you're willing to take the max speed hit, or not do Intrusion Prevention. These words may make you laugh on your whopping 5Mbit/sec DSL line though - so this may be for you.

## Hardware List
**_Prices are accurate as of the build in early 2018_**

| Price      | Description                                                   | Link                             |
| ---------- | ------------------------------------------------------------- | -------------------------------- |
| $169.99    | ZOTAC ZBOX-CI327NANO-U Intel Celron N3450 1.1Ghz DDR3L        | [Amazon](http://amzn.to/2ows1SS) |
| $54.62     | Crucial 4GB DDR3-1866 SODIMM 204-pin                          | [Amazon](http://amzn.to/2F1KcGk) |
| $37.99     | ADATA 64GB SATA III SSD                                       | [Amazon](http://amzn.to/2F2dXHf) |
| $9.99      | Optional: CableCreation USB 3.0 RJ45 Network Adapter GigE     | [Amazon](http://amzn.to/2EV66z7) |

**Total Price: $262.60-272.59**
**NOTES:**
* The ZOTAC supports M.2 Disks, but to install it, you have to void your warranty, so lets just not do that, and install a 2.5" SSD. Disk speed doesn't matter anyway for this. 
* Be careful to follow the directions on installing the thermal tape to the SSD and RAM. There are NO fans in this box, so failure to install the thermal tape may result in things melting.
* The Realtek drivers built into FreeBSD are... well terrible. So we will fix that below. You can eiter trust a pre-built driver (by me), or build it yourself with the directions that follow.

## Hardware Build Instructions
**Install the Hardware**
* The screws on this are a bit odd, you basically have to jam a screw driver into the squishy plastic screw tops to remove them.
* Install the Memory by sliding it in at an angle to the bottom slot, and pushing down until it clicks. We won't use the second memory slot (but feel free to if you want more memory).
* The SSD can be screwed into the bracket that is removable from the ZBOX, then you screw both of them in together.
* Close it up
* Plug up a monitor / keyboard (no mouse needed)
* Power it up - if all went well, you'll get a ZBOX Logo on the screen

**Update the BIOS**
* Download the BIOS here: [ZOTAC](https://www.zotac.com/download/mediadrivers/mb/bios/pb325CI327.zip)
* Extract it to a USB Key's (empty) root of the drive
* Bootup in UEFI mode (should be the default) and select the USB Key as the boot target (if necessary). 
* It should flash without intervention
* Open the BIOS
* In Boot, Change it to "UEFI - Compatible Mode" and set "Setup Prompt Timeout" to 3
* Turn off the hardware until you are ready to install

## Software Build Instructions
**Prepare the pfSense Installer + Install pfSense**
* Download the img from [pfsense](https://www.pfsense.org/download/)
* _NOTE_ You want the AMD64, USB Memory Stick Installer, VGA Console
* Follow the directions in the [pfsense docs](https://doc.pfsense.org/index.php/Writing_Disk_Images) to write it to your USB Flash Drive
* Once the flash drive is ready, insert the Flash Drive into the front of the ZBOX
* Turn on the ZBOX, it should auto-boot from your flash drive
* You will **NOT** be able to accept the default and have this install work, the moment that the pfSense logo appears, you are going to want to enter the "console" to "set hints"
* The hints that need to be set are:
```
set hint.sdhci_pci.0.disabled="1"
set hint.sdhci_pci.1.disabled="1"
set hint.hpet.0.clock="0"
```

* This will allow the install to go through without hanging for minutes on end
* After accepting the defaults (once it starts asking you to install), you'll be asked if you want to enter the console. Select **YES**
* Run the following commands:

```
echo 'legal.intel_ipw.license_ack="1"' >> /boot/loader.conf
echo 'legal.intel_iwi.license_ack="1"' >> /boot/loader.conf
echo 'hint.sdhci_pci.0.disabled="1"' >> /boot/loader.conf.local
echo 'hint.sdhci_pci.1.disabled="1"' >> /boot/loader.conf.local
echo 'hint.hpet.0.clock="0"' >> /boot/loader.conf.local
```

* Now type exit when the hardware reboots, remove the USB key. It should now boot pfsense.

**pfSense Configuration**
* Go through the setup wizard (name things as you like, select which interface is WAN / LAN etc)
* You may be asked to upgrade if there's a new version, go ahead and do that
* Now we need to address the Realtek driver being terrible... so once you're back into the web interface, enable SSH in System -> Advanced
* Now you have a choice:
* Use the Realtek driver included in this repository (easy) or build it yourself (harder, but more secure practice)
* If you _don't_ do this. You can still use the ZBOX with pfSense... but expect absolutely horrible performance. On the order of 70 Megabit/ps **with no Intrusion Prevention** also expect things to crash if you do try to use Intrusion Prevention without installing the module.

**Using my module...**
* If you are going to build it, skip to the next section.
* To use the module in this repository, you are simply going to want to copy it over and enable it with:

```
scp ZotacModule-FreeBSD11.1-rtlv194.01/if_re.ko admin@[pfsenseip]:/boot/kernel/
ssh admin@[pfsenseip]
cd /boot/kernel
chown 0555 if_re.ko
echo 'if_re_load="YES"' >>/boot/loader.conf.local
reboot
```

* Once the zbox comes back up, you can verify that you are using the new module by running:

```
kldstat
```

* You should see something like: _2    1 0xffffffff82e3f000 80900    if_re.ko_

**Building it yourself...**
* If you've installed the module already from this respository, skip this section.
* You're going to need a server / desktop / laptop that you can run some kind of Virtualization on. I use KVM on my linux box, but just about anything is supported. On the device that you'll be doing the virtualization with, download the FreeBSD image that matches your virtualization platform [here](https://download.freebsd.org/ftp/releases/VM-IMAGES/11.1-RELEASE/amd64/Latest/)
* Now Create a new FreeBSD machine in your platform, and select the downloaded image as the disk. This saves you the trouble of having to actually install FreeBSD, it'll be ready to go. 
* Now boot the VM up, and login to it with root, and no password
* Run the following commands (you may want to check RealTek's site for a newer driver):

```
fetch -o /tmp ftp://ftp.freebsd.org/pub/`uname -s`/releases/`uname -m`/`uname -r | cut -d'-' -f1,2`/src.txz
tar -C / -xvf /tmp/src.txz
pkg install curl
curl -o /tmp/rtlv194.tgz http://12244.wpc.azureedge.net/8012244/drivers/rtdrivers/cn/nic/0007-rtl_bsd_drv_v194.01.tgz
tar -xf /tmp/rtlv194.tgz -C /tmp/
cp /tmp/rtl_bsd_drv_v194.01/if_re* /usr/src/sys/dev/re/
cp /tmp/rtl_bsd_drv_v194.01/Makefile /usr/src/sys/dev/re/
cd /usr/src/sys/modules/re/
make
passwd
```

* The last command is to set a root password (pick something for the moment - you will only use it once)
* Now open /etc/ssh/sshd_config with either vim / nano / emacs... whatever
* Enable "PermitRootLogin"
* Save and Exit 

```
sh /etc/rd.d/sshd onestart
```

* Now you need to pull the compiled module out of the vm, locally, and transfer it up to the zbox.
* For example:

```
scp /usr/src/sys/modules/re/if_re.ko root@pfsense:/boot/kernel/
```

* Once it's on the ZBOX, ssh into the ZBOX and run the following:

```
ssh admin@[pfsenseip]
cd /boot/kernel
chown 0555 if_re.ko
echo 'if_re_load="YES"' >>/boot/loader.conf.local
reboot
```

* Once the zbox comes back up, you can verify that you are using the new module by running:

```
kldstat
```

* You should see something like: _2    1 0xffffffff82e3f000 80900    if_re.ko_

**Extra Steps for USB3 Ethernet Adapter**
* If you've purchased the Ethernet Adapter, it should show up on it's own, you'll just need to set it up in "Interface -> Assignments"
* Be sure that you "Enable" the interface, and you can call it WAN2, GUEST, whatever fits

**Admin Access**
* Along the top menu bar: System -> Advanced -> Admin Access
* Select HTTPS
* Select Enable Secure Shell (if you haven't already)
* Primary Console should say VGA Console

**Advanced Networking**
* System -> Advanced -> Networking
* Consider unchecking "Allow IPv6" (if you don't use it). IPv6 is great, once you know how to use it. It provides avenues for VPNs / Tunneling to be bypassed if you don't...
* CHECK "Disable" Hardware Checksum offload - Inline IPS filtering doesn't work reliably unless this is done

**Miscellaneous**
* System -> Advanced -> Miscellaneous
* CHECK "Enable default gateway switching" (you can't do multi-wan without this)
* Cryptographic Hardware -> Set to AES-NI and BSD Crypto Device (allows for better OVPN throughput)

**OpenVPN**
* VPN -> OpenVPN -> Wizards
* Local User access
* Descriptive Name: [External DNS/DDNS Name you will use]
* Key length: 4096
* Fill in the rest as appropriate for where you are
* Interface: WAN
* Protocol: UDP
* Local Port: Something larger than 1194...just to mix it up
* Change DH Param Length to 4096
* Change Auth Digest to SHA512
* Hardware Crypto: BSD cryptodev engine
* Tunnel Network: 10.something.something.0/24
* Redirect Gateway: Check IF YOU WANT Tunnel All
* Leave the lest alone, Next
* Check the firewall and OpenVPN rule add options, Next

**Cert Manager**
* System -> Cert Manager -> CAs
* There should be a cert here... after you go through the OpenVPN wizard
* System -> Cert Manager -> Certificates
* There should be 2 Certificates here now (one for OpenVPN, one for the Configuration interface)

**General Setup**
* System -> General Setup
* Consider if you want to check "Allow DNS server list to be overridden by DHCP on WAN"
* I also prefer the "pfsense-dark" theme... that's purely preference
* Most other things should be fine from the setup wizard

**DNS Resolver**
* Services -> DNS Resolver
* You may (or may not want to turn this off) Some ISPs don't like it when you don't use their DNS servers. It's more secure to use your own though.

**Package Manager**
* System -> Package Manager
* **Recommend installs:** openvpn-client-export and suricata (if you want Intrusion Prevention)

**Multi-WAN / Fail-Over**
* System -> Routing -> Gateways
* If you have a second ISP and want to use it as a fail-over / MultiWAN, you'll need to add the interface as a potential Gateway with "Add"
* If you are doing Multi-WAN, Once you've added the Gateway, setup a Gateway Group for fail-over WAN

**User Manager**
* System -> User Manager
* Create a separate user for VPN purposes here
* Add
* Fill in Username / Password / Full Name
* CHECK create a user certificate
* Select 4096 bits
* SAVE

**NAT**
* Firewall -> NAT
* THIS is where you should start if you are poking a hole for a serer that you run locally, you'll have the option to make the firewall part of it at the same time. 

**Trafic Shaping**
* Firewall -> Traffic Shaper
* This does what it sounds like, you can use the Wizard to give VoIP higher priority etc

**DHCP Server**
* Services -> DHCP Server
* You can setup Reservations here, at the bottom of the page

**Dynamic DNS**
* Services -> Dynamic DNS 
* Tons of services supported, and you can use this to have a DNS name to get into OpenVPN with (or something else)

**Suricata / Intrusion Prevention**
* Services -> Suricata
 * Lots of configuration here, and it's the entire reason for the high-powered hardware
* Services -> Suricata -> Global Settings
 * I recommend ETOpen and Snort VRT. I pay the $30/year for Snort rules, the "registered" set is the same, just always 30-days behind
 * Get your "Oinkmaster" code from the snort webpage, and enter the snapshot name here (version 2990) at time of writing
 * Update Interval -> I do 12 hours, but then I have the pay rules, up to you
 * Update Start time -> Pick something random to reduce load on their servers, like 00:32
 * Remove Blocked Hosts Interval -> I do 12 hours, use good judgement at first. 
 * Make sure "Settings will not be removed during package deinstallation" is checked
* Services -> Suricata -> Updates
 * Once you've configured the Oinkmaster code + checked ETOpen, click "Update" 
* Services -> Suricata -> Interfaces
 * Add one, I do WAN filtering, you can also filter LAN for more security (every interface that you filter requires more memory and processing power). This build can certainly support WAN/LAN filtering, and even some other interfaces.
**WAN Interface Settings**
 * Enable DNS Log (if you like)
 * Enable Stats Log (useful)
 * Block Offenders - You want this on. I recommend IPS Mode: "Legacy" on this hardware. I had random reboots with Inline on the RealTek hardware
 * Max Pending Packets: 2048 (the default is too low)

**WAN Categories**
 * CHECK Auto-enable rules required for checked flowbits
 * CHECK Use rules from one of three pre-defined Snort IPS policies
 * IPS Policy Selection: Up to you, but I have tested / am happy with "Security"
 * IPS Policy Mode: "Policy" 
 * Now you'll need to check the "emerging / ETOpen" rule categories that you want on top of Snort rules. 
 * I have included a file suricata-enable.conf which is a listing of my recommendations
 * Be sure to click SAVE when done

**SID Management**
* Services -> Suricata -> SID Mgmt
* OPTIONAL: I have provided "suricata-dropsid.conf" which allows you to force all auto-enabled rules to have a DROP action attached to them.
 * If using this feature... check the "Enable automatic management of rule state and content using configuration files"
 * Select it in the drop down for "Drop SID File", check REBUILD and click SAVE

**Filter START**
* Services -> Suricata -> Interfaces
* Click the start icon on WAN
* Now open the "Alerts" section in Suricata, and watch as things go nuts. If inline blocking is working, they should show up as highlighted. Have fun.

**VLANs**
* Interfaces -> VLANs (if you want a Guest Wifi / some other sub-interface)
* Add
 * Parent: em1 (LAN)
 * VLAN Tag: [Pick some small #]
 * Description: Guest
WARNING!! WARNING!! DANGER WILL ROBINSON!! 
--------> Suricata will prevent VLAN tagged interfaces from functioning, if it is scanning an interface with tags enabled. The parent interface will work fine, but the tagged interface will NOT pass traffic. Stop Suricata on the parent interface... and the tagged interface will immediately start working again. I'm still researching this issue. Some discussion on an outdated version here: [security-onion](https://github.com/Security-Onion-Solutions/security-onion/wiki/VLAN-Traffic) For now, you may need to have your switch send untagged VLANs to a separate physical interface to work-around the issue (if you even need a network outside of WAN/LAN).

**Interface Setup (for a Guest Wifi network as an example)**
* Interfaces -> Interface Assignments
 * Available network ports: [Select Guest], Add
 * It will now show up as OPT1, click on OPT1
 * Rename it GUEST
 * Check ENABLE Interface
 * IPv4 Configuration Type -> Static IPv4
 * Assign an IP on a different subnet from your LAN (with appropriate netmask)
 * Apply

**DHCP Server setup for the New Interface**
* Services -> DHCP Server -> Click on GUEST
 * CHECK Enable DHCP server on GUEST interface
 * Define a range of IPs within your GUEST subnet

**Firewall setup for the New Interface**
* Firewall Rules -> GUEST
 * Add (at bottom)
 * Pass
 * Destination: WAN net
 * Protocol: Any
 * SAVE, Apply

_Now setup the new SSID / VLAN in your AP interface_

**More fun things to look at**
* Status -> Traffic Graph (fun to watch)
* Status -> OpenVPN (useful)
* Status -> Interfaces (easy way to release / renew)
* Diagnostics -> Backup & Restore (use this + the encryption). If you buy pfSense Gold, they will let you auto-backup to their cloud as well
* Diagnostics -> Edit File (easy way to mess with /boot/loader.conf.local for example
* Diagnostics -> Packet Capture (useful for debugging)
* Diagnostics -> Reboot (obvious)
* Diagnostics -> System Activity (top basically - useful for Suricata performance monitoring somewhat)
