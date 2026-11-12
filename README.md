# 🛡️ Week 2 Security Assessment Report

### 🔍 Footprinting and Local Network Discovery

**Networkwalks Internship | Prepared by Frank-Nnokwe Raymond Chigemezu**

| Item | Detail |
| ---- | ------ |
| **👤 Author** | Frank-Nnokwe Raymond Chigemezu |
| **📅 Date** | 12 Nov 2026 |
| **📦 Modules** | W2-PM1 (Footprinting) and W2-PM5 (Network scanning with Zenmap) |
| **🎯 Targets** | `networkwalks.com` (public information only) and my own home Wi-Fi network |
| **🔐 Scope** | Only the assigned domain and a network I own were examined. No exploitation was attempted. |
| **🧭 Phase reached** | Information gathering and host discovery |

---

## ⚖️ 1. Legal Notice

Everything here was done in a learning setting, against a target I was assigned and a network I personally own. This report is for education. Scanning or probing systems without permission is illegal, so always get authorisation first.

---

## 📖 2. Overview

This report covers two practical modules from Week 2 of the Networkwalks internship.

- **W2-PM1:** I footprinted `networkwalks.com` with six Kali Linux tools to see what is publicly visible about the domain.
- **W2-PM5:** I used Zenmap to find the live devices on my own Wi-Fi network.

The first module looks at a target from the outside. The second turns inward and maps a network I control. Together they show the first steps of an assessment: collect public information, then identify what is reachable.

**Environment:** The footprinting commands ran on Kali Linux in VirtualBox. The network scan ran on Windows, using Command Prompt (CMD) and Zenmap. Each activity below lists the command, what came back, the evidence screenshot, and why a defender should care.

---

## 🧰 3. Tools Used

| Tool | What I used it for |
| ---- | ------------------ |
| 🐧 Kali Linux (VirtualBox) | Main environment for footprinting |
| 📇 `whois` | Look up domain registration details |
| 🧩 `whatweb` | Identify the technologies a website runs |
| 🔎 `nslookup` | Resolve the domain to IP addresses |
| 📨 `curl -I` | Read the HTTP response headers |
| 🧱 `wafw00f` | Check whether a web application firewall sits in front |
| 🗂️ `dnsrecon` | Pull DNS records for the domain |
| 🗺️ Zenmap | Discover hosts on my local network |

---

## 🧪 4. Work Carried Out

## 🔍 4.1 W2-PM1: Footprinting `networkwalks.com`

I started broad and narrowed down: who owns the domain, what it runs on, where it points, how the server responds, whether a firewall protects it, and what its full DNS looks like.

### 4.1.1 📇 WHOIS: registration details

**Command:** `whois networkwalks.com`

**What I found:**

- **Registrar:** GoDaddy.com, LLC
- **Dates:** created 2019-11-06, last updated 2025-11-12, expires 2027-11-06
- **Name servers:** NS6135.HOSTGATOR.COM and NS6136.HOSTGATOR.COM, so the site is hosted at HostGator
- **DNSSEC:** unsigned
- **Status locks:** clientDeleteProhibited, clientRenewProhibited, clientTransferProhibited and clientUpdateProhibited (these protect against unauthorised transfer or changes)
- **Registrant details:** not shown (privacy-protected)

📸 *Evidence:*

