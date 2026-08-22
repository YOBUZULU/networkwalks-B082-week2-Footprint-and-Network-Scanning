# 🧰 Commands Cheat Sheet — Week 2 Footprinting & Network Scanning

All commands used in W2-PM1 (domain footprinting) and W2-PM5 (Zenmap network scanning).

---

## 🕵️ W2-PM1 — Domain Footprinting (Kali terminal)

```bash
# ── Task 1: whois — registration details ──
whois networkwalks.com
whois networkwalks.com | grep -iE "registrar|creation|expiry|name server"

# ── Task 2: whatweb — web technology fingerprint ──
whatweb https://www.networkwalks.com
whatweb -a 3 https://www.networkwalks.com        # more aggressive fingerprint

# ── Task 3: nslookup — domain → IP ──
nslookup www.networkwalks.com
dig www.networkwalks.com +short                  # compact answer
host www.networkwalks.com                        # simplest

# ── Task 4: curl -I — HTTP response headers ──
curl -I https://www.networkwalks.com
curl -I -L https://www.networkwalks.com          # follow redirects
curl -s -o /dev/null -w "%{http_code}\n" https://www.networkwalks.com   # status only

# ── Task 5: wafwoof — WAF detection ──
wafwoof https://www.networkwalks.com
wafw00f https://www.networkwalks.com             # standard package name on Kali

# ── Task 6: dnsrecon — enumerate DNS records ──
dnsrecon -d networkwalks.com
dnsrecon -d networkwalks.com -t std              # standard enumeration
dig networkwalks.com ANY                         # all records (if server allows)
```

**Tool installs (if missing):**

```bash
sudo apt update
sudo apt install whois whatweb dnsrecon wafw00f
```

---

## 📡 W2-PM5 — Network Scanning with Zenmap (Windows host)

### PowerShell / CMD

```powershell
# ── Task 2: local IP & subnet ──
ipconfig
ipconfig | findstr "IPv4"          # just your IPs
ipconfig | findstr "Gateway"

# local ARP table (devices your PC recently talked to)
arp -a
```

### Nmap / Zenmap (command line runs the same engine)

```powershell
# ── Task 3: ping scan the whole subnet — find live hosts ──
nmap -sn 192.168.x.0/24

# THE SCANS I ACTUALLY RAN (Zenmap profiles):
nmap -T4 -F 192.168.x.0/24        # Quick scan  -> 3 hosts up, 9.57s
nmap -T4 -A -v 192.168.x.0/24     # Intense scan (OS + version + scripts)

# quick port scan of one discovered host (bonus)
nmap -F 192.168.x.1

# save output to a file
nmap -sn 192.168.x.0/24 -oN live_hosts.txt
```

### Zenmap GUI steps

1. **Target:** `192.168.x.0/24`  *(use your own subnet from `ipconfig`)*
2. **Profile:** `Ping scan` (discovery only) or `Quick scan` (discovery + top-100 ports)
3. **Scan** → left pane lists live hosts → **Host Details** tab shows MAC/vendor
4. **Topology** tab → **Controls → Save Graphic** → **PDF** → save on Desktop

---

## 📄 Answer Template (copy into your submission)

```
Subnet scanned:     192.168.x.0/24          (masked for privacy)
My IP:              192.168.x.101           (Wi-Fi adapter)
Default gateway:    192.168.x.1             (router)

Live hosts found:   3

  IP              MAC (masked)         Role / open ports
  192.168.x.1     3C:67:C9:XX:XX:XX    Router - 53/tcp DNS, 80/tcp HTTP
  192.168.x.100   AA:77:28:XX:XX:XX    Firewalled device - all ports filtered
  192.168.x.101   (own adapter)        My PC - 135/139/445 SMB, 5357 WSD, ...

Topology PDF saved:  Desktop\network_topology.pdf
```

---

## 🧠 Concept Recap — One-Liner per Tool

| Tool | One-liner |
|------|-----------|
| `whois` | Who owns the domain, since when, which name servers |
| `whatweb` | What web server, CMS & frameworks the site runs |
| `nslookup` / `dig` | Which IP the domain points to |
| `curl -I` | What the HTTP headers reveal about the server |
| `wafwoof` / `wafw00f` | Is there a firewall in front of the site |
| `dnsrecon` | Every DNS record type of the domain |
| `nmap -sn` | Which hosts on the subnet answer ping |
| `arp -a` | IP ↔ MAC pairs my machine has seen |
| `ipconfig` | My own IP, subnet mask and gateway |
