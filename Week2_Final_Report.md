# 📋 Week 2 Final Report — Footprinting & Network Scanning

**Program:** Cybersecurity & Pentesting — Networkwalks Academy  
**Batch:** B082  
**Student:** Zulu Yobu  
**Instructor:** Sir Waqas Karim (CCIE)  
**Date:** 22 August 2026

---

## 1. Executive Summary

This report documents the completion of the Week 2 project, consisting of two
project modules:

1. **W2-PM1 — Domain Footprinting** (target: `networkwalks.com`): collected
   registration, technology, DNS and server information using `whois`,
   `whatweb`, `nslookup`, `curl -I`, `wafwoof` and `dnsrecon`.
2. **W2-PM5 — Network Scanning with Zenmap**: installed Zenmap, identified my
   LAN configuration, discovered **3 live hosts** in the `192.168.x.0/24`
   subnet, recorded their IP/MAC addresses and open ports, and documented the
   topology.

**Key results at a glance**

| Module | Result |
|--------|--------|
| PM1 | networkwalks.com → GoDaddy registration, HostGator hosting, Apache + WordPress stack, IP `192.232.216.135`, DNSSEC unsigned |
| PM5 | 3 live hosts: router (53/80), firewalled device (filtered), my PC (Windows SMB ports) |

> 🔒 **Privacy note:** all personal identifiers (usernames, private IPs, MAC
> addresses, cookie values, verification tokens) have been redacted from this
> report and its screenshots. Target-domain data is public information obtained
> from public DNS/whois services.

---

# Part A — W2-PM1: Domain Footprinting

**Target:** `www.networkwalks.com`  
**Method:** passive reconnaissance using public services (legal, non-intrusive).

## Task 1 — whois: Domain Registration Details

```bash
whois networkwalks.com
```

| Field | Finding |
|-------|---------|
| Registrar | **GoDaddy.com, LLC** (IANA ID 146) |
| Creation Date | **2019-11-06** |
| Updated Date | 2025-11-12 |
| Registry Expiry Date | **2027-11-06** |
| Name Servers | `NS6135.HOSTGATOR.COM`, `NS6136.HOSTGATOR.COM` |
| DNSSEC | unsigned |
| Domain Status | clientDeleteProhibited, clientRenewProhibited, clientTransferProhibited, clientUpdateProhibited |

**Interpretation:** domain registered at GoDaddy, website hosted at HostGator.
The four "client*Prohibited" statuses protect the domain from transfer/deletion.

![Task 1 — whois output](screenshots/01-pm1-whois.png)

## Task 2 — whatweb: Web Technology Fingerprinting

```bash
whatweb networkwalks.com
```

| Finding | Value |
|---------|-------|
| Status | `301 Moved Permanently` (http → https redirect) |
| Web server | **Apache** |
| IP | `192.232.216.135` |
| Country | UNITED STATES |
| Cookies | `_wpdm_client` (HttpOnly) → WordPress Download Manager plugin |
| Uncommon headers | `x-redirect-by`, `x-endurance-cache-level`, `x-nginx-cache`, `permissions-policy`, `referrer-policy` |

**Note:** the automatic HTTPS follow-up probe threw `no address for
networkwalks.com` — a temporary DNS failure inside the lab VM (subsequent tasks
resolved the domain normally).

![Task 2 — whatweb output](screenshots/02-pm1-whatweb.png)

## Task 3 — nslookup: Resolve the Domain to Its IP Address

```bash
nslookup networkwalks.com
```

- DNS server used: `8.8.8.8#53` (Google public DNS, non-authoritative)
- **Result:** `networkwalks.com` → **`192.232.216.135`**

![Task 3 — nslookup output](screenshots/03-pm1-nslookup.png)

## Task 4 — curl -I: HTTP Response Headers

```bash
curl -I https://networkwalks.com
```

| Header | Finding |
|--------|---------|
| Status | **`HTTP/2 200`** — site is up |
| `server` | Apache |
| `content-type` | `text/html; charset=UTF-8` |
| `x-nginx-cache` | `WordPress` → WordPress behind nginx caching |
| `link` | `wp-json` REST API links → WordPress (page ID 53) |
| `permissions-policy` | present |
| `set-cookie` | `_wpdm_client=<redacted>; secure; HttpOnly` *(value masked)* |

**Interpretation:** Apache + nginx cache + WordPress + WordPress Download
Manager — a classic shared-hosting WordPress stack, consistent with HostGator.

![Task 4 — curl -I output](screenshots/04-pm1-curl.png)

## Task 5 — wafwoof: Web Application Firewall Detection

```bash
wafwoof networkwalks.com
```

**Result of this run:** the probe failed before fingerprinting — the lab VM hit
a temporary DNS failure (`NameResolutionError: Failed to resolve
'networkwalks.com'`) and the tool reported the site "appears to be down".

- ✔ Tool executed correctly and probed `https://networkwalks.com`
- ⚠️ Site confirmed UP minutes later (`curl -I` → `HTTP/2 200`)
- 🔜 **Follow-up:** re-run wafwoof with stable DNS to obtain the WAF verdict

**Interpretation:** inconclusive run caused by a lab environment issue (VM DNS),
not by the target. Documented honestly per good recon practice.

![Task 5 — wafwoof output](screenshots/05-pm1-wafwoof.png)

