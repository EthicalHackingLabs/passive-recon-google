# 🔍 Passive Reconnaissance on google.com — ASN, Shodan, WHOIS, DNS & SSL

A complete passive reconnaissance walkthrough targeting google.com using five techniques: Autonomous System lookup, Shodan searches, WHOIS queries, DNS enumeration (Sublist3r, DNS Dumpster, Fierce), and SSL certificate inspection.

All intelligence gathered from public data sources — no packets sent to the target, no logs triggered, no exploits used.

---

## 📌 What This Covers

- Finding a target's ASN and CIDR range using HackerTarget
- Shodan searches: hostname, CIDR subnet, product/version fingerprinting, SSL certs, NTP
- WHOIS domain registration analysis via Kali Linux
- DNS subdomain enumeration with Sublist3r, DNS Dumpster, and Fierce
- SSL certificate inspection and SAN list extraction via SSLLabs

---

## 🛠 Tools Used

| Tool | Purpose |
|---|---|
| HackerTarget | ASN and IP range lookup |
| Shodan | Internet-wide host discovery and fingerprinting |
| whois (Kali Linux) | Domain registration and name server data |
| Sublist3r | Multi-source subdomain enumeration |
| DNS Dumpster | Visual domain map and DNS record extraction |
| Fierce | DNS scanner for subdomains and nearby IP space |
| SSLLabs | SSL certificate inspection and SAN list extraction |

---

## 🌐 Target

| Field | Value |
|---|---|
| Target Domain | google.com |
| Resolved IP | 142.250.202.110 |
| ASN | AS15169 |
| CIDR Range | 142.250.0.0/15 |
| Organization | Google LLC |

---

## Task 1 — Autonomous System (AS) Lookup

