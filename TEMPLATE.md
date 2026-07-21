# Room Name

| Info | |
|------|-|
| **Platform** | TryHackMe |
| **Difficulty** | Easy / Medium / Hard |
| **Tags** | `tag1` `tag2` `tag3` |
| **Date** | YYYY-MM-DD |
| **Room Link** | [TryHackMe](https://tryhackme.com/room/room-name) |

---

## Overview

Brief description of what the room covers.

---

## Recon

### Nmap Scan

```bash
nmap -sC -sV -oN nmap/initial <IP>
```

```
# paste results here
```

### Web Enumeration

```bash
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt
```

---

## Exploitation

Describe the vulnerability and how it was exploited.

```bash
# commands used
```

---

## Privilege Escalation

Describe the privesc vector.

```bash
# commands used
```

---

## Flags

| Flag | Status |
|------|--------|
| User | `obtained ✓` |
| Root | `obtained ✓` |

---

## Lessons Learned

- Key takeaway 1
- Key takeaway 2
