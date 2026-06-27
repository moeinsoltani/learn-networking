---
title: "Phase 16: Tunnels & VPNs"
nav_order: 18
has_children: true
---

# Phase 16: Tunnels & VPNs

This phase builds from plain encapsulation (GRE/IPIP) up to modern encrypted overlays. You'll
learn the cryptography a VPN needs, the two dominant designs (IPsec and WireGuard), the hardest
problem in peer-to-peer VPNs — NAT traversal — and how relays and a coordination plane turn
point-to-point tunnels into a self-configuring mesh. It builds on Lesson 14 (TUN/TAP), Phase 4
(VXLAN), Phase 5 (routing), and Phase 7 (NAT/conntrack).

| # | Lesson | Status |
|---|--------|--------|
| 48 | [Tunneling fundamentals — GRE, IPIP, SIT, FOU](lesson-48-tunneling-fundamentals) | Ready |
| 49 | [VPN cryptography building blocks](lesson-49-vpn-crypto) | Ready |
| 50 | [IPsec and the kernel xfrm framework](lesson-50-ipsec-xfrm) | Ready |
| 51 | [WireGuard fundamentals](lesson-51-wireguard-fundamentals) | Ready |
| 52 | [WireGuard internals & userspace datapaths](lesson-52-wireguard-internals) | Ready |
| 53 | [NAT traversal — STUN, ICE, UDP hole punching](lesson-53-nat-traversal) | Ready |
| 54 | [Relays and fallback paths](lesson-54-relays) | Ready |
| 55 | [Mesh VPNs and coordination/control planes](lesson-55-mesh-vpns) | Ready |
| 56 | [TLS-based VPNs (OpenVPN)](lesson-56-tls-vpns) | Ready |
| 57 | [Capstone — an encrypted mesh across NAT](lesson-57-vpn-capstone) | Ready |