## Task 6 — dnsrecon: Enumerate All DNS Records

```bash
dnsrecon -d networkwalks.com
```

| Record type | Finding |
|-------------|---------|
| DNSSEC | **Not configured** (hardening gap) |
| NS | `ns6136.hostgator.com → 192.232.216.131` |
| TXT (SPF) | `v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all` |
| TXT | `google-site-verification=<redacted>` *(token masked)* |
| SRV | No SRV records found |

**Interpretation:** SPF permits mail from the site IP and HostGator's mail relay
(`websitewelcome.com`). The Google verification token confirms the site owner
uses Google Search Console. DNSSEC not enabled.

![Task 6 — dnsrecon output](screenshots/06-pm1-dnsrecon.png)

### PM1 Summary Table

| Tool | Question answered | Answer |
|------|-------------------|--------|
| whois | Who registered it? | GoDaddy, since 06 Nov 2019, expires 06 Nov 2027 |
| whatweb | What runs it? | Apache + WordPress (HostGator hosting) |
| nslookup | What IP? | `192.232.216.135` |
| curl -I | Server headers? | HTTP/2 200, Apache, WordPress via nginx cache |
| wafwoof | Is there a WAF? | Inconclusive (VM DNS failure) — to be re-run |
| dnsrecon | What DNS records? | HostGator NS, SPF, Google verification TXT, no DNSSEC |

---

# Part B — W2-PM5: Network Scanning with Zenmap

**Scope:** my own home LAN only — authorized scanning of my own network.

## Task 1 — Download & Install Zenmap

Installed the official Nmap/Zenmap Windows bundle from `nmap.org` (includes the
Npcap driver required for scanning).

## Task 2 — Find My Local IP Address & LAN Subnet

```powershell
ipconfig
```

| Item | Result (masked) |
|------|-----------------|
| Active adapter | Wi-Fi |
| IPv4 address | `192.168.x.101` |
| Subnet mask | `255.255.255.0` → `/24` |
| Default gateway | `192.168.x.1` (router) |
| Other adapter | `192.168.x.1` — VirtualBox Host-Only network (from the Week 1 lab) |

![Task 2 — ipconfig, values redacted](screenshots/09-pm5-ipconfig.png)

## Task 3 — Find the List of Live Hosts in the Subnet

```powershell
nmap -T4 -F 192.168.x.0/24      # Zenmap "Quick scan" profile
```

**Result:** `Nmap done: 256 IP addresses (3 hosts up) scanned in 9.57 seconds`

## Tasks 4–6 — Count, IP Addresses and MAC Addresses of Live Hosts

| # | IP (masked) | Role | MAC (masked) | Open ports |
|---|-------------|------|--------------|------------|
| 1 | `192.168.x.1` | Router / gateway | `3C:67:C9:XX:XX:XX` | `53/tcp` domain, `80/tcp` http |
| 2 | `192.168.x.100` | Other device (firewalled) | `AA:77:28:XX:XX:XX` | all filtered |
| 3 | `192.168.x.101` | My PC | own adapter | `7`, `9`, `13`, `135` msrpc, `139` netbios-ssn, `445` microsoft-ds, `5357` wsdapi |

**Answers:** **3 hosts live** in the /24 subnet. Ports `135/139/445` on my PC are
the classic Windows RPC/SMB services — the exact attack surface exploited by
worms such as EternalBlue (WannaCry), which is why SMB must stay patched.

![Tasks 3–6 — Zenmap Quick scan, values redacted](screenshots/10-pm5-quick-scan.png)

### Bonus — Intense Scan

Also ran the **Intense scan** profile (`nmap -T4 -A -v`) — OS detection, service
version detection and NSE scripts on top of the quick scan.

![Bonus — Zenmap Intense scan, values redacted](screenshots/11-pm5-intense-scan.png)

## Task 7 — Save the Topology as PDF

Zenmap → **Topology** tab → **Controls → Save Graphic → PDF** → saved to Desktop
(`network_topology.pdf`). *(PDF attached separately with the submission.)*

---

## 2. Lessons Learned

1. **Footprinting is legal, passive intelligence** — whois, DNS and HTTP headers
   are public data anyone may query; the skill is knowing what each source reveals.
2. **Toolchain thinking** — no single tool answers every question: registration
   (whois) → technology (whatweb) → addressing (nslookup/dnsrecon) → hardening
   (wafwoof) → reachability (curl).
3. **Host discovery is the universal first step** — one ping/quick scan maps an
   entire LAN in seconds.
4. **Everything announces itself** — server headers, DNS records and open ports
   together draw the target's attack surface.
5. **Report sanitization is a professional skill** — masking usernames, private
   IPs, MACs, cookies and tokens before publishing is standard practice.

## 3. Privacy & Ethics Statement

- All scanning was performed **only on my own home network** and against the
  course-designated target domain (`networkwalks.com`).
- All reconnaissance was **passive and non-intrusive** (public services only).
- Personal identifiers were **redacted** from screenshots and this report.
- This work is for **educational purposes**; unauthorized scanning of networks
  you do not own is illegal.

## 4. Acknowledgements

Special thanks to **Sir Waqas Karim (CCIE)** and the **Networkwalks** team for
the guidance and support throughout Batch B082.


---

#Networkwalks #Cybersecurity #Footprinting #Nmap #Zenmap #Pentesting
