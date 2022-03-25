---
layout: page
title: "DNS Resolvers"
description: "The [Domain Name System (DNS)](https://en.wikipedia.org/wiki/Domain_Name_System) is the phonebook of the Internet. DNS translates domain names to [IP](https://en.wikipedia.org/wiki/Internet_Protocol) addresses so browsers and other services can load Internet resources, through a decentralized network of servers."
---

## What is DNS?
Simply put, when you visit a website `privacyguides.org`, an address is returned in this case `198.98.54.105`.

DNS has existed since the [early days](https://en.wikipedia.org/wiki/Domain_Name_System#History) of the Internet. DNS requests to and from DNS servers are **not** generally encrypted. In a residential setting a customer is given servers by the [ISP](https://en.wikipedia.org/wiki/Internet_service_provider) via [Dynamic Host Configuration Protocol (DHCP)](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol).

Unencrypted DNS is able to be easily **surveilled** and **modified** in transit. In some parts of the world ISPs are ordered to do primitive [DNS filtering](https://en.wikipedia.org/wiki/DNS_blocking). When a customer requests the IP of a domain that is blocked the server may not respond, or may respond with a different IP address. As the DNS protocol is not encrypted the ISP (or any network operator) employ [deep packet inspection (DPI)](https://en.wikipedia.org/wiki/Deep_packet_inspection) to monitor requests or further block them based on common characteristics, regardless of which DNS server is used.

1. Using [tcpdump](https://en.wikipedia.org/wiki/Tcpdump) to monitor and record internet packet flow we start recording with some filters (Cloudflare and Google). Unencrypted DNS always uses [port](https://en.wikipedia.org/wiki/Port_(computer_networking)) 53 and is always on the [User Datagram Protocol (UDP)](https://en.wikipedia.org/wiki/User_Datagram_Protocol):
   ```
   sudo tcpdump udp and host 1.1.1.1 or host 8.8.8.8 and port 53 -w dns.pcap
   ```

2. We can then use [dig](https://en.wikipedia.org/wiki/Dig_(command)) to send the DNS lookup to both servers. Software such as web browsers do these lookups  automatically unless they are configured to use [encrypted dns](#what-is-encrypted-dns).
   ```
   dig +noall +answer privacyguides.org @1.1.1.1
   dig +noall +answer privacyguides.org @8.8.8.8
   ```

3. Using [Wireshark](https://en.wikipedia.org/wiki/Wireshark) we can analyse the results. The top portion of the window contains the "[frames](https://en.wikipedia.org/wiki/Ethernet_frame)", while the bottom portion contains all of the data about the selected frame. Enterprise filtering and monitoring solutions (such as those purchased by governments and enterprise) can do the process automatically and without human interaction and can aggregate those frames to produce statistical data useful to the network observer.

{% include table-unencrypted-dns.html %}

An observer could modify any of these packets.

## What is "encrypted DNS"?
Encrypted DNS can refer to one of a many different protocols, the ones below likely to be the ones you may encounter.

### DNSCrypt
[**DNSCrypt**](https://en.wikipedia.org/wiki/DNSCrypt) is one of the first methods of encrypting DNS queries. The [protocol](https://en.wikipedia.org/wiki/DNSCrypt#Protocol) operated on [port 443](https://en.wikipedia.org/wiki/Well-known_ports) and worked in both [TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol) and [UDP](https://en.wikipedia.org/wiki/User_Datagram_Protocol) modes. DNSCrypt was never submitted to the [Internet Engineering Task Force](https://en.wikipedia.org/wiki/Internet_Engineering_Task_Force) nor did it go through the [Request for Comments](https://en.wikipedia.org/wiki/Request_for_Comments) process, so it was never widely used outside of a few [implementations](https://dnscrypt.info/implementations), and as a result it has been largely replaced by the more popular [DNS over HTTPS](/dns/#dns-over-https-doh).

### DNS over TLS (DoT)
[**DNS over TLS (DoT)**](https://en.wikipedia.org/wiki/DNS_over_TLS) is another method for encrypting DNS communication that was defined in [RFC 7868](https://datatracker.ietf.org/doc/html/rfc7858). Support was first implemented in [Android 9](https://en.wikipedia.org/wiki/Android_Pie), [iOS 14](https://en.wikipedia.org/wiki/IOS_14) and on Linux in [systemd-resolved](https://www.freedesktop.org/software/systemd/man/resolved.conf.html#DNSOverTLS=) in version 237. Preference in the industry has been moving away from DoT to [DNS over HTTPS](/dns/#dns-over-https-doh) in recent years as DoT is a [complex protocol](https://dnscrypt.info/faq/) and has varying degrees of compliance to the RFC across the implementations that exist. DoT also operates on a dedicated port 853 and that can be blocked easily by restrictive firewalls.

### DNS over HTTPS (DoH)
[**DNS over HTTPS**](https://en.wikipedia.org/wiki/DNS_over_HTTPS) as defined in [RFC 8484](https://datatracker.ietf.org/doc/html/rfc8484) packages queries in the [HTTP/2](https://en.wikipedia.org/wiki/HTTP/2) protocol and provides security with [HTTPS](https://en.wikipedia.org/wiki/HTTPS). Support was first added in web browsers such as [Firefox](https://support.mozilla.org/en-US/kb/firefox-dns-over-https) and [Chrome 83](https://blog.chromium.org/2020/05/a-safer-and-more-private-browsing-DoH.html).

Native implementations showed up in [iOS 14](https://en.wikipedia.org/wiki/IOS_14), [macOS 11](https://en.wikipedia.org/wiki/MacOS_11), [Microsoft Windows](https://docs.microsoft.com/en-us/windows-server/networking/dns/doh-client-support), and Android 13 (however it won't be enabled [by default](https://android-review.googlesource.com/c/platform/packages/modules/DnsResolver/+/1833144)). General linux desktop support is waiting on the systemd [implementation](https://github.com/systemd/systemd/issues/8639) so installing third party software is still required as described [below](/dns/#linux).

## What can an outside party see?
If we run the modify the above tests to work with a DoH request:

1. Firstly start `tcpdump`:
   ```
   sudo tcpdump host 1.1.1.1 and port 443 -w dns_doh.pcap
   ```

2. Secondly make a request with `curl`:
   ```
   curl -vI --doh-url https://1.1.1.1/dns-query https://privacyguides.org
   ```

If we then look at the output of the capture in Wireshark, we will see the [connection establishment](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#Connection_establishment) and [TLS handshake](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/) that occurs with any encrypted connection. When looking at the application data packets that follow, none of them contain the domain we requested or the IP address returned.

## Why **shouldn't** I use encrypted DNS?
At this point you'd probably be thinking that there is "increased privacy" however, that isn't necessarily the case. We don't make DNS lookups and then do nothing with the returned result. After we know what the privacyguides.org IP address is, we want to see the page that we're looking at.

1. So let's start capturing again with `tcpdump`:
   ```
   sudo tcpdump host 198.98.54.105 and port 443 -w pg.pcap
   ```

2. Then we visit https://privacyguides.org

3. Then we analyze pg.pcap in Wireshark.

Again you'll see the TCP connection establishment, followed by the TLS handshake for the Privacy Guides website. Around frame 5. you'll see a "Client Hello".



To understand more graphically we produced this chart:

<picture>
  <source srcset="/assets/img/dns/dns-dark.svg" media="(prefers-color-scheme: dark)">
  <img class="flowchart" src="/assets/img/dns/dns.svg" alt="DNS flowchart">
</picture>


## Why should I use encrypted DNS?



## What is DNSSEC and when is it used?

## What is QNAME minimization?

## What is EDNS Client Subnet (ECS)?

## How to enable encrypted DNS?

### Android

- <mark>Provide instructions</mark>

### iOS
DoT and DoH are supported natively by installation of profiles (through mobileconfig files opened in *Safari*).
After installation, the encrypted DNS server can be selected in *Settings &rarr; General &rarr; VPN and Network &rarr; DNS*. **Signed profiles** are offered by [AdGuard](https://adguard.com/en/blog/encrypted-dns-ios-14.html) and [NextDNS](https://apple.nextdns.io/)

### MacOS

- <mark>Provide instructions</mark>

### Windows
Users can turn on DoH natively supported by Windows 10 or later by following [this guide](https://docs.microsoft.com/en-us/windows-server/networking/dns/doh-client-support).

### Linux
Linux distributions don't have this by default, so installing [dnscrypt-proxy](https://github.com/DNSCrypt/dnscrypt-proxy) and [configuring](https://wiki.archlinux.org/title/Dnscrypt-proxy#Local_DNS_cache_configuration) it is still necessary.


## DNS Server recommendations:

{% include recommendation-table.html data='dns' %}

## DNS server criteria
- Must support [DNSSEC](/dns/#what-is-dnssec-and-when-is-it-used)
- Must have [anycast](https://en.wikipedia.org/wiki/Anycast#Addressing_methods) support
- [QNAME Minimization](/dns/#what-is-qname-minimization)
