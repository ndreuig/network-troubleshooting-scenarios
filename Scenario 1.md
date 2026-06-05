# Scenario 1 — User Cannot Access the Internet (DNS Failure)

## Description

The user reports they cannot open any websites, but their internet connection appears to be working. The goal is to identify and fix the root cause.

---

## Environment

- **Machine:** Windows 10
- **Adapter:** Intel Dual Band Wireless-AC 8260 (Wi-Fi)
- **IP Address:** 192.168.1.5
- **Default Gateway:** 192.168.1.1
- **Normal DNS Server:** 192.168.1.1

---

## Steps Performed

### Step 1 — Check Network Configuration (Normal State)

Ran `ipconfig /all` to view the current network adapter settings.

**Result:** Everything looked normal. The machine had a valid IP address, gateway, and DNS server all pointing to `192.168.1.1`.

> Screenshot: `1-ipconfig-normal.PNG`
> ![](screenshots/1-ipconfig-normal.PNG)
---

### Step 2 — Verify Connectivity (Normal State)

Ran two ping tests to confirm internet access was working before breaking anything:

- `ping 8.8.8.8` → Success (pinging by IP)
- `ping google.com` → Success (pinging by domain name)

**Result:** Both worked. DNS was resolving correctly.

> Screenshot: `2-ping-normal.PNG`
> ![](screenshots/2-ping-normal.PNG)

---

### Step 3 — Confirm DNS Resolution (Normal State)

Ran `nslookup google.com` to verify DNS was resolving domain names.

**Result:** Returned multiple valid IP addresses for google.com using DNS server `192.168.1.1`.

> Screenshot: `3-nslookup-normal.PNG`
> ![](screenshots/3-nslookup-normal.PNG)

---

### Step 4 — Simulate the Problem (Break DNS)

Changed the DNS server to a fake, non-existent address (`1.2.3.4`) to simulate a misconfigured DNS.

Then ran:
- `ping 8.8.8.8` → **Success** (direct IP still works — internet is up)
- `ping google.com` → **Failed** ("Ping request could not find host google.com")

**Finding:** The machine can reach the internet by IP, but cannot resolve domain names. This points directly to a DNS problem.

> Screenshot: `4-ping-dns-broken.PNG`
> ![](screenshots/4-ping-dns-broken.PNG)

---

### Step 5 — Confirm DNS Failure with nslookup

Ran `nslookup google.com` while DNS was broken.

**Result:** DNS request timed out repeatedly. The server address showed `1.2.3.4` (the fake DNS), and all queries failed.

> Screenshot: `5-nslookup-broken.PNG`
> ![](screenshots/5-nslookup-broken.PNG)

---

### Step 6 — Fix the DNS and Flush Cache

Restored the correct DNS server (`192.168.1.1`), then ran:

```
ipconfig /flushdns
```

**Result:** "Successfully flushed the DNS Resolver Cache." — old broken DNS entries were cleared.

> Screenshot: `6-flushdns.PNG`
> ![](screenshots/6-flushdns.PNG)

---

### Step 7 — Verify Everything Works Again

Ran `ping google.com` and `nslookup google.com` to confirm the fix.

**Result:** Both succeeded. Google.com resolved correctly, and ping returned replies with 0% packet loss.

> Screenshot: `7-ping-restored.PNG`
> ![](screenshots/7-ping-restored.PNG)

---

## Root Cause

The DNS server was set to an invalid IP address (`1.2.3.4`). This caused all domain name resolution to fail, making websites unreachable — even though the internet connection itself was working fine.

---

## Fix Applied

1. Restored the correct DNS server to `192.168.1.1`
2. Ran `ipconfig /flushdns` to clear the DNS cache

---

## Key Takeaway

> If a user can ping an IP address but not a domain name, the problem is almost always **DNS** — not the internet connection itself.
