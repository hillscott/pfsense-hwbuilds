# pfSense - A Firewall For All The Things... on a budget
## Background
pfSense is maintained by the good people at [Netgate](https://www.netgate.com/). Many of the devices that they offer are worth purchasing. Their pfSense Gold membership also includes numerous extremely helpful videos to explain various network technologies to you. All of that being said, the builds that I will lay out in this repository are for the cases where budgets are incredibly tight, or access to timely shipping is... not feasible. I've lived in rural parts of countries such as Thailand and Albania, where getting a secure firewall is next to impossible. 

So don't "get" one... build it. That's what we'll do here. You can still buy support from Netgate, or get pfSense Gold, even using the hardware that I'll lay out here. You may be wondering what the purpose of this project is at this point. After all, I don't contribute code to pfSense, and anyone can build an x86_64 computer and just run pfSense right? 

_Yes... and no_

You can use any x86_64 computer to run pfSense, but that doesn't mean that you'll achieve the performance metrics that you need for your project, the hardware will even be compatible, or that it'll be efficient in power utilization, noise, space, and so forth.

If you don't need any more than ~75Mb/s NAT / basic firewall throughput and:
* You don't know (or care) about VPNs
* You don't know (or care) about IDS/IPS - Intrusion Detection / Intrusion Prevention

I have good news, you can simply buy this: [SG-1000](https://www.netgate.com/solutions/pfsense/sg-1000.html) and even if you can't get one of these where you are, really absolutely any PC can do the above. 

**If on the other hand...**
You **do** care about:
* VPN throughput (IPSec/OpenVPN/whatever)
* Intrusion Prevention (IDS/IPS)
* and are willing to deal with the extra steps to make things work... 

Read on

## Why pfSense?
Within this respository, I will provide as up-to-date of information as I can provide, on how to build network perimiter / firewalls that _work_. I've built and run these myself. I also don't ask you to trust me with... anything. Building your firewall on an open-source platform, from scratch, means that I (or anyone else) can't screw with you. Unless someone has managed to get some bad code past the many watchful eyes on the FreeBSD / pfSense projects... the vulnerabilities that you'll have to worry about are the normal ones that come up over time, and you'll get all the updates (for free - as in beer).

It should be noted that I work as an IT consultant, and I have worked with **many** firewall platforms: 
* Sonicwalls
* Watchguards
* Junipers
* Ciscos
* Palo Altos
* Fortinet
* Sophos
* ... Probably others that I have forgotten.

All of these have nice features / benefits, but none of them are both:
**Open-Source** and **Free**

It's also worth noting that pfSense can do _most_ of what all of the above can do. It may take 20 clicks with pfSense, instead of 3... and things such as that, but they get the job done. They are also an **AMAZING** learning tool. Running some of this stuff with a pay firewall, can run in the 10s of **THOUSANDS** of USD. 

All of that being said, I will continue to deploy many of the above products at client sites. I'm not going all pfSense, all the time. For one thing, the IPSec VPN stack wants to configure a separate P2 session for every subnet routed through a tunnel. Not a big deal, but annoying in some cases. pfSense is also not great at making Intrusion Prevention **EASY**. It will take much more work to get it doing some decent filtering. That's why I'm here though. We'll get through it. 

It's also worth noting that pfSense has no Intrusion Prevention built-in. It's done through either Snort or Suricata packages. They integrate nicely, but the distinction is important in terms of understanding your support options. For example... **I recently discovered that Suricata has a bug (at least in FreeBSD) in terms of it's ability to filter on an interface that is utilizing tagged vlans. If you enable Suricata on an interface with vlans, the parent interface can filter fine... but it'll prevent the tagged packets from making it to the child interfaces.** This may be a big problem for some deployments. I will report / dig into this bug as I have time, but for now, just bear it in mind. This is accurate as of Suricata version 4.0.3.

## Obligatory Shaking of the Cup
If you want to help me out, and are able - buy the things referenced here with the Amazon links to kick me some dough. Or help me on [Patreon](https://www.patreon.com/hillscott). Or don't, because you have no extra money, or live somewhere that Amazon doesn't deliver (effectively) to. 

Or... if you work in International Development, for sure don't give me money, just kick butt out there. Maybe drop me a line at **netsec4idev@protonmail.com** to tell me how you're using them. That's payment enough. Your conscience is clear my friend.

## Build Overview
Within this repository (and linked here), will be the various builds that I've done, with details on their performance. I will also walk-through the big features, that I view as important for a serious / enterprise network. I won't walk through _every_ feature. That's insane, RTFM. Without further adieu though...

### [Enterprise Build](./EnterpriseBuild.md)
If you need _over_ 125Mb/s Up/Down Throughput, with full Inline (as in block it before it gets through) Intrusion Prevention + OpenVPN... you need this build. 
_I say "over" as I haven't reached the limits of this build yet_. I need to take it to a GigE pipe to see how hard I can push it. This build also has 5x GigE Ethernet ports.
**Cost**: ~$560 + tax (and maybe shipping)
**Power Consumption**: 25-30watts (120v)
**Build Difficulty**: 
* _Physical_ - If you've ever built a PC, not hard. If you haven't, remain calm... you'll have to install the processor, but it's not that bad. 
* _Technical_ - Super easy, install and you're off to the races.

**Downsides**: It's the size of a media PC, and has a fan (that you probably won't hear)

### [Mid-Level SILENT Build](./MidLevelBuild.md)
If you need _under_ 70Mb/s Up/Down Throughout with OpenVPN, and want some Intrusion Prevention, even if it's with delayed blocking (instead of Inline), and you don't have $560 + tax, or just are scared by installing processors... or power usage, or fans / noise, this build may be for you. 
**Cost**: ~$275 + tax (and maybe shipping)
**Power Consumption**: 7-9watts (120v)
**Build Difficulty**:
* _Physical_ - Super easy, remove a few screws, drop in a stick of memory + a disk - and go. 
* _Technical_ - We have to build a kernel module. Or you have to trust mine. It's not hard if you take it step by step, but the words "kernel module" seem to scare some people. 

**Downsides**: It's not going to be good enough for a full 100Mb/s Up/Down pipe, unless you're willing to take the max speed hit. These words may make you laugh on your whopping 5Mbit/sec DSL line though - so this may be for you.