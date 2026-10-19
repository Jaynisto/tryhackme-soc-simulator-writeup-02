# TryHackMe: SOC Simulator - Introduction to Phishing (Lab Write-Up)

![SOC Operations](https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue)
![Platform](https://img.shields.io/badge/Platform-TryHackMe-red)
![Category](https://img.shields.io/badge/Category-Incident%20Response%20%26%20Phishing-brightgreen)

## 📌 Project Overview
This repository contains the walkthrough, investigative analysis, and incident triage reports for the **TryHackMe: SOC Simulator (Introduction to Phishing)** module.

As a Tier 1 SOC Analyst, the objective was to analyze incoming SIEM/Email/Firewall alerts, correlate logs across multiple data sources, identify True vs. False Positives, determine threat containment status, and execute remediation or escalation protocols.

---

## 🛠️ Tools & Data Sources Used
* **SIEM / Alert Management System:** Evaluated alert metadata (timestamps, vectors, rules).
* **Email Gateway Logs:** Inspected senders, lookalike domains, shorteners, and header information.
* **Firewall / Proxy Logs:** Investigated outbound traffic (`Action: allowed` vs. `Action: blocked`).
* **Threat Intelligence Feeds:** Verified domain/IP reputations (e.g., TryDetectThis lookup).

---

## 📁 Summary of Triaged Alerts

| Alert ID | Alert Type | Source / Target | Classification | Disposition |
| :--- | :--- | :--- | :--- | :--- |
| **Alert 1** | Firewall Blocked Web Traffic | `10.20.2.17` ➔ `bit.ly` shortener | True Positive | Closed (Contained) |
| **Alert 2** | Suspicious Inbound Link | `onboarding@hrconnex.thm` | False Positive | Closed (Benign Vendor) |
| **Alert 3** | Inbound Amazon Delivery Phish | `urgents@amazon.biz` | True Positive | Closed (Blocked by Firewall) |
| **Alert 4** | Microsoft Sign-In Alert Phish | `no-reply@m1crosoftsupport.co` | True Positive | **Escalated** (Connection Allowed) |

---

## 🔎 Detailed Case Investigations

### Case 1: Firewall Block - Malicious Bitly URL
* **Alert Trigger:** Outbound traffic to a blacklisted domain/shortener.
* **Source IP:** `10.20.2.17` | **Destination IP:** `67.199.248.11` (Bitly) | **Port:** `80` (TCP)
* **URL:** `http://bit.ly/3sHkX3da12340`
* **Analysis:** The user clicked a shortened link embedded in a phishing email. The firewall rule `Blocked Websites` successfully dropped the connection.
* **Verdict:** True Positive (Blocked)
* **Action Taken:** Resolved at Tier 1. No host containment needed since the download/connection was prevented.

---

### Case 2: HR Onboarding Email
* **Alert Trigger:** Inbound email containing external links.
* **Sender:** `onboarding@hrconnex.thm` | **Recipient:** `j.garcia@thetrydaily.thm`
* **Subject:** Action Required: Finalize Your Onboarding Profile
* **Analysis:** Cross-referenced SIEM internal communications. Verified that `hrconnex.thm` is an authorized third-party HR partner assisting with user onboarding.
* **Verdict:** False Positive
* **Action Taken:** Recommended domain tuning/whitelisting for HR Connex to prevent duplicate alert fatigue.

---

### Case 3: Amazon Delivery Phishing Email
* **Alert Trigger:** Suspicious inbound link detected in email gateway.
* **Sender:** `urgents@amazon.biz` | **Recipient:** `h.harris@thetrydaily.thm`
* **Subject:** Your Amazon Package Couldn’t Be Delivered – Action Required
* **URL:** `http://bit.ly/3sHkX3da12340`
* **Analysis:** Spoofed domain (`amazon.biz` vs `amazon.com`) with psychological urgency tactics. Correlated with Firewall logs showing user `h.harris` (`10.20.2.17`) clicked the link at `13:08:06`, but the connection was successfully dropped.
* **Verdict:** True Positive (Phishing / Contained)
* **Action Taken:** Purged email from target inbox. Added `amazon.biz` and the Bitly URL to corporate blocklists.

---

### Case 4: Microsoft Credential Harvesting Campaign
* **Alert Trigger:** Inbound email containing suspicious external link.
* **Sender:** `no-reply@m1crosoftsupport.co` | **Recipient:** `c.allen@thetrydaily.thm`
* **Target Host:** `10.20.2.25` | **Destination IP:** `45.148.10.131`
* **Subject:** Unusual Sign-In Activity on Your Microsoft Account
* **URL:** `https://m1crosoftsupport.co/login`
* **Analysis:**
  * **Typosquatting:** Sender domain uses `1` instead of `i` (`m1crosoftsupport.co`).
  * **Threat Intel:** IP `45.148.10.131` is flagged as malicious infrastructure.
  * **Log Correlation:** Firewall logs showed outbound connection from `10.20.2.25` was **ALLOWED**, indicating the user successfully navigated to the malicious login portal.
* **Verdict:** True Positive (Active Compromise Risk)
* **Disposition:** **ESCALATED TO TIER 2 / INCIDENT RESPONSE**
* **Remediation Plan:**
  1. Immediately reset Microsoft/AD credentials for `c.allen@thetrydaily.thm` and revoke active sessions.
  2. Isolate endpoint `10.20.2.25` via EDR to check for credential dumping or malware deployment.
  3. Block domain `m1crosoftsupport.co` and IP `45.148.10.131` enterprise-wide.

---