![HackerTarget ASN lookup showing AS15169 for google.com](https://dev-to-uploads.s3.us-east-2.amazonaws.com/uploads/articles/kv6rzuj0vy4ti31awa7x.png)

```bash
ping google.com
# Returns: 142.250.202.110
# Feed that IP into HackerTarget ASN Lookup
```

| Field | Value |
|---|---|
| ASN | AS15169 |
| Organization | Google LLC |
| CIDR | 142.250.0.0/15 |

Google's AS15169 covers 130,000+ IP addresses. Every subsequent search was scoped to this range.

---

## Task 2 — Shodan Searches

### Search 1 — Hostname Filter

```
hostname:google.com
```

![Shodan hostname:google.com — 37,734 results](https://dev-to-uploads.s3.us-east-2.amazonaws.com/uploads/articles/nh80zwhj6zdvxi5x511k.png)

**37,734 hosts** — Top countries: US (12,722), UK (1,985), Canada (1,700). Top ports: 443, 80, 25.

---

### Search 2 — CIDR Subnet Filter

```
net:142.250.0.0/15
```

![Shodan net filter — 52,636 results](https://dev-to-uploads.s3.us-east-2.amazonaws.com/uploads/articles/hy5qk334v1qdeewg1ap4.png)

**52,636 hosts** within Google's IP range — broader than hostname filter, catches non-google.com-labeled hosts on Google's own address space.

---

### Search 3 — Microsoft IIS Within Google's Range

```
net:142.250.0.0/15 product:"Microsoft IIS httpd"
```

![Shodan IIS filter — 876 results](https://dev-to-uploads.s3.us-east-2.amazonaws.com/uploads/articles/i27xavasbu1g0fwdsewv.png)

**876 hosts** running Microsoft IIS inside Google's CIDR — likely from acquired companies absorbed into Google's address space.

---

### Search 4 — IIS Version 10.0

```
net:142.250.0.0/15 product:"Microsoft IIS httpd" version:"10.0"
```

![Shodan IIS version 10.0 — 869 results](https://dev-to-uploads.s3.us-east-2.amazonaws.com/uploads/articles/lg91h2d1e41n73213gye.png)

**869 of 876 hosts** (99%) on IIS 10.0 — consistent with managed Windows Server 2016/2019 infrastructure.

---

### Search 5 — SSL Certificates on IIS 10.0 Hosts

```
hostname:google.com product:"Microsoft IIS httpd" version:"10.0" port:443
```

![Shodan SSL cert results — 4 hosts](https://dev-to-uploads.s3.us-east-2.amazonaws.com/uploads/articles/85snstr30bq9ncmslkic.png)

**4 hosts** — SSL common names revealed third-party domains cached in Shodan's certificate index.

---

### NTP Server Search

```
org:"Google" port:123
```

![Shodan NTP server — org:Google port:123](https://dev-to-uploads.s3.us-east-2.amazonaws.com/uploads/articles/datevhb8aequ47n5c9n9.png)

**NTP Server Found: `34.102.13.166`** — 7,834 results total. Direct CIDR+port:123 search returned nothing on free tier; org-name search worked instead.

---

## Task 3 — WHOIS

```bash
whois google.com
```

![whois google.com terminal output — part 1](https://dev-to-uploads.s3.us-east-2.amazonaws.com/uploads/articles/zal8e69dd40vffkj32mp.png)

![whois google.com terminal output — part 2](https://dev-to-uploads.s3.us-east-2.amazonaws.com/uploads/articles/3gbqslugyk15ekljq13k.png)

| Field | Value |
|---|---|
| Registrar | MarkMonitor Inc. |
| Created | 1997-09-15 |
| Expires | 2028-09-14 |
| Registrant Country | US |
| DNSSEC | unsigned |
| Technical Contact | Privacy-protected via MarkMonitor |

**Authoritative Name Servers:**
```
NS1.GOOGLE.COM
NS2.GOOGLE.COM
NS3.GOOGLE.COM
NS4.GOOGLE.COM
```

Google self-manages DNS — no third-party DNS provider involved.

---

## Task 4 — DNS Enumeration

### Sublist3r

```bash
sudo apt install sublist3r
wget https://raw.githubusercontent.com/aboul3la/Sublist3r/3cb826c2f36f4972dfd286c704efc07de3a7f94c/sublist3r.py
sudo mv sublist3r.py /usr/lib/python3/dist-packages/sublist3r.py
export VT_APIKEY="your-virustotal-api-key"
sublist3r -d google.com -e google,yahoo,bing,virustotal,baidu,ask,ssl
```

![Sublist3r output — 2610 unique subdomains](https://dev-to-uploads.s3.us-east-2.amazonaws.com/uploads/articles/nkx7bclxkwqjc2gb13es.png)

**2,610 unique subdomains found.**

**5 interesting subdomains discovered:**

| Subdomain | Why It Stands Out |
|---|---|
| `actionscenter.google.com` | Admin/developer portal — not publicly documented |
| `actualities.google.com` | Internal news or content endpoint |
| `admanager.google.com` | Google Ad Manager platform |
| `admin.google.com` | Google Workspace admin console — always high-value |
| `admob.google.com` | Mobile advertising platform |

---

### DNS Dumpster — Domain Map

![DNS Dumpster domain map for google.com](https://dev-to-uploads.s3.us-east-2.amazonaws.com/uploads/articles/06v20trysth8qv8eriop.png)

Visual map of google.com's DNS infrastructure — MX records, name servers, host IPs, and connections all in one view.

---

### Fierce

```bash
fierce --domain google.com
```

![Fierce scan output for google.com](https://dev-to-uploads.s3.us-east-2.amazonaws.com/uploads/articles/mfg331wyhbhbkyapoeff.png)

```
NS: ns2.google.com. ns3.google.com. ns4.google.com. ns1.google.com.
SOA: ns1.google.com. (216.239.32.10)
Wildcard: failure
```

**Wildcard: failure** = every resolving subdomain is a real deployed host.

| Host | IP Address |
|---|---|
| 1.google.com | 142.250.202.78 |
| about.google.com | 142.250.200.174 |

---

## Task 5 — SSL Certificate Inspection

```
https://www.ssllabs.com/ssltest/analyze.html?d=google.com
```

![SSLLabs SSL report for google.com — Grade B](https://dev-to-uploads.s3.us-east-2.amazonaws.com/uploads/articles/e2qrnzn5ubsxv8ng1mux.png)

| Field | Value |
|---|---|
| Grade | B |
| Certificate Type | EC 256 bits (SHA256withECDSA) |
| Issued by | WE2 → Google Trust Services LLC |
| Valid from | Mon, 23 Feb 2026 |

![SSL certificate alternative names / SAN list](https://dev-to-uploads.s3.us-east-2.amazonaws.com/uploads/articles/ugozlzx37u2m7tvd1303.png)

**Selected Alternative Names (DNS SANs):**
```
google.com / *.google.com / *.android.com
*.appengine.google.com / *.bdn.dev / *.cloud.google.com
*.googleapis.cn / *.googlevideo.com / *.gstatic.cn
*.recaptcha.net / *.widevine.com / *.ampproject.org
*.google-analytics.com / *.doubleclick.net / *.doubleclick.cn
*.googlesyndication.com / *.gvt1.com / *.youtube-nocookie.com
youtube.com / *.youtube.com / youtu.be / youtubeeducation.com
android.com / aistudio.google.com / developers.google.cn
```

One certificate covering Google's entire product ecosystem — YouTube, Android, DoubleClick, reCAPTCHA, Widevine, and more.

---

## 🗺 The Full Reconnaissance Picture

| Recon Layer | Finding |
|---|---|
| Network | ASN AS15169, CIDR 142.250.0.0/15, 52,636+ hosts |
| Server fingerprinting | 876 hosts running Microsoft IIS 10.0 in Google's range |
| Registration | Registered 1997, MarkMonitor, 4 self-managed name servers |
| DNS surface | 2,610 unique subdomains |
| Certificate surface | 50+ domains and wildcards on one certificate |
| Time infrastructure | NTP at 34.102.13.166 |

---

## ⚠️ Common Mistakes in Passive Recon

| Mistake | What to Do Instead |
|---|---|
| Stopping at the first IP | Look up the full ASN and CIDR range |
| Ignoring IIS on non-Microsoft orgs | Acquired infra leaves unexpected fingerprints |
| Treating WHOIS privacy as total | Name servers, dates, and country are usually still visible |
| Using only one subdomain tool | Sublist3r, DNS Dumpster, and Fierce return different results |
| Skipping SSL inspection | SAN lists map the full certificate scope |
| CIDR-only NTP search in Shodan | Try org-name search when CIDR+port returns nothing |

---

## 📖 Full Write-Up

The complete step-by-step article with analysis and tool output is on Dev.to:

👉 [Passive Reconnaissance on google.com — What I Found Using Shodan, WHOIS, and DNS Tools](https://dev.to/almahmudkhalif/passive-reconnaissance-on-googlecom-what-i-found-using-shodan-whois-and-dns-tools-5hea)

---

## 🌐 Connect With Me

- **Dev.to** → [dev.to/almahmudkhalif](https://dev.to/almahmudkhalif)
- **LinkedIn** → [linkedin.com/in/almahmudkhalif](https://linkedin.com/in/almahmudkhalif)
- **GitHub** → [github.com/almahmudkhalif](https://github.com/almahmudkhalif)
