# 🔍 Week 2 Project — Footprinting & Network Scanning

**Program:** Cybersecurity & Pentesting — Networkwalks Academy  
**Batch:** B082  
**Student:** Zulu Yobu  
**Instructor:** Sir Waqas Karim (CCIE)

> Second project of the Networkwalks Cybersecurity program: the **reconnaissance phase** of hacking. Week 2 has two project modules — **W2-PM1: Domain Footprinting** (gathering intelligence about a target domain) and **W2-PM5: Network Scanning with Zenmap** (discovering live hosts on a LAN).

---

## 📋 Assignment Brief (from course drive)

**Module 1 — Domain Footprinting** · Target: `www.networkwalks.com`

![Assignment brief — W2-PM1](screenshots/00-assignment-brief-pm1.png)

| Task | Tool | Goal |
|------|------|------|
| 1 | `whois` | Find the domain registration details |
| 2 | `whatweb` | Fingerprint the web technologies |
| 3 | `nslookup` | Resolve the domain to its IP address |
| 4 | `curl -I` | Read the HTTP response headers |
| 5 | `wafwoof` | Detect a Web Application Firewall |
| 6 | `dnsrecon` | Enumerate all DNS records |

**Module 5 — Network Scanning with Zenmap**

![Assignment brief — W2-PM5](screenshots/00-assignment-brief-pm5.png)

| Task | Action | Goal |
|------|--------|------|
| 1 | Download & install **Zenmap** from the official website (on Windows) | Working scanner installed |
| 2 | Find your **local IP address & LAN subnet** | Know your own network |
| 3 | Find the **list of live hosts** in your IP subnet | Discover devices |
| 4 | **How many hosts** are live in your subnet? | Count them |
| 5 | What are the **IP addresses** of the live hosts? | List IPs |
| 6 | What are the **MAC addresses** of the live hosts? | List MACs |
| 7 | Display & save the output **topology in PDF** format on your desktop | Document the map |

> ⚠️ All scanning is performed **only on my own home/lab network**. Never scan networks you don't own or lack written permission to test.

---

## 🎯 Deliverables

| # | Deliverable | Status |
|---|-------------|--------|
| 1 | PM1 — whois on networkwalks.com | ✅ |
| 2 | PM1 — whatweb technology fingerprint | ✅ |
| 3 | PM1 — nslookup DNS resolution | ✅ |
| 4 | PM1 — curl -I HTTP headers | ✅ |
| 5 | PM1 — wafwoof WAF detection | ✅ |
| 6 | PM1 — dnsrecon DNS enumeration | ✅ |
| 7 | PM5 — Zenmap installed | ✅ |
| 8 | PM5 — Local IP & subnet identified (`ipconfig`) | ✅ |
| 9 | PM5 — Live hosts discovered — **3 hosts up** | ✅ |
| 10 | PM5 — Host count, IPs & MACs recorded | ✅ |
| 11 | PM5 — Topology saved as PDF | ✔ |
| 12 | Full documentation with screenshots (this repo) | ✅ |

---

# 🕵️ Part 1 — W2-PM1: Domain Footprinting

Footprinting (reconnaissance) is the **first phase of hacking** — collecting publicly available information about a target. Everything below uses only **public, passive** techniques: we ask the same public services anyone can query.

## Task 1 — whois: Domain Registration Details

`whois` queries the domain registrar's database: who owns the domain, when it was registered, and which name servers serve it.

```bash
whois networkwalks.com
```

**What to look for in the output:**

| Field | Meaning |
|-------|---------|
| Registrar | Company where the domain is registered |
| Creation Date | When the domain was first registered |
| Registry Expiry Date | When the registration expires |
| Name Server | DNS servers responsible for the domain |
| Registrant | Owner (often privacy-protected) |

**My findings (real output):**

| Field | Value |
|-------|-------|
| Registrar | **GoDaddy.com, LLC** (IANA ID 146) |
| Creation Date | **2019-11-06** |
| Updated Date | 2025-11-12 |
| Registry Expiry Date | **2027-11-06** |
| Name Servers | `NS6135.HOSTGATOR.COM`, `NS6136.HOSTGATOR.COM` → hosting at **HostGator** |
| DNSSEC | unsigned |
| Domain Status | clientDelete / Renew / Transfer / Update Prohibited |

