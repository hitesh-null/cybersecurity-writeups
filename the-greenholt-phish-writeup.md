# TryHackMe — The Greenholt Phish Writeup

**Platform:** TryHackMe
**Room:** The Greenholt Phish
**Category:** Phishing Email Analysis / SOC / DFIR
**Difficulty:** Easy
**Author:** hitesh-null

---

## Introduction

In this room, we are given a suspicious email file named `challenge.eml` on the desktop of our virtual environment. Our goal is to investigate it like a real SOC analyst — analyzing email headers, tracing the originating IP, performing DNS record checks, and examining a suspicious attachment. This is a classic phishing email analysis task that sharpens your email forensics skills.

---

## Task Walkthrough

### Q1. What is the Transfer Reference Number listed in the email's Subject line?

Opening the `challenge.eml` file, we can directly read the subject line of the email. The transfer reference number is clearly mentioned there.

**Answer:** `09674321`

---

### Q2. What is the display name of the sender?

Looking at the email header section, the **From** field shows the display name associated with the sender's address.

**Answer:** `Mr. James Jackson`

---

### Q3. What is the sender's email address?

Checking the email headers more carefully, the actual sender's email address is visible at the top of the email in the header fields.

**Answer:** `info@mutawamarine.com`

---

### Q4. What email address will receive a reply to this email?

This is a classic phishing trick — the **Reply-To** address is different from the **From** address. When a victim hits "Reply," the response goes to a different, attacker-controlled address. Here the Reply-To field reveals the real destination.

**Answer:** `info.mutawamarine@mail.com`

> 💡 **Note:** Notice the subtle difference — `mutawamarine.com` vs `mail.com`. This is a classic social engineering technique where the attacker uses a lookalike domain to intercept replies without the victim noticing.

---

### Q5. What is the originating IP address of this email?

To find this, we navigate to **View → Message Source** inside the email client. The raw message source contains the full chain of `Received:` headers, which trace the path the email took across mail servers. The very first (bottom-most) `Received:` header shows the originating IP — the server that first injected the email into the internet.

**Answer:** `192.119.71.157`

---

### Q6. Who is the owner of the originating IP?

Using a **WHOIS lookup** on the IP address `192.119.71.157`, we can identify who owns that IP block and what organization it belongs to. WHOIS is a query tool that returns registration details for IP addresses and domains.

**Tool used:** [whois.domaintools.com](https://whois.domaintools.com) or `whois 192.119.71.157` in terminal

**Answer:** `Hostwinds LLC`

> 💡 **Note:** Hostwinds LLC is a web hosting company. Threat actors often abuse legitimate hosting services to send phishing emails, making the IP look less suspicious at first glance.

---

### Q7. What is the full SPF record for the Return-Path domain?

**SPF (Sender Policy Framework)** is an email authentication method that specifies which mail servers are allowed to send emails on behalf of a domain. By looking up the SPF record of the Return-Path domain from the email headers, we can verify whether the email was sent from an authorized server.

**Tool used:** [mxtoolbox.com/spf.aspx](https://mxtoolbox.com/spf.aspx) or similar SPF lookup tools

**Answer:** `v=spf1 include:spf.protection.outlook.com -all`

> 💡 **Note:** The `-all` at the end means all senders not listed should be rejected (hard fail). Despite this strict policy, the email still made it through — a red flag.

---

### Q8. What is the complete DMARC record for the Return-Path domain?

**DMARC (Domain-based Message Authentication, Reporting & Conformance)** is a policy that tells receiving mail servers what to do when SPF or DKIM checks fail. Looking up the DMARC record gives us insight into how aggressively the domain owner has configured their email security.

**Tool used:** [mxtoolbox.com/dmarc.aspx](https://mxtoolbox.com/dmarc.aspx)

**Answer:** `v=DMARC1; p=quarantine; fo=1`

> 💡 **Note:** `p=quarantine` means emails that fail DMARC checks should be sent to spam/quarantine — not rejected outright. This is a moderate policy. `fo=1` means forensic reports are sent when any authentication mechanism fails.

---

### Q9. What is the file name of the attachment found in the email?

Opening the email, we can see an attachment at the bottom of the message. The file name is clearly visible there.

**Answer:** `SWT_#09674321____PDF__.CAB`

> 💡 **Note:** The file name is crafted to look like a legitimate PDF related to the transfer reference number. The `.CAB` extension is a Windows Cabinet archive format — a common trick to disguise malicious files.

---

### Q10. What is the SHA256 hash of the attachment?

After downloading the attachment to our virtual environment, we use the `sha256sum` command in the terminal to generate its cryptographic hash. SHA256 hashes are used to uniquely identify files and cross-reference them with threat intelligence databases.

```bash
sha256sum SWT_#09674321____PDF__.CAB
```

**Answer:** `2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f`

---

### Q11. What is the attachment's file size in KB (as shown on VirusTotal)?

We take the SHA256 hash from the previous step and search it on [VirusTotal](https://www.virustotal.com). VirusTotal is a free online service that analyzes files and URLs using multiple antivirus engines and threat intelligence feeds. The **Details** tab on VirusTotal shows the file size.

**Answer:** `400.26 KB`

---

### Q12. What is the actual file type of the attachment?

Even though the file has a `.CAB` extension, VirusTotal's analysis reveals its true file type. Attackers often rename or disguise malicious archives with misleading extensions to trick both users and security filters.

**Answer:** `RAR`

> 💡 **Note:** The file masquerades as a `.CAB` (Cabinet Archive) but is actually a **RAR archive**. This is a common obfuscation technique used by malware distributors — the file extension does not reflect the actual content. Always verify file types using hash analysis and tools like VirusTotal or `file` command in Linux.

---

## Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Transfer Reference Number | `09674321` |
| 2 | Sender Display Name | `Mr. James Jackson` |
| 3 | Sender Email Address | `info@mutawamarine.com` |
| 4 | Reply-To Address | `info.mutawamarine@mail.com` |
| 5 | Originating IP | `192.119.71.157` |
| 6 | IP Owner | `Hostwinds LLC` |
| 7 | SPF Record | `v=spf1 include:spf.protection.outlook.com -all` |
| 8 | DMARC Record | `v=DMARC1; p=quarantine; fo=1` |
| 9 | Attachment File Name | `SWT_#09674321____PDF__.CAB` |
| 10 | SHA256 Hash | `2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f` |
| 11 | File Size on VirusTotal | `400.26 KB` |
| 12 | Actual File Type | `RAR` |

---

## Key Takeaways

- **Reply-To spoofing** is one of the simplest and most effective phishing tricks — always verify where your reply is actually going.
- **SPF, DKIM, and DMARC** are the three pillars of email authentication. Even with strict records, misconfigurations or abuse of legitimate hosting services can allow phishing emails to slip through.
- **File extension ≠ File type.** Always verify the real file type using hashing tools or the `file` command. Never trust what the extension says.
- **VirusTotal** is an essential tool in any SOC analyst's arsenal for rapid threat intelligence on suspicious files and URLs.
- **WHOIS lookups** on originating IPs can quickly reveal whether an email came from a suspicious or legitimate infrastructure.

---

*Written by [hitesh-null](https://github.com/hitesh-null) | Also posted on [Medium](https://medium.com/@hitesh-null)*
