# 🔥 Firewall, IDS & IPS Lab

A hands-on lab exploring network defense with three tools: **Windows Defender Firewall**, **Snort**, and **Fail2ban**.

- **Author:** Shree Vakthi 
- **Environment:** Windows 10/11 host, Ubuntu VM in VirtualBox

---

## 📌 Objectives

- Understand the difference between a firewall, an IDS, and an IPS.
- Write and test a rule or policy in each tool.
- Compare detection (alert) vs prevention (block).
- Document results with screenshots.

---

## 🧠 Concepts

- **Firewall:** Allows or blocks traffic using rules (IP, port, protocol).
- **IDS (Intrusion Detection System):** Monitors traffic and **alerts** only.
- **IPS (Intrusion Prevention System):** Detects threats and **blocks** them automatically.

| Feature | Firewall | IDS | IPS |
|---|---|---|---|
| Filters traffic | ✅ | ❌ | ✅ |
| Generates alerts | ⚠️ Limited | ✅ | ✅ |
| Blocks threats | ✅ | ❌ | ✅ |

---

## 🛠️ Tools Used

| Tool | Role | Purpose |
|---|---|---|
| Windows Defender Firewall | Host firewall | Block/allow ports on Windows |
| Snort | IDS | Rule-based traffic detection |
| Fail2ban | IPS | Auto-bans IPs after repeated failed SSH logins |

---

## 🧪 Lab 1: Windows Defender Firewall

**Goal:** Block inbound traffic on TCP port 8080.

1. Start a test web server on Windows:
```
   python -m http.server 8080
```
2. Open Windows Defender Firewall with Advanced Security:
```
   wf.msc
```
3. Go to **Inbound Rules → New Rule → Port → TCP → 8080 → Block the connection**.
4. Test from the Ubuntu VM (should fail):
```
   curl http://<windows-ip>:8080
```
5. Disable the rule and test again (should work).

**Result:** 

![Account creation alert](screenshots/account-created.png)
![Account creation alert](screenshots/account-created.png)
![Account creation alert](screenshots/account-created.png)

---

## 🧪 Lab 2: Snort (IDS)

**Goal:** Detect traffic with a custom Snort rule.

1. Install Snort on Ubuntu:
```
   sudo apt install snort
```
2. Add a rule to `/etc/snort/rules/local.rules`:
```
   alert tcp any any -> any 8000 (msg:"HTTP test"; sid:1000001; rev:1;)
```
3. Run Snort in IDS mode on the loopback interface:
```
   sudo snort -A console -q -c /etc/snort/snort.conf -i lo
```
4. In a second terminal, start a test server:
```
   python3 -m http.server 8000
```
5. In a third terminal, generate traffic:
```
   curl http://127.0.0.1:8000
```
6. Confirm the alert appears in the Snort console.

**Bonus:** write a rule to detect an Nmap SYN scan.

**Result:** _Add what happened here._

📸 `screenshots/snort-alert.png`

---

## 🧪 Lab 3: Fail2ban (IPS)

**Goal:** Automatically block an IP after repeated failed SSH logins.

1. Install Fail2ban and OpenSSH:
```
   sudo apt install fail2ban openssh-server -y
```
2. Create `/etc/fail2ban/jail.local`:
```
   [sshd]
   enabled = true
   maxretry = 3
   bantime = 600
```
3. Restart the service:
```
   sudo systemctl restart fail2ban
```
4. Fail 3 SSH logins on purpose:
```
   ssh wronguser@127.0.0.1
```
5. Check the jail status:
```
   sudo fail2ban-client status sshd
```
6. Check the log for ban events:
```
   sudo tail -n 20 /var/log/fail2ban.log
```

**Result:** Fail2ban detected 16 failed SSH attempts and banned 5 offenders. Bans expire after 600 seconds, so "Currently banned" returned to 0.

📸 `screenshots/fail2ban-status.png`
📸 `screenshots/fail2ban-log.png`

---

## 📊 Comparison Summary

| Tool | Detects | Blocks | Ease of Setup | Notes |
|---|---|---|---|---|
| Windows Defender Firewall | ❌ | ✅ | ⭐⭐⭐⭐⭐ | Built-in, host-level |
| Snort | ✅ | ❌ (IDS mode) | ⭐⭐⭐ | Rule-based alerts |
| Fail2ban | ✅ (log-based) | ✅ | ⭐⭐⭐⭐ | Bans IPs via firewall rules |

---

## ✅ Key Takeaways

- A firewall controls **what is allowed**, an IDS tells you **what happened**, and an IPS **stops it**.
- Fail2ban shows IPS behavior: detect, then block automatically.
- Layering a host firewall with IDS/IPS gives defense in depth.

---

## 🚧 Challenges Faced

- Suricata packet capture did not produce alerts in the VM, so it was replaced with Fail2ban for the IPS part.
- Ping between Windows and the VM failed (bridged networking), so Snort was tested on the loopback interface using TCP traffic instead of ICMP.
- _Add your own._

---

## 🔭 Next Steps

- Fix VirtualBox networking (Host-only adapter) for two-machine testing.
- Retry Suricata in IDS and inline IPS mode.
- Write a Snort rule to detect Nmap scans.
- Add a network firewall (pfSense) to the lab.

---

## 📁 Repository Structure

```
firewall-ids-ips-lab/
├── README.md
├── rules/
│   ├── snort-local.rules
│   └── jail.local
└── screenshots/
    ├── defender-rule.png
    ├── defender-blocked.png
    ├── defender-allowed.png
    ├── snort-alert.png
    ├── fail2ban-status.png
    └── fail2ban-log.png
```

---

## 📚 References

- [Snort Documentation](https://www.snort.org/documents)
- [Fail2ban Documentation](https://fail2ban.readthedocs.io)
- [Microsoft: Windows Defender Firewall](https://learn.microsoft.com/windows/security/operating-system-security/network-security/windows-firewall/)

---

## ⚠️ Disclaimer

This lab was done in a controlled, personal environment for educational purposes only. Do not test against networks you do not own or have permission to test.
