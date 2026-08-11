# 📘 hMailServer — Technical Configuration & Troubleshooting Write‑Up

This document provides a full technical breakdown of how I installed, configured, tested, and troubleshot hMailServer inside my Active Directory home lab.  
It complements the screenshot documentation stored in `/screenshots`.

---

## 1. Environment Overview

**Server:**  
- Windows Server (SRV‑DC01)  
- Active Directory Domain Services  
- DNS Server  
- hMailServer installed  
- IP: 192.168.56.10  
- Domain: mydomain.local  

**Client:**  
- Windows 11 workstation  
- Thunderbird installed  
- IP: 192.168.56.104  
- Joined to domain  

**Network:**  
- VirtualBox Host‑Only: 192.168.56.0/24  
- NAT adapter for internet  
- DNS handled by SRV‑DC01  

---

## 2. Objectives

- Deploy a functional internal mail server  
- Configure SMTP and IMAP  
- Create internal mailboxes  
- Connect Thunderbird to hMailServer  
- Troubleshoot DNS, firewall, and authentication issues  
- Validate successful mail delivery  

---

## 3. Server Setup

### 3.1 Install hMailServer
Installed hMailServer on SRV‑DC01 and configured the administrative password.

### 3.2 Create Mail Domain
Domain created: **mydomain.local**

### 3.3 Create Mailboxes
- iskutashi@mydomain.local  
- siyaan@mydomain.local  

### 3.4 Configure Protocol Bindings
Set all bindings to `0.0.0.0`:

- SMTP  
- IMAP  
- POP3  

This ensures the workstation can reach the server.

### 3.5 Configure IP Ranges
Allowed internal network (192.168.56.0/24) to use SMTP and IMAP.

### 3.6 Enable Logging
Enabled:
- SMTP logging  
- Application logging  

Used for troubleshooting authentication and connection issues.

---

## 4. DNS Configuration

### 4.1 Hostname Resolution
Created A record:
```
SRV-DC01.mydomain.local → 192.168.56.10
```

Thunderbird uses this hostname for IMAP and SMTP.

### 4.2 Verification
Used:
```
nslookup SRV-DC01.mydomain.local
```

Confirmed correct DNS resolution.

---

## 5. Firewall Configuration

Opened required ports:

- **25** → SMTP  
- **143** → IMAP  

Verified using:

```
Test-NetConnection SRV-DC01.mydomain.local -Port 25
Test-NetConnection SRV-DC01.mydomain.local -Port 143
```

---

## 6. Thunderbird Configuration

Incoming (IMAP):  
- Server: SRV-DC01.mydomain.local  
- Port: 143  
- Security: None  
- Authentication: Password transmitted insecurely  

Outgoing (SMTP):  
- Server: SRV-DC01.mydomain.local  
- Port: 25  
- Authentication: Normal password  

Username: full email address  
Example: `iskutashi@mydomain.local`

---

## 7. Troubleshooting  
### **Format: Problem → Investigation → Root Cause → Resolution → Verification**

---

### Issue 1 — Server and Client on Different Subnets
**Problem:** No connectivity  
**Investigation:** Ping, ipconfig  
**Root Cause:** Server was on 192.168.1.x  
**Resolution:** Moved server to Host‑Only network  
**Verification:** Ping and DNS succeeded  

---

### Issue 2 — Wrong Domain Name
**Problem:** Thunderbird couldn’t resolve hostname  
**Investigation:** nslookup  
**Root Cause:** Used lab.local instead of mydomain.local  
**Resolution:** Corrected FQDN  
**Verification:** nslookup successful  

---

### Issue 3 — SMTP Port Blocked
**Problem:** SMTP unreachable  
**Investigation:** Test-NetConnection  
**Root Cause:** Firewall blocking port 25  
**Resolution:** Opened port 25  
**Verification:** SMTP reachable  

---

### Issue 4 — Auto-Ban Triggered
**Problem:** Authentication refused  
**Investigation:** Checked hMailServer auto-ban list  
**Root Cause:** Too many failed logins  
**Resolution:** Removed auto-ban entries  
**Verification:** Thunderbird authenticated  

---

### Issue 5 — Wrong SMTP Binding
**Problem:** Thunderbird couldn’t reach SMTP  
**Investigation:** Checked hMailServer bindings  
**Root Cause:** Bound to 127.0.0.1  
**Resolution:** Changed to 0.0.0.0  
**Verification:** Thunderbird connected  

---

## 8. Validation Tests

### 8.1 Connectivity Tests
```
Test-NetConnection SRV-DC01.mydomain.local -Port 25
Test-NetConnection SRV-DC01.mydomain.local -Port 143
```

### 8.2 Email Delivery Test
Sent email from:
- iskutashi → siyaan  
Received successfully.

### 8.3 IMAP Folder Sync
Thunderbird synced inbox and sent items without errors.

---

## 9. Lessons Learned

- DNS and subnet alignment are critical  
- SMTP requires explicit firewall rules  
- hMailServer auto-ban can silently block clients  
- Thunderbird requires full email address authentication  
- Internal mail systems teach real troubleshooting skills  

---

## 10. Skills Demonstrated

- Windows Server administration  
- DNS troubleshooting  
- SMTP/IMAP configuration  
- Firewall management  
- Client configuration (Thunderbird)  
- Enterprise-style documentation  
- Problem → Investigation → Root Cause → Resolution → Verification  
- AD-integrated mail system setup  

