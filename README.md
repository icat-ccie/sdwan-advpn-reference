# FOS Reference Configuration: SD-WAN with ADVPN

## Intro

This repository contains FortiOS configuration templates for different SD-WAN topologies with ADVPN.
All the templates are written in Jinja. Deployment variables can be defined in YAML, as demonstrated in
several examples (see `example-*.yml` under each topology).

A simple renderer written in python is also provided (see `render_config.py`).
It can render a single template for a specified device or an entire topology for all devices defined in YAML.

Usage example:

```
% ./render_config.py -d topo1-separate-underlays/example-1reg-2hub.yml -t topo1-separate-underlays
Rendering device dc1_fgt...
   Rendering 01-hub-overlay.conf.j2...
   Rendering 02-hub-routing.conf.j2...
   Rendering 03-hub-sdwan.conf.j2...
   Rendering 04-hub-firewall.conf.j2...
Rendering device dc2_fgt...
   Rendering 01-hub-overlay.conf.j2...
   Rendering 02-hub-routing.conf.j2...
   Rendering 03-hub-sdwan.conf.j2...
   Rendering 04-hub-firewall.conf.j2...
Rendering device branch1_fgt...
   Rendering 01-edge-overlay.conf.j2...
   Rendering 02-edge-routing.conf.j2...
   Rendering 03-edge-sdwan.conf.j2...
   Rendering 04-edge-firewall.conf.j2...
Rendering device branch2_fgt...
   Rendering 01-edge-overlay.conf.j2...
   Rendering 02-edge-routing.conf.j2...
   Rendering 03-edge-sdwan.conf.j2...
   Rendering 04-edge-firewall.conf.j2...
Rendering complete.
% ls -l out
total 80
-rw-r--r--@ 1 dperets  staff  11638 Dec  6 09:59 branch1_fgt
-rw-r--r--@ 1 dperets  staff  11638 Dec  6 09:59 branch2_fgt
-rw-r--r--@ 1 dperets  staff   4483 Dec  6 09:59 dc1_fgt
-rw-r--r--@ 1 dperets  staff   4483 Dec  6 09:59 dc2_fgt
```

## Topology 1: Separate Underlays

Requirements: FOS 6.4.1+

This base topology includes two separate underlay transport networks.
A typical example would be: Internet and MPLS.

The topology is multi-regional, with arbitrary number of regions and Regional Hubs.

### Features:

Corporate Access (CPE to CPE):

- Separate IPSEC overlay between Edge CPEs (Spokes) and each Regional Hub, over each underlay transport
  (hence, with 2 underlays + 2 Regional Hubs, each Edge CPE builds 4 IPSEC tunnels)

- Full-mesh IPSEC overlays between Regional Hubs, over each underlay transport  

- IBGP sessions over each overlay (both within regions and between regions)

- ADVPN shortcuts are enabled within each overlay

- ADVPN shortcuts are not possible between different overlays (e.g. between Internet and MPLS)

- Cross-regional ADVPN shortcuts are enabled

- The traffic prefers to flow via ADVPN shortcut, when possible

- Overlay stickiness: the traffic prefers to stay within the same overlay end-to-end

- Cross-overlay traffic is possible as last resort, via the Hub
  (e.g. if source CPE is connected only to Internet and target CPE - only to MPLS)

Internet Access:

- Direct Internet Access (DIA) from each Edge CPE, using 1st underlay only (Internet)

- Remote Internet Access (RIA) via Regional Hub, using overlays over 2nd underlay only (MPLS)

- Application-aware SD-WAN rules with different SLA targets per application
