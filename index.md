# WoST Zone - Web Of Secured Things - DRAFT

WoST is a language agnostic implementation of Web Of Things (WOT) with a focus on security, based on the 'Things Are NOT Servers' paradigm.

It aims to be WoT compliant while providing restrictions to establish a secured implementation of the WoT architecture.

## Status & Roadmap

This project is currently in the definition phase. 

Planned project deliverables, subject to change no doubt:
1. Conceptual design [in progress]
2. Secured Thing Gateway pipeline core
3. Core pipeline plugins


## Objectives

The primary objective of this project is the creation of a secure approach to the Web of Things (WoT) while remaining compatible with the WoT architecture specifications.

## Overview

With billions of IoT devices being used, and this number growing fast, the [WoT](https://www.w3.org/TR/?title=web%20of%20things) provides standards to support interoperability between IoT devices and its consumers. It addresses one of the biggest problems in IoT today.

However, without sufficiently high guarantee of security, the work to ensure interoperability only address a part of the problem faced in building IoT applications.
The concern is that the current WoT architecture allows for Things to act as a server and the given examples seem to be biased towards it. Implementing a secure server is difficult and requires constant vigilance and software updates to keep it secure. Having billions of servers potentially exposed to the internet is a security nightmare waiting to happen. Especially considering that most will never see an update after they are shipped. WoST aims to prevent this from becoming a problem. 

The primary objective of WoST is to improve security in WoT simply by not allowing Things to expose a server, except when its purpose is to be a server. It aims to be WoT compliant except where this objective would be compromised.

It is the hope that 'WoST' compliancy will become a must have for the security minded consumer when purchasing IoT devices.

This approach requires the use of a gateway to act on behalf of connected Things. Rather than configuring each Thing, a centralized configuration facility on a gateway will let users manage the configuration. The gateway approach is already quite common for existing IoT devices such as ZWave and Zigbee and does not seem to be a hindrance to adoption.

## The Problem With WoT's Reference Implementation

WoT's [reference implementation](https://github.com/WebThingsIO) implements a 'Thing' as a client and as a server. The problem is with running a server on every 'Thing'.

A web server is a security risk, no matter how well written. If all 'WoT Things' implement a web server, the number of web servers would run in the many billions. Each server can potentially be hacked or used in denial of service attacks. Hardening a server for exposure to the Internet is difficult and requires constant vigilance and regular security patches. It is irresponsible to expose a server to the Internet without it.

What are the chances of IoT device manufacturers to provide this level of security and support? If the past is any indication then the chances are nil [1]. A good example that even the best of intentions is not enough is that of CoAP. CoAP is lauded as one of the top IoT communication protocols of the future [2]. Unfortunately it is also vulnerable to so called DDOS reflection attacks [3]. A simple malformed message to a CoAP device results in a much larger message to a target address which is what DDOS attackers use to overload a web site. There is no fix (Dec 2020) and even if there was, who is going to update all these devices? ... The reality is that this won't happen.

Okay you might say, but what about putting all these devices behind a firewall and use a gateway or 'intermediary' to connect to the Internet? This is certainly possible but no-where in the spec it says that 'Things' should not be connected to the Internet. In fact, the architecture shows several use-cases [4] where a remote controller connects to a 'Thing' via the Internet. While the WoT architecture specification is agnostic of how this connection takes place, the reference implementation interprets this as opening a listening port on a Thing.

A related issue is that the LAN is not a safe place. A cross scripting attack can let a web site run a script on your browser that connects to your Thing unless the 'thing' is well protected against such attacks. Javascript running in your browser is already on the LAN and has already bypassed the firewall. The script can scan for devices on your LAN and potentially connect to them[5]. Imagine visiting a web site and suddenly your 'smart door lock' thing is controlled by someone else... In addition, viruses are well known for compromising PC's and can perform attacks on LAN based Things without browser restrictions.

As if these aren't enough risks, a lot of so-called smart devices today use UPnP or some tunneling solution to tell their router to expose them to the Internet[6] so it seems common practice to do so. Various manuals explain how to turn on UPnP to control devices remotely [7]. Oh the irony.

The fact is that the moment you allow Internet access to a 'Thing', the convenience factor wins and you have lost control of the situation. Warnings that the situation is dire make the news regularly but there is little heed paid to these warnings. Many manufacturers, including the WoT working group still see nothing wrong with running servers on IoT devices.

WoT security goal is that quote: ["devices should not be used in any form of attack"](https://www.w3.org/TR/wot-security/#wot-threat-model). However the architecture itself stays silent on how to achieve this class of security risks. One can argue that instead it should provide guidance and bear responsibility to implementations of said architecture.

## The Security First Approach In WOST

WoST's paradigm is: 'Things Are Not Servers'. WoST requires that after provisioning a Thing MUST NOT act as a server, unless its purpose is to be a server. 

WoST compliant 'Things' MUST adhere to simple but strict rules:
<code>
1. Secured Things MUST connect to their provisioned Secured Thing Gateway. The STG will not connect to Things.

2. Secured Things MUST NOT operate a server for the purpose of WoT after they are provisioned. It MUST NOT listen on any ports until it is unprovisioned by a factory reset.
   
   If the main purpose of the Thing is to be a server this functionality remains separate from the Thing functionality. Eg, a server is allowed for other purposes. For example, a web server can be monitored as a Secured Thing.

3. Secured Things MUST NOT request firewall ports to be opened using UPnP or any other means, including written instructions.

4. Secured Things are only allowed to connect to the Internet to accomplish their primary purpose unless explicitly documented and approved by the user. This is more of a privacy issue but important nontheless.

   Secured Thing providers can provide an Internet service that a Thing connects to after the user accepts the terms and MUST clearly identify what data is send to the service and how often.

   For example, sending usage data from a Thing to an Internet provider is not allowed until the administrator is informed about what data is sent, how often, and gives its consent.
</code>

The above set of rules reduces the number of servers greatly as only Gateways run a server. At the same time it improves privacy. Thing manufacturers don't have to be as stringent about security, and less memory and CPU are needed on Things as no server is needed. Simpler and cheaper, what is not to like.

This raises an obvious question, how to connect to a Thing? The answer is that a consumer does not connect to a Thing. Instead, a consumer connects to the gateway that a Thing is provisioned (paired) with. A Thing connects to its provisioned gateway to send and receive messages while the connection is in place.

A Gateway supports HTTP and sub protocols like WebSockets as defined in their protocol binding (see WoT on profiles and protocol bindings). A Thing that implements one of these protocols will be able to connect to a gateway which serves the Thing Description and relays actions and events.


## Secured Thing Gateway (STG) 

The primary purpose of a 'Secured Thing Gateway' (STG) is to act as an intermediary for Secured Things. The STG supports provisioning of Things, serves Thing TD's and relays actions and events that are defined in the TD. In addition a gateway can relay messages to cloud based STG intermediaries so that no direct LAN access is needed to view and control Secured Things from the Internet.

STG's can be extended with plugins to also act as a gateway for 3rd party IoT devices such as ZWave and one-wire, and for services such as a mDNS discovery, directory service and web server for user interaction. 

The burden of proper security lies therefore with the STG. Gateway providers must be committed to ensure their gateways are up to date with security patches, similar to Windows, Android devices or iPhones. While the risk doesn't disappear it is much more managable than requiring this for every single Thing itself.

## WoST Compliance

In addition to being WoT compliant, WoST compliant Secured Thing Gateways MUST adhere to the following rules:
<code>
1. STG's MUST support provisioning of Secured Things.

2. STG's MUST implement a [directory service](https://www.w3.org/TR/2020/WD-wot-discovery-20201124/#exploration-directory) that serves discovery of Secured Thing TDs. The STG MUST update the Thing address to itself as it is responsible for routing messages to and from the Thing.

3. STG's MUST relay actions and events between Things and their consumers as described in their TD.

4. STG's CAN be configured to act as an intermediary and push selected Exposed Things to another STG or intermediary. 

5. STG's MUST have the ability for [post-manufacturing updates of itself, its scripts and its plugins](https://www.w3.org/TR/wot-architecture/#sec-security-consideration-update-provisioning). The authenticity of security updates MUST be verified before they are applied.

6. Commercial STG Manufacturers MUST make security patches available for the duration of the support period of the gateway. The security update interval for minor to intermediate vulnerabilities MUST be 6 months or less. After being notified of a severe vulnerability, a security patch MUST be made available within one month of notification. (TODO, adhere to common definitions of minor, intermediate and severe vulnerabilities)

   Support for automatic updates of the firmware with security patches from a trusted source is STRONGLY recommended where possible. It MUST have the ability to disable automatic updates and use manual updates. 
</code>

## Secured Thing Gateway Discovery

### mDNS 
A Secured Thing Gateway MAY implement a [DNS-Based Service Discovery](https://www.w3.org/TR/2020/WD-wot-discovery-20201124/#introduction-dns-sd). Things and Thing consumers can use this to discover the gateway on a local network. 

The service name of the STG follows the WoT Service Discovery naming. Tenatively "_directory._sub._wot". The type in the TXT record of the service instance is "Directory".

Secured Thing discovery works via the Secured Gateway that provides the WoT Directory Service.

### Manual Discovery

Secured Things that do not support mDNS but do have a configuration file or utility, can be linked to the gateway using the Secured Gateway hostname or IP address.

### Unsecured Things

Things that expect the gateway to discover and connect to it can be supported through a plugin for legacy devices, similar to 3rd party protocols such as zwave and onewire controllers. 


## Thing Provisioning With Secured Thing Gateways

Provisioning is the act of setting up a trusted relationship between Secured Thing and Secured Thing Gateway. The process is initiated by a Secured Thing when it is unprovisioned and a gateway is discovered. One of the methods described in the [OCF Security Specification](https://www.w3.org/TR/wot-security/#bib-ocf17) is used. Things with the ability to present a number can use the random pin method to prevent a man in the middle attack during the provisioning process.

Each Secured Thing MUST have a private and public key for signing messages. A new key set is best generated during the provisioning process. During the provisioning process, the  public keys are exchanged over an encrypted channel using JWE. After provisioning all messages are signed using JWS. The message content can be encrypted using JWE. The preferred encryption method for keys and session is elliptic curve cryptography. 

The STG keeps a list of provisioned Secured Things and their public key. Messages from the Thing are only accepted when signed with JWS (unless they are provisioned in test mode).

### TD Registration With Secured Thing Gateway

After provisioning, Secured Things MUST [register](https://www.w3.org/TR/2020/WD-wot-discovery-20201124/#exploration-directory-api-registration) their TD with a Secured Thing Gateway. The gateway MUST make this TD available through the associated directory service using the directory service API.


# References
[1] [Top 10 IoT security threats and vulnerabilities](https://blog.particle.io/the-top-10-iot-security-threats/)

[2] [CoAP](https://www.cse.wustl.edu/~jain/cse574-14/ftp/coap/)

[3] [FBI warns of new DDoS attack vectors: CoAP, WS-DD,...](https://www.zdnet.com/article/fbi-warns-of-new-ddos-attack-vectors-coap-ws-dd-arms-and-jenkins/)

[4] [WOT Architecture use-cases](https://www.w3.org/TR/wot-architecture/#remote-access)

[5] [Cross site scripting attacks](https://owasp.org/www-community/attacks/xss/)

[6] [UPnP a key threat to home IoT networks](https://securityledger.com/2019/03/devices-upnp-service-emerges-as-key-threat-to-home-iot-networks/) and [UPnP-enabled home devices](https://www.trendmicro.com/en_ca/research/19/c/upnp-enabled-connected-devices-in-home-unpatched-known-vulnerabilities.html)

[7] [Enabling UPnP for remote access](https://ltsecurityinc.zendesk.com/hc/en-us/articles/360008649453-How-to-enable-UPNP-to-perform-the-Port-Forwarding-without-accessing-the-router-)

[8] [OCF Security Specifications](https://www.w3.org/TR/wot-security/#bib-ocf17)
