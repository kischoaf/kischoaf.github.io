---
title: Immich on Unraid
layout: page
---

## Overview

A self-hosted photo and video library running on Unraid, accessible remotely through a Cloudflare Tunnel with no open ports on the router. It manages around 11,000 photos and videos and serves as a full replacement for cloud photo storage. The interesting part of this project is not the application itself — it is how the network access and data persistence are architected around it.

## What I Built

Immich runs as a multi-container stack on Unraid via Docker Compose. The stack includes the Immich server, a microservices container, a machine learning container for photo recognition and search, Redis for caching, and PostgreSQL as the primary database. Each component runs in its own container, and the data that needs to survive — the PostgreSQL database and the photo library itself — is stored in Docker volumes mapped to persistent storage on the Unraid array.

**Remote Access via Cloudflare Tunnel**

Rather than forwarding a port on the router, remote access runs through a Cloudflare Tunnel (`cloudflared`) deployed as an additional container in the stack. The tunnel creates an outbound-only connection from my Unraid server to Cloudflare's edge network. Incoming requests hit Cloudflare's edge, travel through the tunnel, and reach Immich internally. The router has no inbound ports open for this service.

Cloudflare handles TLS termination and certificate management at the edge, so the service is reachable over HTTPS without any self-signed certificate setup on my end. The tradeoff is that traffic passes through Cloudflare's infrastructure before reaching my server, but for a photo library the privacy calculus still comes out well ahead of using a cloud service directly.

**Data Persistence Design**

The PostgreSQL database and media library are stored in Docker volumes that are decoupled from the container lifecycle. This distinction — between the application layer (containers) and the data layer (volumes on persistent storage) — ended up mattering in practice, which is covered in the Results section.

## Technical Depth

Immich's Docker Compose configuration is more involved than a typical single-container deployment. The machine learning container in particular has specific resource requirements, and the compose file needs to wire the containers together correctly — shared networks, volume mounts, environment variables for database credentials, and dependency ordering so the database is healthy before the application containers try to connect to it.

The Cloudflare Tunnel architecture is worth understanding at the network level. Traditional remote access to a home service requires inbound port forwarding: a port on the public IP maps to an internal host, and anything that can reach your public IP can attempt a connection. A Cloudflare Tunnel inverts this. The `cloudflared` daemon running inside the network initiates and maintains an outbound connection to Cloudflare's edge. There is no listening socket exposed to the internet on the router. Access control, DDoS protection, and TLS are all handled at the edge before traffic ever reaches the home network. For a homelab service, this is a meaningfully better security posture than port forwarding.

DNS for the service resolves through Cloudflare, and the tunnel configuration maps the public hostname to the internal Immich address. Combined with the AdGuard Home DNS infrastructure running on the same network, local requests to the same hostname resolve internally rather than hairpinning out through the tunnel.

## Problem It Solves

Cloud photo storage trades convenience for ongoing cost and, more directly, for control over where your data lives and who can access it. With 11,000 photos and videos — and the number growing — the cost of cloud storage compounds over time, and the data is subject to whatever the provider decides to do with it.

Self-hosting solves both of those, but it introduces the problem of remote access: how do you reach a service running on hardware at home from anywhere, without punching holes in the network that create other risks? The Cloudflare Tunnel is the answer to that specific problem. It makes the service reachable from anywhere with a real HTTPS connection while keeping the network perimeter intact.

## Results

The stack has been running since 2023 and currently manages approximately 11,000 photos and videos. It handles automatic backup from mobile devices, facial recognition, and full-text search across the library.

At one point the application layer broke badly enough that I rebuilt it from scratch. The PostgreSQL database, stored in a Docker volume on the Unraid array, was completely unaffected. All photo metadata, albums, and user data survived. After the rebuild, Immich reattached to the existing database and the library came back exactly as it was. The only thing that was gone was the application itself, which was the part that was supposed to be replaceable.

That outcome was not an accident — it was the result of keeping persistent data in named Docker volumes on the Unraid array rather than inside the container filesystem. But it is one thing to understand that principle and another to have it validated under a real failure. The rebuild confirmed that the data persistence design was correct and that the operational risk of running this application at home is lower than it might appear.

## Photos

<!-- Drop your images in /assets/img/projects/ and uncomment the lines below -->
<!-- ![Description](../assets/img/projects/your-image-1.jpg) -->
<!-- ![Description](../assets/img/projects/your-image-2.jpg) -->

## Documentation

<!-- Link to a PDF or embed a document if you have one -->
<!-- [View Documentation](../assets/docs/your-document.pdf) -->
