---
title: "Data Poisoning in Digital Forensics: a growing threat to evidences’s integrity"
date: 2026-03-24 12:00:00 +0100
tags: [articial intelligence, security]    
author: [erica_malafronte, david_cardoner, secai_team]          
pin: false                           
image:
  path: /assets/img/posts/coverimages/DataPoisoningCoverImage.png
---

**Digital forensics** plays a crucial role in reconstructing digital events, attributing responsibility, and collecting legally admissible evidence. The entire value of a digital forensic investigation depends on the integrity of the collected data: if this data is manipulated, the reliability of the results can be severely compromised. In this context, data poisoning emerges as a significant threat, especially for investigations that rely on artificial intelligence (AI) and machine learning (ML) models.

## **What is Data Poisoning?**

Data poisoning is a **type of cyber attack** in which attackers **manipulate or corrupt the training data used to develop machine learning (ML) or artificial intelligence (AI) models**.
This is particularly dangerous for forensic investigations that rely on machine learning to analyze malware signatures, perform intrusion detection, or correlate events across multiple data sources.
Once the training data is compromised, the model may make incorrect decisions or even be driven by malicious intent.

## **The importance of Data Poisoning in Digital Forensics**

In digital forensics, various automated tools are used to analyze large volumes of data, such as logs, network traffic, or file samples, to identify malicious behavior and attribute responsibility.\
If these tools are compromised through data poisoning, a forensic investigator may encounter false positives or fail to detect real threats.\
Even worse, if these manipulations remain undetected, legal proceedings can be undermined, and criminals may escape justice.

Consider a scenario where a forensic tool used by law enforcement is trained on a dataset of known malicious programs (malware) and benign software.\
If an attacker poisons this dataset by systematically **altering labels** or **injecting malicious** code disguised as benign, the model will suffer degradation that may lead it to generate incorrect classifications.

Investigators relying on this compromised model could inadvertently allow harmful artifacts to pass through. This highlights why data integrity and source reliability are fundamental in any digital investigation.

## **Types of Data Poisoning**

We can distinguish two main categories of data poisoning: **non-targeted and targeted**.

- **Non-targeted data poisoning**: in this type of attack, the attacker's objective is more general and it is to degrade the overall accuracy or reliability of the model. This has a broad and immediate impact, causing a noticeable performance drop. Examples include model inversion attacks (where attackers exploit model outputs to uncover sensitive training data), stealth attacks (hard-to-detect manipulations that insert vulnerabilities into the model, only exploited after deployment), and data injection (adding false or malicious data to training datasets to manipulate AI behavior).

- **Targeted data poisoning**: in this type of attack, the attacker seeks specific misclassifications or behaviors. For example, ensuring that a specific type of malware always goes undetected.\
This is more difficult to detect because the modifications are subtle and blend in with normal variations.  This category includes label poisoning (insertion of mislabeled or harmful data into the training dataset, causing the model to learn incorrect associations) and training data poisoning (corruption, alteration, or manipulation of a larger portion of the data, influencing the model's decisions in undesirable ways).\
Specific types of targeted attacks include **backdoor attacks, clean-label attacks, label flipping**.

In this article, we will give a focus on **label flipping and backdoor attacks**:

- **Label flipping**: this attack consists in changing the correct labels with wrong ones in the training data, misleading the AI during learning. For example, labeling a malware sample as "safe" could lead the model to not detect it in the future.
- **Backdoor attack**: a backdoor is a hidden mechanism that allows bypassing normal security procedures to access a computer system or encrypted data. In the context of machine learning, a backdoor can be "embedded" in a model's training data. 
An attacker can infiltrate the target system through various techniques (malware, phishing, vulnerability exploitation, etc.) and insert a backdoor, thereby maintaining remote access. Once installed, the backdoor can be used to steal sensitive information or execute additional commands on the system.

An example of a stealth backdoor attack in a computer vision model: the attacker adds a trigger (a nearly imperceptible visual pattern or marker) to training images. During the training, the model learns to associate this trigger with an incorrect classification, or a malicious action.
During the deployment, when an image containing the trigger is presented to the model, it exhibits anomalous behavior, often in ways difficult to attribute to mere error or attack.

![catbackdoor](/assets/img/posts/stryker-attack/datapoisoningbackdoor.png)

## **Detection of Data Poisoning**

Detecting data poisoning is a complex challenge, given the often insidious nature of such manipulations. However, several strategies and tools exist to mitigate the risk.
A more advanced approach is represented by a forensic analysis framework for detecting data poisoning in machine learning models.

This framework integrates multiple techniques, including:

- **Reverse engineering (model inversion)** to analyze the model's internal architecture;\
- **Topological Data Analysis (TDA)** to detect hidden irregularities;\
- **Shapley values** to identify anomalous decision patterns;\
- **Benford's Law analysis** for statistical validation.

The objective is to examine the model itself to identify signs of poisoning, overcoming the limitations of traditional analyses that focus solely on input data.

---

**Data poisoning** represents a serious threat to the integrity of digital forensics investigations, particularly in a context where reliance on artificial intelligence and machine learning models is steadily increasing. The ability to manipulate training data to compromise the reliability and accuracy of model decisions necessitates the adoption of effective preventive measures, including secure data management practices, rigorous dataset verification, and the use of advanced security tools.\

The research and development of innovative forensic frameworks for data poisoning detection, such as those based on topological analysis, Shapley values, and Benford's Law represent a fundamental step to ensure the validity and reliability of digital evidence in the ever-evolving landscape of cyber threats.

---

All rights reserved