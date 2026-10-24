# Scenario D — Wireshark Loopback ICMP vs Packet Tracer

## Capture setup

- Application: **Wireshark 4.6.9** on macOS
- Capture interface: `Loopback: lo0`
- Traffic produced by: `ping -c 4 127.0.0.1`
- Display filter: `icmp`
- Observed: **4 Echo Requests (Type 8)** and **4 Echo Replies (Type 0)**
- Source/destination IPv4: `127.0.0.1 → 127.0.0.1`
- Displayed TTL: **64** for both requests and replies
- Sequence-number pairs: **0/0, 1/1, 2/2, 3/3** (human-readable sequential representation)

## Why sequence numbers matter

An Echo Reply can be associated with the corresponding Echo Request by its ICMP identifier and sequence number. The Wireshark **frame number** (e.g. frames 45 and 46) is a separate index in the capture and is not the ICMP sequence number.

## What this does and does not confirm

| Question | Packet Tracer | Wireshark loopback |
|---|---|---|
| Can we identify ICMP Echo Request and Reply? | **Yes** — Type 8 and Type 0 in the simulated PDU | **Yes** — Type 8 and Type 0 in captured ICMP |
| Can we inspect source and destination IPv4? | **Yes** — PC0 and Server0 endpoints | **Yes** — both `127.0.0.1`, a different experiment |
| Can we observe gateway ARP? | **Yes** — PC0 requests router's MAC | **No** — loopback does not require Ethernet ARP |
| Can we observe MAC rewrite at a router? | **Yes** — Router0 inbound/outbound | **No** — no routed Ethernet hop in loopback capture |
| Can we observe routed TTL decrement? | **Yes** — 255 → 254 request; 128 → 127 reply | **No** — TTL remained 64 in this loopback test |

These are **different paths and hosts**. We compare protocol behaviors, not claim that Packet Tracer's simulation was captured by Wireshark.

## Evidence and handling

- `screenshots/wireshark/01-loopback-icmp.png`
- `screenshots/wireshark/02-icmp-request-details.png`
- `screenshots/wireshark/03-icmp-reply-details.png`

The capture session contained traffic other than the eight ICMP packets exposed by the display filter. The raw capture is **not part of the repository**. The selected screenshots are sufficient for this scoped experiment.
