---
title: AdGuard Home DNS Infrastructure
layout: page
---

## Overview

A dedicated DNS infrastructure running on a Raspberry Pi that handles name resolution for every device on my home network. It filters ads and malicious domains, resolves internal hostnames for self-hosted services, and has been running continuously since 2023. This is less of a "block ads" project and more of a deliberate DNS layer sitting between my network and the public internet.

## What I Built

AdGuard Home runs on a dedicated Raspberry Pi — not a VM, not a container on a shared host. The Pi is the sole purpose of that hardware, which keeps the service isolated and makes its availability predictable.

The Ubiquiti Dream Machine handles DHCP for the network and is configured to hand out the Raspberry Pi's IP as the DNS server for all clients. Every device on the network — phones, laptops, smart home gear, anything that gets a DHCP lease — resolves DNS through AdGuard without any per-device configuration.

**Upstream DNS**

For upstream resolution, I use Cloudflare (1.1.1.1) as the primary and Quad9 (9.9.9.9) as the secondary. Cloudflare is fast and has consistent latency. Quad9 adds a layer of malware and threat-intelligence filtering at the resolver level, so domains associated with known malicious infrastructure get blocked before they even reach my blocklists. Fallback DNS is configured with Quad9 and Google (8.8.8.8) in case the primary upstreams are unreachable.

**Internal Name Resolution**

Beyond filtering, AdGuard handles internal DNS records for my self-hosted services. `server.lan` resolves to my home server, which runs Immich and other services. `computer.lan` resolves to my desktop PC and is what I use for RDP access. This is a simple form of split-horizon DNS — internal hostnames that only make sense inside my network resolve correctly without touching public DNS, and I do not have to remember or type IP addresses to reach local services.

**Blocklists**

I run the default AdGuard blocklists plus several community-maintained lists I added over time. The community lists target specific categories the defaults are lighter on — trackers, telemetry endpoints, and a few domain categories I wanted to filter more aggressively.

## Technical Depth

The integration between the Dream Machine and AdGuard is worth explaining. The Dream Machine is the DHCP server and therefore controls what DNS server every client receives. By setting the DNS server in the Dream Machine's DHCP configuration to point at the Raspberry Pi, all resolution flows through AdGuard automatically. If AdGuard is unavailable, the fallback DNS addresses I configured (Quad9 and Google) ensure clients can still resolve names — they just bypass filtering while the primary is down.

The internal DNS records for `server.lan` and `computer.lan` are static A records configured directly in AdGuard. When a client queries `server.lan`, AdGuard answers from its local records rather than forwarding the query upstream. This keeps internal traffic internal and means I can reach services by name rather than IP from anywhere on the network.

The upstream choice to run plain DNS rather than DNS-over-HTTPS or DNS-over-TLS is a deliberate tradeoff. In a home environment with a trusted local resolver, the primary privacy benefit of encrypted DNS — preventing your ISP from seeing your queries — is already partially addressed by centralizing resolution through a controlled server. The added complexity of encrypted upstream configuration was not worth it for this use case.

The Raspberry Pi's role as dedicated hardware (rather than a service running on a shared machine) means a reboot of any other device on the network does not affect DNS availability. It is a small architectural decision, but it matters for something that everything else depends on.

## Problem It Solves

Without this setup, DNS resolution on the network was whatever each device's default was — usually the ISP's resolver, or whatever the router was configured to forward to. That means no filtering, no visibility into what domains are being queried, and no way to reach internal services by name.

The filtering aspect addresses the practical nuisance of ads and trackers across all devices at once, including ones that do not support per-app ad blocking (smart TVs, IoT devices, game consoles). But the more useful outcome for day-to-day use is the internal name resolution — being able to type `server.lan` in a browser or RDP client and reach the right machine without managing a hosts file on every device.

## Results

The infrastructure has been running continuously since 2023. As of current dashboard stats, it has handled approximately 45,000 DNS queries with around 8,600 blocked by filters — a block rate of roughly 19%. That figure reflects real network traffic across all devices on the network, not a test environment.

The internal DNS records have made local service access noticeably simpler. Reaching Immich, RDP-ing to my desktop, or pointing a client at any internal service is a hostname lookup rather than a remembered IP address.

Running this has also built familiarity with DNS as an operational layer — not just as a name-to-IP lookup mechanism, but as a place where traffic policy, visibility, and service routing can be applied across an entire network from a single point.

## Photos

<!-- Drop your images in /assets/img/projects/ and uncomment the lines below -->
<!-- ![Description](../assets/img/projects/your-image-1.jpg) -->
<!-- ![Description](../assets/img/projects/your-image-2.jpg) -->

## Documentation

<!-- Link to a PDF or embed a document if you have one -->
<!-- [View Documentation](../assets/docs/your-document.pdf) -->