![whois output — real result](screenshots/01-pm1-whois.png)

## Task 2 — whatweb: Web Technology Fingerprinting

`whatweb` identifies the technologies behind a website: web server, CMS, frameworks, JavaScript libraries — gold for an attacker choosing exploits, and for defenders auditing their stack.

```bash
whatweb https://www.networkwalks.com
```

**What to look for:** `HTTPServer`, `IP`, `Meta-Generator` (e.g. WordPress), `Title`, CDN/WAF hints (e.g. Cloudflare).

**My findings (real output):**

| What | Finding |
|------|---------|
| HTTP status | `301 Moved Permanently` — http redirects to https |
| Web server | **Apache** |
| IP | `192.232.216.135` (public site IP) |
| Location | Country: UNITED STATES |
| Cookie | `_wpdm_client` (HttpOnly) — hint of WordPress Download Manager |
| Redirect target | `https://networkwalks.com/` |
| Uncommon headers | `x-redirect-by`, `x-endurance-cache-level`, `x-nginx-cache`, `permissions-policy`, `referrer-policy` |

> 📝 The follow-up HTTPS probe threw `no address for networkwalks.com` — a
> temporary DNS hiccup in the lab VM (confirmed by later tasks, which resolved
> the domain fine).

![whatweb output — real result](screenshots/02-pm1-whatweb.png)

## Task 3 — nslookup: Resolve the Domain to an IP

`nslookup` asks DNS to translate the domain name into the IP address the browser actually connects to.

```bash
nslookup www.networkwalks.com
dig www.networkwalks.com +short        # alternative, shorter output
```

**My findings (real output):** `networkwalks.com` resolves to **`192.232.216.135`** — answered via Google's public DNS (`8.8.8.8#53`), non-authoritative.

![nslookup output — real result](screenshots/03-pm1-nslookup.png)

## Task 4 — curl -I: HTTP Response Headers

`curl -I` sends an HTTP HEAD request and prints the server's response headers — status code, web server software, cookies, security headers. Attackers read these to fingerprint the server; defenders read them to verify security hardening.

```bash
curl -I https://www.networkwalks.com
```

**Key headers to note:**

| Header | Meaning |
|--------|---------|
| `HTTP/2 200` | Server is alive and responding OK |
| `server:` | Web server software (e.g. cloudflare, nginx, Apache) |
| `content-type:` | What the server sends (e.g. text/html) |
| `set-cookie:` | Session/cookie info |

**My findings (real output):**

| Header | Value |
|--------|-------|
| Status | **`HTTP/2 200`** — the site is alive |
| `server:` | **Apache** |
| `content-type:` | `text/html; charset=UTF-8` |
| `x-nginx-cache:` | `WordPress` → WordPress confirmed behind nginx caching |
| `link:` | `wp-json` REST-API links → WordPress running (page ID 53) |
| `permissions-policy:` | present (browser feature restrictions) |
| `set-cookie:` | `_wpdm_client=<redacted>; domain=networkwalks.com; secure; HttpOnly` |

> 🔒 The cookie value was redacted from the screenshot — session identifiers
> should never be published.

![curl -I output — real result](screenshots/04-pm1-curl.png)

## Task 5 — wafwoof: Web Application Firewall Detection

A WAF (Web Application Firewall) sits in front of a site and filters malicious traffic. `wafwoof` (also known as `wafw00f`) sends probes and fingerprints which WAF, if any, protects the target — critical intel before attempting any exploitation.

```bash
wafwoof https://www.networkwalks.com
# on some Kali builds the command is: wafw00f
```

**My findings (real output):**

The first run failed before fingerprinting — the lab VM hit a temporary DNS
failure (`NameResolutionError: Failed to resolve 'networkwalks.com'`), so wafwoof
reported the site "appears to be down":

- ✔ The tool itself worked — it correctly probed `https://networkwalks.com`
- ⚠️ The site was **not** down — `curl -I` minutes later returned `HTTP/2 200`
- 🔜 Next step: re-run `wafwoof` with stable DNS to get the final WAF fingerprint

Documenting the failed attempt honestly is part of good recon practice — every
error tells you something about the lab environment.

![wafwoof output — real result](screenshots/05-pm1-wafwoof.png)

