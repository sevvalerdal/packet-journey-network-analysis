# Scenario B — Observing a Packet's Journey

## Objective

Observe a single PC0 → Server0 ICMP Echo Request as it traverses a router and then inspect the independently generated Echo Reply. Distinguish final IP addresses from link-level MAC addresses.

## Device addresses

- PC0: `192.168.10.10/24`; gateway `192.168.10.1`
- PC1: `192.168.10.20/24`; gateway `192.168.10.1`
- Router0: `G0/0 = 192.168.10.1/24`, `G0/1 = 192.168.20.1/24`
- Server0: `192.168.20.10/24`; gateway `192.168.20.1`

## Step 1 — Resolve the next hop

- PC0 determines that Server0 (`192.168.20.10`) is **outside** PC0's `192.168.10.0/24` subnet.
- PC0 needs the MAC address of its next hop, the gateway `192.168.10.1`.
- PC0 sends an ARP broadcast with destination MAC `FFFF.FFFF.FFFF` asking for `192.168.10.1`.
- Switch0 delivers the broadcast to other devices on the local LAN; PC1 is not `192.168.10.1`, so it does not reply.
- Router0 responds with its G0/0 MAC `0060.7099.2201`.

Evidence: `screenshots/scenario-b-remote/03-pc0-arp-for-gateway.png` and `04-gateway-arp-reply.png`.

## Step 2 — Deliver the Echo Request to the gateway

- ICMP Echo Request: **Type 8**.
- IPv4 source: `192.168.10.10`; destination: `192.168.20.10`.
- Ethernet source: `0060.3E57.2E55` (PC0); destination: `0060.7099.2201` (Router0 G0/0).
- TTL at PC0 and Router0 inbound: **255**.

Evidence: `05-pc0-icmp-to-gateway.png`, `06-router-icmp-inbound.png`.

## Step 3 — Router forwards to the server LAN

Router0 receives the frame on **G0/0** and sends a new frame on **G0/1**:

| Field | Inbound G0/0 | Outbound G0/1 |
|---|---|---|
| Source MAC | `0060.3E57.2E55` | `0060.7099.2202` |
| Destination MAC | `0060.7099.2201` | `000A.F309.96BE` |
| Source IPv4 | `192.168.10.10` | `192.168.10.10` |
| Destination IPv4 | `192.168.20.10` | `192.168.20.10` |
| TTL | **255** | **254** |

**Readout:** The router rewrites the Ethernet delivery addresses for the next LAN. It **does not** change the source/destination IPv4 addresses in this example. It decreases the IP packet's TTL by one.

Evidence: `07-router-icmp-outbound.png` and `08-router-osi-comparison.png`.

## Step 4 — A new packet travels back

Server0 produces a separate **ICMP Type 0 Echo Reply**:

- Source IPv4: `192.168.20.10`
- Destination IPv4: `192.168.10.10`
- Reply initial TTL: **128**; TTL observed inbound at PC0: **127**.

The request's `255 → 254` and the reply's `128 → 127` are **two independent packet TTL histories**, not one counter that goes down on a round trip.

Evidence: `09-server-echo-reply.png` and `10-pc0-echo-reply-received.png`.

## Reading a PDU details pane

1. Check the **device name** at the top (e.g. Router0).
2. Check **Inbound** vs **Outbound** relative to that device.
3. Under **Ethernet II**, read `SRC ADDR` and `DEST ADDR` (this hop's sender and receiver).
4. Under **IP**, read `SRC IP`, `DST IP` (endpoints) and `TTL` (remaining forwarding limit).
5. Under **ICMP**, check **Type 8** request vs **Type 0** reply.

**Avoid:** describing all MAC values as fixed for the entire journey, or treating the request and reply as the same packet.
