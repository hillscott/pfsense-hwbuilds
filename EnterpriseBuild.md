# Enterprise Build
If you need _over_ 125Mb/s Up/Down Throughput, with full Inline (as in block it before it gets through) Intrusion Prevention + OpenVPN... you need this build. 

_I say "over" as I haven't reached the limits of this build yet_. I need to take it to a GigE pipe to see how hard I can push it. This build also has 5x GigE Ethernet ports.

**Cost**: ~$560 + tax (and maybe shipping)

**Power Consumption**: 25-30watts (120v)

**Build Difficulty**: 
* _Physical_ - If you've ever built a PC, not hard. If you haven't, remain calm... you'll have to install the processor, but it's not that bad. 
* _Technical_ - Super easy, install and you're off to the races.

**Downsides**: It's the size of a media PC, and has a fan (that you probably won't hear)

## Hardware List
**_Prices are accurate as of the build in early 2018_**

| Price      | Description                                                   | Link                             |
| ---------- | ------------------------------------------------------------- | -------------------------------- |
| $238.84    | Shuttle XPC Cube SH110R4 Socket LGA 1151 (case + motherboard) | [Amazon](http://amzn.to/2HQzhRR) |
| $179.96    | Intel Core i5 7400 (Kaby Lake) 4C 3-3.5Ghz w/ 6MB L3 Cache    | [Amazon](http://amzn.to/2oBkAJt) |
| $46.00     | Intel XPI9404PTL PRO/1000 PT Quad Port Server Adapter PCIe x4 | [Amazon](http://amzn.to/2FyYcbz) |
| $44.99     | Patriot Signature 8GB (2x 4GB) 288-pin DDR4 2133Mhz RAM       | [Amazon](http://amzn.to/2ov2LfP) |
| $44.99     | Silicon Power 120GB M.2 2280 SSD                              | [Amazon](http://amzn.to/2BUAgj5) |

**Total Price: $554.78**

**NOTES:**
* The Motherboard included in the XPC Cube SH110R4 has no PCIex4 slot. It does have on-board graphics and an x16 slot though, so I used the x16 slot for the x4 NIC, and that works just fine. All PCIe slots can be used by a smaller card with no performance / safety issues.
* You will likely have to flash the firmware on the motherboard to get "full" Kaby Lake support. This is fairly easy though. 
* The Memory / Disk brands were just the cheapest at the time. Don't feel stuck on those, do mind the speeds, pin counts, and M.2 length specifications though. 

## Hardware Build Instructions
**Install the Hardware**
* Remove the thumb screws and take a peek
* There are some decent instructions included with the Shuttle XPC. One note, this type of socket / CPU needs to line up where the little arrow is pointing, it won't "slip" in though, it just sits on top.
* Take care to install the thermal paste in a thin layer over the CPU (between it and the thermal heatsink). I use an old credit card or postcard to apply the layer of paste over the entire processor.
* Install the Memory / M.2 SSD + screw it in
* Use a screw-driver to _forcibly_ push out the PCIe slot protector that is closer to the CPU (higher up) - for the PCIe x16 slot. 
* Install the 4x Gbe NIC
* Close it up
* Plug up a monitor / keyboard (no mouse needed)
* Power it up - if all went well, you'll get a Shuttle Logo on the screen

**Update the BIOS**
* Download the BIOS here: [Shuttle](http://global.shuttle.com/products/productsDownload?productId=1990&osId=13)
* Extract it to a USB Key's (empty) root of the drive
* Bootup in UEFI mode (should be the default) and select the USB Key as the boot target (if necessary). 
*  It should flash without intervention
* Once rebooted, you'll likely get Checksum CMOS errors - that's fine, let it reboot a couple of times, and open the BIOS with DEL
* In Boot, Change it from UEFI -> Legacy
* Turn off the hardware until you are ready to install

## Software Build Instructions
**Prepare the pfSense Installer + Install pfSense**
* Download the img from [pfsense](https://www.pfsense.org/download/)
* _NOTE_ You want the AMD64, USB Memory Stick Installer, VGA Console
* Follow the directions in the [pfsense docs](https://doc.pfsense.org/index.php/Writing_Disk_Images) to write it to your USB Flash Drive
* Once the flash drive is ready, insert the Flash Drive into the front of the Shuttle XPC Cube
* Turn on the XPC Cube, it should auto-boot from your flash drive
* Go ahead and accept defaults the whole way through, enter, enter, enter, etc. 
* When the install is complete, just type exit, and when the hardware reboots, remove the USB key. It should now boot pfsense. 

**pfSense Configuration**
* Go through the setup wizard (name things as you like, select which interface is WAN / LAN etc)
* _NOTE_ em1 is the TOP interface on the 4x port Intel nic, em5 is the on-board
* You may be asked to upgrade if there's a new version, go ahead and do that

**Admin Access**
* Along the top menu bar: System -> Advanced -> Admin Access
* Select HTTPS
* Select Enable Secure Shell
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
* I recommend that you UNCHECK "Allow DNS server list to be overridden by DHCP on WAN"
* Under DNS Servers, I'd actually invite you to consider switching to [Quad9](https://www.quad9.net/about/) for your DNS services. They automatically block queries for known bad hosts. If you want to use their services, you would just set the DNS Servers as follows:

| IP              | Hostname       | Gateway     |
| --------------- | -------------- | ----------- |
| 9.9.9.9         | dns.quad9.net. | WAN_DHCP... |
| 149.112.112.112 | dns.quad9.net. | WAN_DHCP... |

* I also prefer the "pfsense-dark" theme... that's purely preference
* Most other things should be fine from the setup wizard

**DNS Resolver**
* Services -> DNS Resolver
* You may (or may not want to turn this off) Some ISPs don't like it when you don't use their DNS servers. It's more secure to use your own though.
* NOTE: I've been trying out DNS over TLS, since Quad9 now supports this... thus far I haven't had great results though, so I'll update here when I have something better to report.

**Package Manager**
* System -> Package Manager
* **Recommend installs:** openvpn-client-export, pfBlockerNG, and suricata (if you want Intrusion Prevention)

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

**pfBlockerNG IP / DNS Blocking**
* While using Quad9 DNS will help you immensely, it may also make sense to handle some blacklisting of domains or IPs yourself. By doing so, you'll also be able to block most Ads at the NETWORK level. So even your phone / your fire stick, your whatever can't connect to doubleclick / other trackers. It also helps cut down on some of the noise, by blocking known bad actors outright at the IP level. 
* The sections to look at to configure this are in: 
Firewall -> pfBlockerNG -> IPv4 blocks
Firewall -> pfBlockerNG -> DNSBL
Firewall -> pfBlockerNG -> DNSBL Feeds
Firewall -> pfBlockerNG -> DNSBL EasyList
* There is a good example set of feeds [here](https://supratim-sanyal.blogspot.com/2017/04/pfsense-pfblockerng-ultimate-list-of-ip.html)

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
* Block Offenders - You want this on. InLine should work on Intel hardware
* Max Pending Packets: 2048 (the default is too low)
* Detect Engine Profile: High (we have lots of memory)

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
