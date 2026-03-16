# Cybersecurity Hands-On Lab — Module 1
### Understanding Basic Security Weaknesses

| Field | Details |
|---|---|
| **Student** | GUISSOU Ali |
| **Instructor** | Lebian Wilfried NIKIEMA, Ph.D |
| **Date** | March 16, 2026 |
| **Course** | Introduction to Cybersecurity |

---

## Table of Contents

1. [Lab Overview](#lab-overview)
2. [Part 1 — Password Security](#part-1--password-security)
3. [Part 2 — Network Traffic Capture](#part-2--network-traffic-capture)
4. [Part 3 — Packet Analysis](#part-3--packet-analysis)
5. [Analysis Questions](#analysis-questions)
6. [Conclusion & Security Lessons](#conclusion--security-lessons)

---

## Lab Overview

This laboratory session introduced fundamental cybersecurity concepts through hands-on activities using **Wireshark**, a network protocol analyzer. The lab covered password security principles and practical observation of live network traffic to understand why encryption and strong authentication mechanisms are essential.

---

## Part 1 — Password Security

### Step 1: Weak Password Analysis

The following passwords are considered extremely weak and dangerous:

| Password | Why It Is Insecure |
|---|---|
| `123456` | Purely numeric, sequential, and the most commonly used password worldwide |
| `password` | Dictionary word, trivially guessed by any brute-force tool |
| `admin` | Default credential for many systems — first thing attackers try |
| `12345678` | Slightly longer but still entirely numeric and sequential |
| `qwerty` | Keyboard pattern, well-known and included in every wordlist |

**Why these passwords are dangerous:**
Attackers use automated tools that can test millions of combinations per second using **dictionary attacks** (trying known common words and passwords) and **brute-force attacks** (systematically trying every possible combination). A password like `123456` can be cracked in under one second.

### Step 2: Strong Password Examples

Following the required rules (minimum 12 characters, uppercase + lowercase, numbers, special characters):

1. `G!u1ss0u_S3cur3`
2. `Cy83r#Lab@2026!`
3. `N3tw0rk$Pr0t3ct!`

These passwords are resistant to dictionary attacks due to their length, character variety, and absence of recognizable words or patterns.

---

## Part 2 — Network Traffic Capture

Wireshark was launched and traffic was captured on the active network interface for approximately two minutes while browsing several websites. The capture yielded **thousands of packets** across multiple protocols.

### Screenshot — DNS Traffic

The following capture shows DNS query and response traffic observed during the session:

![DNS Traffic Capture](Capture_d_écran_2026-03-16_181657.png)

**Observation:** The host `192.168.100.69` was actively sending DNS queries to the DNS resolver at `192.168.100.1`. Domain names resolved include `bit.bf`, `opentor.org`, `safebrowsing.google.com`, `www.google.com`, `www.bing.com`, and `dns.msftncsi.com`. Each query triggered a standard DNS response containing the resolved IP address.

### Screenshot — HTTP Traffic

The following capture shows unencrypted HTTP traffic:

![HTTP Traffic Capture](Capture_d_écran_2026-03-16_182213.png)

**Observation:** Multiple `GET /ncsi.txt HTTP/1.1` requests were sent from `192.168.100.69` to Microsoft connectivity check servers (`2.19.251.89` and `2.19.251.106`). These are Windows Network Connectivity Status Indicator (NCSI) probes sent automatically by the OS. Also visible are BitTorrent tracker announce requests (`GET /announce?info_hash=...`) sent over plain HTTP — a significant security concern as these expose user activity in cleartext.

### Screenshot — TLS/Encrypted Traffic

The following capture shows encrypted HTTPS/TLS traffic:

![TLS Traffic Capture](TLS.png)

**Observation:** Multiple TLS 1.3 and TLS 1.2 sessions were observed. Notably, several `Client Hello` packets were sent to `85.13.142.29` with **SNI=bit.bf**, initiating encrypted sessions with that server. Other encrypted sessions involved IP addresses associated with GitHub (`140.82.112.26`), Google, and Microsoft. QUIC protocol packets were also observed, used by modern HTTPS connections.

---

## Part 3 — Packet Analysis

### DNS Traffic

Using the `dns` filter in Wireshark, DNS queries and responses were clearly visible.

**Domain name observed:** `bit.bf`

This domain was queried multiple times (both HTTPS and A record types), and its IP address was resolved to `85.13.142.29`.

### HTTP Traffic

Using the `http` filter, unencrypted HTTP packets exposed:
- Full request URLs (e.g., `/ncsi.txt`, `/announce?info_hash=...`)
- HTTP method (`GET`)
- Server responses including status codes (`200 OK`, `403 Forbidden`, `404 Not Found`)
- Client and server IP addresses
- User activity indicators (BitTorrent tracker communication)

This clearly illustrates how unencrypted traffic reveals sensitive behavioral information.

### Encrypted Traffic (TLS)

Using the `tls` filter, TLS handshake and application data were visible. While the content of application data is unreadable (as expected from proper encryption), the **Server Name Indication (SNI)** field in `Client Hello` packets still leaks the target domain name in plaintext, even in TLS 1.3.

---

## Analysis Questions

### 1. Approximately how many packets were captured?

During approximately two minutes of browsing, Wireshark captured **over 44,000 packets** (as evidenced by packet numbers reaching `44,745+` in the DNS capture). This highlights how much network activity occurs passively in the background, even with minimal user interaction.

### 2. List three protocols observed during the experiment

| Protocol | Description |
|---|---|
| **DNS** | Domain Name System — translates domain names to IP addresses |
| **HTTP** | Hypertext Transfer Protocol — unencrypted web traffic |
| **TLS / HTTPS** | Transport Layer Security — encrypted web communications |

Additional protocols observed: **QUIC** (UDP-based transport used by HTTP/3) and **TCP**.

### 3. Why is encryption important when browsing the internet?

Encryption is essential for several reasons:

- **Confidentiality:** Without encryption, anyone on the same network (or any intermediate router) can read the full content of communications — including passwords, session tokens, personal data, and browsing history.
- **Integrity:** Encryption prevents man-in-the-middle attackers from silently modifying data in transit (e.g., injecting malicious scripts into web pages).
- **Authentication:** TLS certificates verify that the user is communicating with the legitimate server and not an impersonator.
- **Protection against passive surveillance:** Encrypted traffic cannot be easily indexed, profiled, or monetized by third parties.

As observed in this lab, even with TLS, the SNI field can reveal which domains are being visited. This is why technologies like **Encrypted Client Hello (ECH)** are being developed as a further privacy improvement.

### 4. What type of information could attackers obtain from unencrypted traffic?

From unencrypted (HTTP) traffic, an attacker performing passive capture could obtain:

- **Login credentials** (usernames and passwords submitted via HTTP forms)
- **Session cookies** (which can be used to hijack authenticated sessions)
- **Full URLs and query parameters** (revealing what pages are visited and what searches are made)
- **Application behavior** (e.g., BitTorrent tracker announces revealing file-sharing activity)
- **HTTP headers** (revealing browser type, operating system, referrer pages)
- **Personal data** (any information submitted in unencrypted forms)

In this lab specifically, the HTTP capture revealed BitTorrent tracker communication, exposing the user's peer ID and the info_hash of files being shared — information that could be used for legal or targeted attacks.

---

## Conclusion & Security Lessons

This laboratory session provided a concrete demonstration of network communication and the critical role of cybersecurity mechanisms.

**Key lessons learned:**

1. **Weak passwords are a primary attack vector.** Common passwords can be cracked in seconds using freely available tools. Strong, unique passwords are a minimal baseline of security.

2. **Network traffic is observable.** Every packet crossing a network can be captured and analyzed by anyone with physical or logical access to the network path — routers, ISPs, and local network participants alike.

3. **Unencrypted communications are inherently insecure.** HTTP traffic exposes URLs, headers, cookies, and payload data in full plaintext, making it unsuitable for any sensitive communication.

4. **Encryption (TLS/HTTPS) is essential but not perfect.** While TLS protects the content of communications, metadata such as IP addresses and SNI domain names may still be visible, and misconfigured or outdated TLS implementations can introduce vulnerabilities.

5. **Background activity is extensive.** Even without deliberate browsing, operating systems and applications continuously generate network traffic (NCSI probes, DNS pre-fetching, cloud sync, etc.), all of which can be observed and analyzed.

---

*Report submitted by **GUISSOU Ali** — Introduction to Cybersecurity, Module 1*