![WHOIS output](https://github.com/user-attachments/assets/fac29550-f93f-4abe-960b-0b88574fd8a2)

🎯 *Why it matters:* Registration data can reveal who runs a domain and which providers it relies on.
🛡️ *Defender's takeaway:* Privacy protection and transfer locks limit what is exposed and guard against hijacking.

### 4.1.2 🧩 WhatWeb: website technologies

**Command:** `whatweb networkwalks.com`

**What I found:**

- **Web server:** Apache
- **CMS:** WordPress 7.1.4 with WordPress Download Manager 3.3.76 (from the MetaGenerator output)
- **Front-end:** jQuery 3.7.1, Bootstrap, HTML5, Google Tag Manager
- **IP address:** 192.232.216.135 (United States)
- **Redirect:** `http://networkwalks.com` returns 301 Moved Permanently to `https://`
- **Other details:** HttpOnly cookie `__wpdm_client`, contact email info@networkwalks.com, page title "Networkwalks Academy"

📸 *Evidence:*

![WhatWeb output](https://github.com/user-attachments/assets/2a87fdf8-1580-4622-a794-e71b25bb01a1)

🎯 *Why it matters:* Visible technology and version details help an attacker look for known weaknesses.
🛡️ *Defender's takeaway:* Keep the CMS and plugins updated and avoid advertising exact versions.

### 4.1.3 🔎 nslookup: name resolution

**Command:** `nslookup networkwalks.com`

**What I found:**

- **DNS server used:** 8.8.8.8 (Google), with a non-authoritative answer
- **Result:** `networkwalks.com` resolves to 192.232.216.135, which matches the IP found by WhatWeb

📸 *Evidence:*

![nslookup output](https://github.com/user-attachments/assets/4d3b3679-989f-434e-82ae-4a2ae8437611)

🎯 *Why it matters:* The IP address gives the network location of the web service and is the starting point for scanning.
🛡️ *Defender's takeaway:* The address is public by design, so protection has to come from the services behind it.

### 4.1.4 📨 curl: HTTP headers

**Command:** `curl -I https://networkwalks.com`

**What I found:**

- **Status:** HTTP/2 200, server: Apache
- **Caching headers:** `x-nginx-cache: WordPress`, `x-endurance-cache-level: 0`
- **Cookie:** `__wpdm_client` set with the Secure and HttpOnly flags (good practice)
- **Present headers:** `permissions-policy`, `referrer-policy: no-referrer-when-downgrade`
- **Exposed endpoint:** Link headers reveal the WordPress REST API at `/wp-json/`
- **Missing security headers:** Strict-Transport-Security (HSTS), X-Frame-Options, X-Content-Type-Options, Content-Security-Policy

📸 *Evidence:*

![curl -I output](https://github.com/user-attachments/assets/d68dbb24-5e7b-4fbe-93e9-f6caa40d150b)

🎯 *Why it matters:* The missing headers show where browser-side protection is weak, and the Link and Server headers confirm the technology stack.
🛡️ *Defender's takeaway:* Adding the missing headers is a low-effort improvement.

### 4.1.5 🧱 wafw00f: firewall detection

**Command:** `wafw00f https://networkwalks.com`

**What I found:**

- **Result:** the site is behind a ModSecurity (SpiderLabs) WAF, detected in 2 requests (wafw00f v2.4.2)
- **Note:** my first attempt timed out because of a slow connection; the second attempt succeeded

📸 *Evidence:*

![wafw00f output](https://github.com/user-attachments/assets/42d11dcc-5b8e-43bf-b2ee-8dd53787f764)

🎯 *Why it matters:* Knowing the WAF lets an attacker choose evasion techniques or look for ways around it.
🛡️ *Defender's takeaway:* A WAF is a good control, but it should not be the only layer.

### 4.1.6 🗂️ dnsrecon: DNS records

**Command:** `dnsrecon -d networkwalks.com`

**What I found:**

- **SOA / NS:** ns6135.hostgator.com (50.87.144.87) and ns6136.hostgator.com (192.232.216.131)
- **A record:** `networkwalks.com` → 192.232.216.135
- **MX record:** `mail.networkwalks.com` → 192.232.216.135 (web and mail share the same server)
- **SPF (TXT):** `v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all` (uses a softfail, `~all`)
- **TXT:** a Google site-verification record
- **SRV:** 8 `_autodiscover._tcp` records pointing to `cpanelemaildiscovery.cpanel.net`, which shows the host uses cPanel
- **DNSSEC:** no answer to the DNSSEC query, which matches the unsigned status in WHOIS
- **BIND version:** both name servers disclose version 9.16.23-RH

📸 *Evidence:*

![dnsrecon output](https://github.com/user-attachments/assets/512356a8-c3d5-4189-a959-4b1100f524dc)

🎯 *Why it matters:* The records map the infrastructure, the exposed software version helps targeting, and the softfail SPF policy makes spoofed emails easier.
🛡️ *Defender's takeaway:* Tighten the SPF policy and hide DNS software versions where possible.

---

## 🗺️ 4.2 W2-PM5: Finding Live Hosts on My Network with Zenmap

This part was done on my own Wi-Fi network only.

| Step | Action | My result |
| ---- | ------ | --------- |
| 1️⃣ | Installed or opened Zenmap | ✅ Zenmap opened and ready |
| 2️⃣ | Found my IP, subnet mask and gateway (`ipconfig`) | ✏️ IP: 10.248.50.54, mask: [255.255.255.0], gateway: [ 10.248.50.164] |
| 3️⃣ | Worked out the CIDR range to scan | 10.248.50.0/24 |
| 4️⃣ | Ran a ping scan across that range | `nmap -sn 10.248.50.54/24` |
| 5️⃣ | Counted the hosts that were up | 2 hosts |
| 6️⃣ | Listed the IP addresses found | 10.248.50.164 and 10.248.50.54 |
| 7️⃣ | Listed the MAC addresses and vendors | 6E:D8:5C:F3:8E:B6 |
| 8️⃣ | Exported the topology view | ✅ [`zenmap_topology.pdf`](screenshots/zenmap_topology.pdf) |

**Scan summary line from Zenmap:** "256 IP addresses (2 hosts up) scanned in 6.09 seconds"

**Observations:** ✏️ [Add one or two sentences: which of the two devices is yours and which is the other, why only one MAC address was listed, and whether the MAC looks randomised.]

📸 *Evidence:*

📎 **Export:** [`zenmap_topology.pdf`](screenshots/zenmap_topology.pdf)

✏️ *To add:* <img width="476" height="400" alt="zenmapscreenshot" src="https://github.com/user-attachments/assets/54ba53e3-3283-4529-b481-0182e412ee4b" />
.
---

## 🧯 4.3 Problems I Ran Into

| Problem | Cause | How I fixed it |
| ------- | ----- | -------------- |
| `nslookup` and `wafw00f` timed out | Unstable network connection | I switched to a stable Wi-Fi connection and repeated the commands, and they worked |
| `whatweb` did not work at first | I misspelled the command as `whatwed` | I corrected the spelling and ran it again |

---

## ⚠️ 5. Risk Assessment

The table below records what I noticed during testing, with a rough rating of how useful each item could be to an attacker.

| # | Finding | What I observed | Possible impact | Level |
| - | ------- | --------------- | --------------- | ----- |
| 1 | WordPress and plugin versions exposed | WhatWeb reported WordPress and the Download Manager plugin along with version numbers | Public lists of known flaws can be searched for those exact versions | 🟡 Medium |
| 2 | Important security headers missing | `curl -I` showed no HSTS, X-Frame-Options, X-Content-Type-Options or CSP | Browsers get less protection against clickjacking, content-type sniffing and script injection | 🟡 Medium |
| 3 | SPF record uses softfail | dnsrecon returned an SPF policy ending in `~all` | Spoofed emails that claim to come from the domain may still be accepted | 🟡 Medium |
| 4 | WordPress REST API advertised | Response headers include a link to `/wp-json/` | Makes the site easier to fingerprint and its content easier to enumerate | 🟢 Low |
| 5 | DNSSEC not enabled | WHOIS marked the domain as unsigned and dnsrecon found no DNSSEC records | DNS answers are not cryptographically verified | 🟢 Low |
| 6 | DNS server software version disclosed | The name servers reported BIND 9.16.23-RH | Attackers can look for flaws tied to that specific version | 🟢 Low |
| 7 | Hosting provider easy to identify | HostGator, cPanel and the server IP appeared in WHOIS and DNS results | Shows where and how the site is hosted | 🟢 Low |
| 8 | Firewall product identifiable | wafw00f detected ModSecurity | An attacker learns which filter to work around, although having a WAF is still a good control | 🟢 Low |
| 9 | Devices on my network easy to discover | The Zenmap ping scan listed every live device with its IP and MAC details (my result: 2 devices) | Any device on a network can be found this way, which also lets defenders spot unknown ones | 🟢 Low |

**Rating key:** 🔴 Critical  🟠 High  🟡 Medium  🟢 Low

> ℹ️ These are observations, not confirmed vulnerabilities. Nothing was exploited.

---

## ✅ 6. Recommendations

1. 🔎 **Review publicly exposed technology information:** regularly check what the web server, CMS and plugins reveal publicly, and keep them updated.
2. 🧾 **Add missing HTTP security headers:** enable HSTS, X-Frame-Options, X-Content-Type-Options and a Content-Security-Policy, and review which headers expose unnecessary technical detail.
3. ✉️ **Strengthen the SPF policy:** move from softfail (`~all`) to a hard fail (`-all`) once all legitimate senders are covered, and consider adding DKIM and DMARC.
4. 🙈 **Keep software and server versions out of public banners where possible.**
5. 🌐 **Review DNS records regularly** and remove anything unused.
6. 📶 **Secure home networks** with a strong Wi-Fi password and check for unknown devices.
7. ⚖️ **Run reconnaissance and scanning only where you have permission.**

---

## 🏁 7. Conclusion

Week 2 showed me how much can be learned about a system before anyone tries to attack it. Using six Kali Linux tools, I put together a profile of `networkwalks.com` without ever breaking in: who registered the domain, where it is hosted, which software it runs, whether a firewall sits in front of it, and how its DNS and email are configured. Each tool added one more piece, and none of the pieces looked serious alone, but together they gave a clear picture of the target.

The Zenmap exercise turned the same thinking inward. I scanned my own Wi-Fi network, found every live device, and exported the topology. It was surprising how quickly a small home network gave up its devices, and it made a strong case for defenders running the same scan on themselves so they can spot anything that doesn't belong.

I also learned that results only matter if they are recorded properly: the command, the output, what it means, the risk, and the fix. Finally, this work reminded me that tools like these belong only on systems I have permission to test. Along the way I ran into a few problems, which are listed in the problems section above, and fixing them taught me to read error messages carefully instead of guessing.

---

## 📸 8. Evidence Index

Each screenshot is shown under its activity in section 4. This table lists where to find them.

| # | Evidence | Tool | Shown in |
| - | -------- | ---- | -------- |
| 1 | WHOIS output | `whois` | 4.1.1 |
| 2 | WhatWeb output | `whatweb` | 4.1.2 |
| 3 | nslookup output | `nslookup` | 4.1.3 |
| 4 | HTTP headers | `curl -I` | 4.1.4 |
| 5 | Firewall detection | `wafw00f` | 4.1.5 |
| 6 | DNS records | `dnsrecon` | 4.1.6 |
| 7 | Topology export | Zenmap | 4.2 ([PDF](screenshots/zenmap_topology.pdf)) |

---

*End of report*
