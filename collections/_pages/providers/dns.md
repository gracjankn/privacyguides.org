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

3. Using [Wireshark](https://en.wikipedia.org/wiki/Wireshark) we can analyse the results. Enterprise filtering and monitoring solutions (such as those purchased by governments and enterprise) can do the process automatically and without human interaction.

{% include table-unencrypted-dns.html %}

An observer could modify any of these packets.

## What is "encrypted DNS"?




## When should I use encrypted DNS?


## What kinds of encrypted DNS are available?

## When to use encrypted DNS?

HTTPS already provides security from anyone modifying Normal cleartext DNS by requiring the websites to have a valid TLS certificate, although anyone can still see which domains you visited. But that is the case with Encrypted DNS too. As an adversary can use [SNI](https://madaidans-insecurities.github.io/encrypted-dns.html#sni), [OCSP](https://madaidans-insecurities.github.io/encrypted-dns.html#ocsp) or [IP Addresses](https://madaidans-insecurities.github.io/encrypted-dns.html#ip-addresses) of websites to know that anyway. As a result any attempt to bypass censorship evasion using encrypted DNS is still noticed and possibly recorded too.

Which is to say, <mark>**Encrypted DNS alone doesn't provide any real privacy or security benefits and VPN and TOR are more suitable for censorship evasion.**</mark>

After going via the flowchart if you decide to use Encrypted DNS over Normal DNS. We recommend the natively supported methods over third party apps.




<picture>
  <source srcset="/assets/img/dns/dns-dark.svg" media="(prefers-color-scheme: dark)">
  <img class="flowchart" src="/assets/img/dns/dns.svg" alt="DNS flowchart">
</picture>

## Encrypted DNS Resolvers
If you need to use Encrypted DNS Resolvers or change your current DNS provider then listed below are some privacy-centric alternatives to the traditional DNS providers.

All recommended public DNS service providers offer <mark>**DNS over HTTPS (DoH) and DNS over TLS (DoT), with QNAME minimization and DNSSEC**</mark>. For the terms used in the table refer to the [definitions](/providers/dns/#definitions) below.

{% include recommendation-table.html data='dns' %}

## Encrypted DNS on Windows
Users can turn on DoH natively supported by Windows 10 or later by following [this guide](https://docs.microsoft.com/en-us/windows-server/networking/dns/doh-client-support).


## Encrypted DNS on Android

{% for item_hash in site.data.software.dns-android-applications %}
{% assign item = item_hash[1] %}

{% if item.type == "Recommendation" %}
{% include recommendation-card.html %}
{% endif %}
{% endfor %}

## Encrypted DNS on Apple devices

In iOS, iPadOS, tvOS 14 and macOS 11, DoT and DoH were introduced. DoT and DoH are supported natively by installation of profiles (through mobileconfig files opened in *Safari*).
After installation, the encrypted DNS server can be selected in *Settings &rarr; General &rarr; VPN and Network &rarr; DNS*.

* **Signed profiles** are offered by [AdGuard](https://adguard.com/en/blog/encrypted-dns-ios-14.html) and [NextDNS](https://apple.nextdns.io/).

### Glossary
A security protocol for encrypted DNS on a dedicated port 853. Some providers support port 443 which generally works everywhere while port 853 is often blocked by restrictive firewalls.

#### DNS-over-HTTPS (DoH):
Similar to DoT, but uses HTTPS instead, being indistinguishable from "normal" HTTPS traffic on port 443 and more difficult to block. {% include badge.html color="warning" text="Warning" tooltip="DoH contains metadata such as user-agent (which may include system information) that is sent to the DNS server." link="https://tools.ietf.org/html/rfc8484#section-8.2" icon="fas fa-exclamation-triangle" %}

#### DNSSEC:
It creates a secure domain name system by adding cryptographic signatures to existing DNS records. By checking its associated signature, you can verify that a requested DNS record comes from its authoritative name server and wasn’t altered en-route, opposed to a fake record injected in a man-in-the-middle attack.

#### QNAME minimization:
  It's a privacy oriented feature in DNS which aims to limit DNS queries from the recursive resolver to include only as much detail in each query as is required for that step in the resolution process.

#### ECS:
EDNS Client-Subnet (ECS) is an extension to the DNS protocol to include components of the end-user IP address data in requests that are sent to the authoritative DNS servers. This leads to gains in network connection speed as the requested domain's IP returned to the client is usually the closest to them. This is typically used to improve the performance of Content Distribution Networks.

#### DNSCrypt:
With an [open specification](https://dnscrypt.info/protocol/), DNSCrypt is an older, yet robust method for encrypting DNS.

#### Anonymized DNSCrypt:
A [lightweight protocol](https://github.com/DNSCrypt/dnscrypt-proxy/wiki/Anonymized-DNS) that hides the client IP address by using pre-configured relays to forward encrypted DNS data. This is a relatively new protocol created in 2019 currently only supported by [dnscrypt-proxy](/providers/dns/#dns-desktop-clients) and a limited number of [relays](https://github.com/DNSCrypt/dnscrypt-resolvers/blob/master/v3/relays.md).
