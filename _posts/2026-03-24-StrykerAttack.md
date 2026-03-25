---
title: Stryker Attack 
date: 2026-03-24 12:00:00 +0100
categories: [Meta]
tags: [security]
author: [erica_malafronte, secai_team]
pin: false 
image:
  path: /assets/img/posts/coverimages/StrykerAttackCoverImage.jpg
---

CISA is urging organizations to secure Microsoft Intune environments after attackers used it to wipe nearly 80,000 devices in the Stryker cyberattack.

But what happened?
## The cloud as a weapon: Handala’s attack and the strategic sabotage of Microsoft Intune

On March 11, 2026, the American medical technology giant Stryker Corporation became the target of one of the largest and most destructive cyber sabotage operations ever recorded.

The attack, claimed by the Iranian group Handala (also known as Void Manticore or Storm-842), caused a global operational paralysis,  affecting critical infrastructures in 79 countries.

![strikerattack](/assets/img/posts/stryker-attack/strikerattack.png)

**Geopolitical context and retaliation**

The Handala Hack group (also known as Void Manticore or Storm-0842), an actor linked to the Iranian Ministry of Intelligence and Security (MOIS), stated that the operation was a direct retaliation for a U.S. missile strike on February 28, 2026, against a girls’ school in Minab, Iran, which allegedly caused 175 casualties.

Stryker Corporation was targeted not only for its economic significance ($25 billion in sales), but also for its $450 million contracts with the U.S. Department of Defense and its ties to Israel.

**Technical analysis: how Microsoft Intune works**

To understand the scope of the sabotage, it is necessary to analyze how Microsoft Intune works. It is a cloud-based Mobile Device Management (MDM) service that acts as a configuration delivery center for IT departments.

Through BYOD (Bring Your Own Device) policies, companies require employees to enroll their personal devices to access corporate resources such as email or Teams.
This grants the IT department administrative oversight of the device.
One of Intune’s standard features is “Wipe”: an emergency switch designed to restore a lost or stolen device to factory settings.

**Attack chain**

The operation did not exploit any zero-day vulnerabilities but instead targeted the weakest point: digital identity.

- **Wiper Execution**: Instead of deploying a virus, the hackers simply selected all registered devices and activated the legitimate “Wipe” command simultaneously.
- **Operating System Response**: When an iPhone or Android device receives a wipe command from its trusted MDM server, the operating system immediately and unconditionally obeys, initiating a complete factory reset.

As a result of this action, the phones removed not only corporate applications, but also performed a complete operating system reset, deleting personal photos, banking apps, and even cellular eSIMs, leaving many users unable to use two-factor authentication (2FA) for their personal accounts.

![attackchain][def]

Sources indicate that **more than 200,000 systems, including servers and mobile devices across 79 countries**, were affected by this attack.

To maximize the damage, Handala simultaneously employed several destructive methods:

- **Offline Systems and GPO**: To target computers that were offline at the time of the initial command, the attackers prepared malicious scripts distributed as legitimate Group Policy Objects (GPOs) from the Domain Controller. When these systems reconnected to the network, the scripts automatically executed the wiper.
- **PowerShell and Artificial Intelligence**: An additional PowerShell-based wiper, likely developed with AI assistance, was deployed. It was capable of enumerating and recursively deleting all user directories.
- **VeraCrypt Encryption and RDP**: For the systems that survived, the attackers used VeraCrypt to encrypt drives and leveraged Remote Desktop Protocol (RDP) access to manually delete virtual machines (VMs) from virtualization platforms.

**Global impact and operational consequences**

The impact was catastrophic:

- **Data Loss:** Handala claimed to have erased data from over 200,000 systems, including servers and mobile devices, and to have exfiltrated 50 terabytes of critical data. The group also threatened to publicly release the data, stating that the goal is to expose what it describes as “injustice and corruption.”\
- **Work Paralysis:** In Ireland, more than 5,000 employees were sent home due to total network inaccessibility.
The login pages of the remaining active devices were defaced with the Handala logo.\
- **Economic Impact:** Stryker Corporation’s stock dropped by approximately 3.6% on the same day.\
- **Disruption of Medical Services:** The attack disabled the LifeNet platform in several regions, including Maryland, where the system went offline across much of the territory. LifeNet is used to transmit real-time electrocardiograms (ECGs) from paramedics to hospitals.
Emergency medical personnel were instructed to resort to radio consultations, verbally describing ECG results to the receiving doctors.

---

The Handala group’s attack against Stryker Corporation highlighted how the **cloud management plane** has become the **new critical perimeter of corporate security**.

In a modern infrastructure, tools like **Microsoft Intune** centralize control over **hundreds of thousands of devices** into a single dashboard to ensure operational efficiency. However, this same centralization **turns the portal into a single point of catastrophic failure**.

If an attacker obtains administrative credentials, they gain access to the **“main remote control”** of the entire organization, capable of causing **massive damage without needing to breach each endpoint individually**.

The operation conducted by Handala/Void Manticore demonstrates that in **modern cyber warfare**, saboteurs no longer need to develop complex malware or zero-day exploits.
It is sufficient to **abuse the trust placed in legitimate management suites**: if a company uses software to manage and protect its assets, an attacker with the right credentials can **use that same software to destroy them**.

For this reason, it becomes essential to implement **adequate security measures and governance policies** to protect centralized management systems.

Key preventive practices include enforcing **multi-factor authentication (MFA)** for all privileged accounts, applying the **principle of least privilege, segmentation of administrative roles**, continuous monitoring and logging of administrative activities, periodic rotation of sensitive credentials, and constant **verification of security configurations**.

Identity is no longer just an access control mechanism; it has become the **true new perimeter of national and corporate defense**. Protecting privileged accounts and management systems effectively means **protecting the entire infrastructure**.

A **strong and proactive security governance** is therefore a necessary condition to prevent tools designed to defend the organization from becoming, in the wrong hands, **the most effective weapons against it**.

---

All rights reserved

[def]: assets\img\posts\stryker-attack\strykerattackchain.jpg