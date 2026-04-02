---
title: FBI and CISA Warn of Russian Espionage Targeting Signal and WhatsApp
date: 2026-03-26 12:00:00 +0100
tags: [security]
categories: [Threats and Attacks]
author: [erica_malafronte, secai_team]
pin: false 
image:
  path: assets\img\posts\coverimages\RussianAttackCoverImage.jpg
---
On **March 20, 2026**, the Federal Bureau of Investigation (FBI) and the Cybersecurity and Infrastructure Security Agency (CISA) issued a joint Public Service Announcement, Alert Number **I-032026-PSA**, warning of a global espionage campaign orchestrated by the **Russian Intelligence Services (RIS)**. This operation specifically targets commercial messaging applications (CMAs), most notably Signal and WhatsApp, to gain unauthorized access to the communications of high-value individuals.

## **Targets and Global Reach**

The campaign focuses on **individuals of high intelligence value**, including current and **former U.S. government officials, military personnel, political figures, and journalists**.\
FBI Director Kash Patel noted that the effort has already resulted in the unauthorized access of thousands of individual accounts worldwide.

While the FBI's announcement is a primary source of attribution, previous reports from Microsoft and Google have linked similar activities to Russian-aligned threat groups such as **Star Blizzard** and **UNC5792**.

## **The Mechanism: Social Engineering over hacking**

A critical distinction of this attack is that the RIS actors **are not "breaking" the encryption of the applications themselves**.\
Instead, they use **social engineering to bypass security protections** by tricking users into granting access.\
Attackers typically masquerade as **automated support accounts**, using names like **"Signal Support"** or **"Signal Security Bot"** to send messages that create a false sense of urgency regarding "suspicious login attempts" or "data leaks".

## **Dual methods of account compromise**

The RIS actors employ two primary methodologies to infiltrate these accounts: 

➤ **Method 1: Linked Device Feature Abuse**

This technique is characterized by its stealth, often allowing attackers to monitor a victim's communications for extended periods without detection.

- **Identification of High-Value Targets:** The actors specifically identify individuals with access to sensitive information, including current and former U.S. government officials, military personnel, political figures, and journalists.

- **Impersonation and Initial Contact:** Attackers masquerade as trusted contacts or, more commonly, as automated support entities with names like "Signal Support," "Signal Security Bot," or "Signal Support Bot".

- **Delivery of Malicious Links or QR Codes:** The victim is sent a direct message urging them to take action—such as **"verifying" their account due to a fake data leak—by clicking a malicious link or scanning a QR code**.

- **Device Linking:** Once the victim interacts with the link or QR code, they unwittingly allow the attacker to link an adversary-controlled device to their existing CMA account.

**Access and Persistence:** Unlike a full takeover, the victim retains access to their account and may remain completely unaware of the compromise.

Meanwhile, the RIS actors gain access to the account, allowing them to silently monitor communications, view contact lists, and join private group chats.

On **WhatsApp**, this method can even grant the attacker access to past **message histories**.

➤ **Method 2: Account Takeover**

The second method is more overt and involves a complete hijack of the victim's digital identity on the messaging platform.

**Target Selection:** Similar to the linking scheme, the RIS identifies high-intelligence-value targets.

**Phishing for Credentials:** Actors send phishing messages designed to create a false sense of urgency, pressuring the victim to provide **their SMS verification code or account PIN**.

**Credential Submission:** Deceived by the "support" persona, the victim provides the PIN or 2FA code to the actor.

**Loss of Access:** The attacker immediately uses these credentials to register the victim’s phone number on a device under their control. Consequently, **the victim loses access to their CMA account and is effectively locked out**.

**Full Account Access:** The actors now have full access to the victim's account. While they cannot see past Signal messages (which are stored locally), they can monitor all fresh incoming messages and view the victim's contact lists.

## **Defense and Mitigation**

To protect against these threats, the FBI and CISA recommend rigorous cyber hygiene.\
Key steps include:

- **Never share verification codes**: Legitimate support for Signal or WhatsApp will never ask for verification codes or PINs via in-app messages.
- **Scrutinize "Support" messages**: Users should treat any unsolicited message from a "Support" account as suspicious by default.
-**Review Linked Devices**: Periodically check application settings to revoke any unrecognized linked devices.
-**Enable Advanced Security**: Features such as registration locks, PINs, and disappearing messages can limit the effectiveness of an account compromise.

Victims of these attacks are encouraged to report incidents immediately to the Internet Crime Complaint Center (IC3) at ic3.gov or their local FBI field office.

---

All rights reserved