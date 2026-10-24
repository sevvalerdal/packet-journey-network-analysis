# Scenario C — Default Gateway Misconfiguration

## Hypothesis

An IPv4 host with a correct local IP address and subnet mask can reach on-subnet peers directly, even when its configured default gateway is wrong. Off-subnet traffic depends on the configured gateway.

## Controlled change

On **PC0 only**, changed the default gateway:

```text
Correct: 192.168.10.1
Wrong:   192.168.10.254
```

PC0's own IPv4 address remained `192.168.10.10/24`. Router0 addresses, cabling and switch configuration were not modified.

## Tests

| PC0 destination | What it tests | With wrong gateway |
|---|---|---|
| PC1 `192.168.10.20` | Same-subnet delivery | **4/4 replies** |
| Router0 `192.168.10.1` | Reachability of real router on local LAN | **4/4 replies** |
| Server0 `192.168.20.10` | Off-subnet delivery | **0/4 replies; timeouts** |

After restoring the gateway to `192.168.10.1`, PC0 → Server0 returned **4/4 replies**, with **TTL 127**.

## Explanation

PC0 compares each destination IP against its local subnet. It can deliver to PC1 and the router's **on-link address** without relying on the default gateway setting. For Server0 on the `192.168.20.0/24` network, PC0 instead attempts to use the configured next hop. The deliberately unused `.254` address is incorrect, so the off-subnet test fails.

The observed failure **does not** mean the physical link from PC0 to the switch is down; local connectivity remains functional.

## Evidence

- `screenshots/scenario-c-wrong-gateway/01-wrong-gateway-settings.png`
- `screenshots/scenario-c-wrong-gateway/02-local-ok-remote-failed.png`
- `screenshots/scenario-c-wrong-gateway/03-gateway-restored-remote-test.png`

The final project `.pkt` is saved **after restoring** the correct gateway.