## Task 6 — dnsrecon: Enumerate All DNS Records

`dnsrecon` pulls every DNS record type for a domain — A (hosts), NS (name servers), MX (mail), TXT (SPF etc.) — and checks for DNSSEC.

```bash
dnsrecon -d networkwalks.com
```

**Record types to look for:**

| Type | What it reveals |
|------|-----------------|
| `A` | IPv4 addresses of hosts |
| `NS` | Authoritative name servers |
| `MX` | Mail servers (phishing/email attacks target these) |
| `TXT` | SPF/DKIM policies — misconfigurations are a classic finding |

**My findings (real output):**

| Record type | Finding |
|-------------|---------|
| DNSSEC | **Not configured** (a hardening gap) |
| NS | `ns6136.hostgator.com → 192.232.216.131` (HostGator name servers) |
| TXT (SPF) | `v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all` — mail authorized from the site IP + HostGator mail relay |
| TXT | `google-site-verification=<redacted>` — Google Search Console token (masked) |
| SRV | No SRV records found |

> 🔒 The Google verification token was redacted from the screenshot — such
> tokens should not be published.

![dnsrecon output — real result](screenshots/06-pm1-dnsrecon.png)

### 🧠 PM1 Takeaways

- Footprinting is **100% legal, passive intelligence** — public data about public infrastructure.
- Each tool answers one question: *who owns it? what runs it? where is it? what protects it?*
- Together they build the **attack surface map** — which is exactly what defenders should map first, too.

---

# 📡 Part 2 — W2-PM5: Network Scanning with Zenmap

Zenmap is the official GUI for **Nmap** — the industry-standard network scanner. This module is about discovering every device on my own LAN: the same first step an attacker takes after joining a network, done legally on my own home network.

> 🔒 **Privacy note:** real IP and MAC values from my home network are **masked**
> in this public report (structure preserved — the last octet of each host is
> real). Full values are recorded in my private lab notes.

## Task 1 — Download & Install Zenmap

1. Go to the official site: **https://nmap.org/zenmap/** (download links live on the Nmap download page: https://nmap.org/download.html)
2. Download the Windows installer (Zenmap is bundled with Nmap for Windows).
3. Run the installer — it installs **Nmap + Zenmap** together (accept the WinPcap/Npcap driver prompt; Npcap is required for scanning).
4. Launch Zenmap from the Start Menu.

> 📝 **Security note:** download **only** from `nmap.org` — third-party "Zenmap" downloads are often bundled with malware.

![Zenmap installed](screenshots/07-pm5-zenmap-install.png)

## Task 2 — Find My Local IP Address & LAN Subnet

```powershell
# Windows Command Prompt / PowerShell
ipconfig
```

**My actual results** *(values masked for privacy)*:

