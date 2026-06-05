# Scenario 2 — User Cannot Reach a Network Device by IP

## Description

The user reports they cannot reach a specific device on the network by its IP address. The goal is to determine whether the problem is with the local network, a specific device, or the connection to the internet.

---

## Environment

- **Machine:** Windows 10
- **Adapter:** Intel Dual Band Wireless-AC 8260 (Wi-Fi)
- **IP Address:** 192.168.1.5
- **Default Gateway:** 192.168.1.1
- **Target (unreachable device):** 192.168.1.200

---

## Steps Performed

### Step 1 — Check Network Configuration

Ran `ipconfig /all` to find the machine's IP address and default gateway.

**Result:** Machine IP was `192.168.1.5`, gateway was `192.168.1.1`. Network adapter was connected and properly configured.

> Screenshot: `1-ipconfig-gateway.PNG`
> ![](../screenshots/1-ipconfig-gateway.PNG)

---

### Step 2 — Ping the Gateway

Ran `ping 192.168.1.1` to check if the local network (router) was reachable.

**Result:** All 4 packets succeeded with very low latency (1–3ms). The local network is working fine.

> Screenshot: `2-ping-gateway-success.PNG`
> ![](../screenshots/2-ping-gateway-success.PNG)

---

### Step 3 — Ping the Target Device (Simulated Unreachable IP)

Ran `ping 192.168.1.200` to try reaching the target device.

**Result:** "Destination host unreachable" — the device at that IP does not exist or is offline. The reply came from the local machine itself (`192.168.1.5`), meaning the router could not find the target anywhere on the network.

> Screenshot: `3-ping-fake-ip-fail.PNG`
> ![](../screenshots/3-ping-fake-ip-fail.PNG)

---

### Step 4 — Continuous Ping to Confirm Internet Is Working

Ran `ping google.com -t` to verify that internet connectivity was not affected.

**Result:** All 17 packets succeeded with consistent response times (23–27ms, 0% loss). The internet connection is fully functional — the problem is isolated to the specific IP `192.168.1.200`.

> Screenshot: `4-ping-continuous.PNG`
> ![](../screenshots/4-ping-continuous.PNG)

---

### Step 5 — Run Tracert to Map the Route

Ran `tracert google.com` to trace the full path packets take from the machine to Google's servers.

**Result:** The route completed successfully in 20 hops:
- Hop 1: Gateway `192.168.1.1` (local router)
- Hops 2–8: ISP infrastructure and transit nodes
- Hops 9–10, 13–19: Timed out (normal — some routers block ICMP)
- Hop 20: `lcfrai-in-f113.1e100.net` — Google's server at `142.251.127.113`

**Finding:** The path to the internet is healthy. The issue is only with the specific device at `192.168.1.200`.

> Screenshot: `5-tracert-google.PNG`
> ![](../screenshots/5-tracert-google.PNG)

---

## Root Cause

The IP address `192.168.1.200` does not correspond to any active device on the network. No device responded to ping requests, and the router returned "Destination host unreachable." The internet connection and local network were both functioning normally throughout.

---

## Key Takeaway

> When a user cannot reach a device by IP, always confirm the local network and internet still work first. If they do, the problem is with that specific device — it may be offline, powered off, or assigned a different IP than expected.

---

## Diagnostic Checklist Used

| Test | Command | Result |
|------|---------|--------|
| View network config | `ipconfig /all` | IP and gateway confirmed |
| Ping gateway | `ping 192.168.1.1` | Success — local network OK |
| Ping target device | `ping 192.168.1.200` | Failed — device unreachable |
| Continuous ping (internet) | `ping google.com -t` | Success — internet working |
| Trace route | `tracert google.com` | Complete — 20 hops to Google |