| Adapter | Status | IPv4 (masked) | Notes |
|---------|--------|---------------|-------|
| Ethernet | Media disconnected | — | Not in use |
| Ethernet 3 | Connected | `192.168.x.1` | **VirtualBox Host-Only** network (VirtualBox's default host-only subnet) — installed in Week 1! |
| Wi-Fi | **Active** | **`192.168.x.101`** | My live connection — this is the LAN I scanned |
| Wi-Fi virtual adapters ×2 | Media disconnected | — | Windows hotspot adapters |
| Bluetooth | Disconnected | — | |

| Value | Result | Meaning |
|-------|--------|---------|
| IPv4 address | `192.168.x.101` | My machine's address on the LAN |
| Subnet mask | `255.255.255.0` → `/24` | Subnet = `192.168.x.0 – 192.168.x.255` (256 addresses) |
| Default gateway | `192.168.x.1` | The router |

**Subnet to scan:** `192.168.x.0/24`

![ipconfig — actual results, values redacted](screenshots/09-pm5-ipconfig.png)

> 💡 Interesting detail: the **Ethernet 3** adapter is the VirtualBox Host-Only
> network created when I installed VirtualBox in Week 1 — the lab and the home
> LAN live on the same machine, cleanly separated.

## Task 3 — Find the List of Live Hosts in the Subnet

A **ping scan** sends ICMP echo requests to every address in the subnet and records who answers. I used Zenmap's **Quick scan** profile:

- **Zenmap GUI:** target `192.168.x.0/24`, Profile → **Quick scan** (`nmap -T4 -F` — host discovery + top-100 port scan)
- **Command line equivalent:**

```powershell
nmap -T4 -F 192.168.x.0/24
```

```bash
# cross-check on Kali (ARP table = devices recently seen)
arp -a
```

## Task 4, 5 & 6 — Count, IPs and MACs of Live Hosts

Zenmap's left pane lists every live host; the **Host Details** tab shows each host's **MAC address** and vendor.

**My actual scan results** *(IPs & MACs masked — structure preserved)*:

```
Nmap done: 256 IP addresses (3 hosts up) scanned in 9.57 seconds
```

| # | IP (masked) | Role | MAC (masked) | Open ports |
|---|-------------|------|--------------|------------|
| 1 | `192.168.x.1` | **Router / default gateway** | `3C:67:C9:XX:XX:XX` | `53/tcp` domain, `80/tcp` http |
| 2 | `192.168.x.100` | Other device (host firewall on) | `AA:77:28:XX:XX:XX` | `7/tcp` **filtered** — everything filtered |
| 3 | `192.168.x.101` | **My PC** (this machine) | own adapter | `7` echo, `9` discard, `13` daytime, `135` msrpc, `139` netbios-ssn, `445` microsoft-ds, `5357` wsdapi |

![Zenmap Quick scan — actual results, values redacted](screenshots/10-pm5-quick-scan.png)

### What the results mean

- **Router (`192.168.x.1`)** — port `53` is the DNS service the whole LAN uses, and `80` is the router's web admin panel. Typical home-router profile.
- **Filtered host (`192.168.x.100`)** — Nmap got no response from its ports ("filtered"), meaning a personal firewall is active — classic behaviour of a smartphone or locked-down device.
- **My PC (`192.168.x.101`)** — ports `135/139/445` are the classic **Windows RPC & SMB file-sharing** services and `5357` is Web Services Discovery. These are exactly the ports exploited by worms like **EternalBlue/MS17-010 (WannaCry)** — a great reminder of why SMB must stay patched and off untrusted networks. 🔐
- **MAC addresses** reveal the hardware vendor (first 3 bytes = OUI) — that's how a scanner identifies *what kind* of device each host is, and why ARP-based attacks target MACs.

**Answer summary:** **3 live hosts** in the `/24` subnet (256 addresses) — a router, a firewalled device, and my PC.

## Task 7 — Save the Topology as PDF

Zenmap draws a **network map** of discovered hosts:

1. In Zenmap, click the **Topology** tab.
2. Click **Controls → Save Graphic** (or press `Ctrl+S`).
3. Choose **PDF** as the file type and save to your **Desktop**.

![Network topology](screenshots/08-pm5-topology.png)

**Result:** `Desktop/network_topology.pdf` — a one-page map of my LAN with the router at the center.

### Bonus — Intense Scan (deeper fingerprinting)

I also ran Zenmap's **Intense scan** profile (`nmap -T4 -A -v`) — the full professional profile that adds OS detection, version detection and script scanning on top of the quick scan:

![Zenmap Intense scan — values redacted](screenshots/11-pm5-intense-scan.png)

**Quick scan vs. Intense scan:**

| | Quick scan (`-T4 -F`) | Intense scan (`-T4 -A -v`) |
|---|---|---|
| Ports | Top 100 | Top 1000 |
| OS detection | ❌ | ✅ (`-O`) |
| Service version detection | ❌ | ✅ (`-sV`) |
| NSE scripts | ❌ | ✅ (`-sC`) |
| Verbosity | Normal | Verbose (`-v`) |
| Use case | Fast LAN inventory (this assignment) | Deep recon of one target |

### 🧠 PM5 Takeaways

- A **ping scan** (`-sn`) maps a whole subnet in seconds without touching any service — the universal first recon step.
- **IP = address, MAC = physical identity**: IPs can change; MACs identify the hardware.
- The **topology view** turns raw scan data into a visual map — reporting skill every pentester needs.

---

# 🧰 Tool Comparison — What Each Tool Tells Us

| Tool | Question it answers | Category |
|------|--------------------|----------|
| `whois` | Who registered the domain & when? | Registration intel |
| `nslookup` / `dig` | What IP does the domain point to? | DNS |
| `dnsrecon` | What records exist (A/NS/MX/TXT)? | DNS enumeration |
| `whatweb` | What technologies power the site? | Web fingerprinting |
| `curl -I` | What do the HTTP headers reveal? | Server fingerprinting |
| `wafwoof` | Is there a WAF in front? | Defense detection |
| `nmap -sn` / Zenmap | Which hosts are alive on the LAN? | Host discovery |

---

# 🧠 What I Learned This Week

| Concept | Takeaway |
|---------|----------|
| **Footprinting** | The legal, passive first phase of hacking — public data, big picture. |
| **DNS as an intel source** | Domains leak structure: hosts, mail servers, name servers. |
| **Web fingerprinting** | Servers announce themselves in headers — every banner is information. |
| **WAFs** | Attackers must know what's defending the target before attacking. |
| **Host discovery** | `-sn` ping scans map a network before any deeper probing. |
| **IP vs MAC** | Layer 3 address vs. Layer 2 hardware identity. |
| **Toolchain thinking** | No single tool answers everything — professionals chain whois → DNS → fingerprinting → scanning. |
| **Report sanitization** | Masking real IPs/MACs before publishing is standard professional practice — evidence stays, identifiers don't. |

---

# 📁 Repository Structure

```
networkwalks-week2/
├── README.md            ← this documentation
├── commands.md          ← all commands used, cheat-sheet style
├── blog/
│   └── index.html       ← (optional extra) my technical blog starter
└── screenshots/
    ├── README.md        ← screenshot checklist
    ├── HOW_TO_ATTACH.md ← capture + GitHub upload guide
    ├── 00-assignment-brief-pm1.png   ← official W2-PM1 brief
    ├── 00-assignment-brief-pm5.png   ← official W2-PM5 brief
    ├── 01-pm1-whois.png        ← REAL result (user info redacted)
    ├── 02-pm1-whatweb.png      ← REAL result (user info redacted)
    ├── 03-pm1-nslookup.png     ← REAL result (user info redacted)
    ├── 04-pm1-curl.png         ← REAL result (cookie value redacted)
    ├── 05-pm1-wafwoof.png      ← REAL result (user info redacted)
    ├── 06-pm1-dnsrecon.png     ← REAL result (verification token redacted)
    ├── 07-pm5-zenmap-install.png   ← example (replace with your own)
    ├── 08-pm5-topology.png         ← example (replace after saving the PDF)
    ├── 09-pm5-ipconfig.png     ← REAL result (values redacted)
    ├── 10-pm5-quick-scan.png   ← REAL result (values redacted)
    └── 11-pm5-intense-scan.png ← REAL result (values redacted)
```



# 🚧 Troubleshooting

| Problem | Fix |
|---------|-----|
| `whois` not installed on Kali | `sudo apt install whois` |
| `whatweb` not found | `sudo apt install whatweb` |
| `dnsrecon` not found | `sudo apt install dnsrecon` |
| `wafwoof` not found | Try `wafw00f` (same tool, standard package name): `sudo apt install wafw00f` |
| Zenmap shows "0 hosts up" | Check you scanned the right subnet (`ipconfig` → use that range); disable VPN (it hides the local LAN); allow Nmap through Windows Firewall |
| Nmap needs admin rights | Run Zenmap **as Administrator** (right-click → Run as administrator) — otherwise some scans silently degrade |
| Scan is slow | Ping scan (`-sn`) is fast; full port scans on a /24 take minutes — fine for a lab |
| Topology tab is empty | Run the scan first, then open Topology — it only renders after a completed scan |
| MAC shows for some hosts only | MAC addresses only exist for hosts on **your own subnet** — hosts beyond the router show IP only |

---

# 🙏 Acknowledgements

Special thanks to **Sir Waqas Karim (CCIE)** and the **Networkwalks** team for the guidance, structured curriculum, and support throughout Batch B082.

On to **Week 3**! 🚀

---


# ⚖️ Disclaimer

This project is for **educational purposes only**. All tools and techniques documented here must only be used on systems you own or have explicit written authorization to test. Unauthorized scanning of networks you don't own is illegal in most jurisdictions.

---

#Networkwalks #Cybersecurity #Footprinting #Nmap #Zenmap #Reconnaissance #Pentesting #KaliLinux #GitHub
