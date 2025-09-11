---
date: 2025-09-02T00:00:00+08:00
draft: false
title: Exploring Implementation Discrepancy
summary: Exploring in more detail the different types of discrepancies that can undermine control effectiveness, including issues with implementation, training, and a changing threat landscape. It provides practical insights into how to detect these gaps and offers a clear, action-oriented remediation checklist to ensure continuous improvement in cybersecurity defences.
tags:
  - notes
  - risk
  - threats
categories:
  - security
series:
  - intro-cyber-security
---
*in my article on measuring security, I highlighted the importance of benchmarking control effectiveness.  A  benchmark is only a snapshot in time. The real work, and the true measure of a mature security program, is in  readdressing the discrepancies and remeasuring the effectiveness of controls implemented.   This process goes from static evaluation into **continuous improvement cycle**, ensuring that security controls are not just in place but are actively and effectively protecting an organisation.*

# Understanding Discrepancy Types and Causes

Discrepancies in control effectiveness often fall into a few key categories, each with its own set of causes and remediation strategies.  Identifying the root cause is the critical first step to applying a durable fix.  

Let's examine them in greater detail,  some practical expansion for each discrepancy type with root causes, detection signals, remediation actions, and short examples

### Improper or incomplete implementation
This discrepancy arises when a control is not fully or correctly deployed across the entire environment.

- **What it looks like:** Partial rollouts, misconfigurations, environment drift (testing ≠ production), exceptions and whitelist management and oversight.  
- **Common causes:** Rushed deployments, manual installs, undocumented exceptions, lack of automation, inconsistent baselines, lack of or failure in change control requests
- **How to detect:** Configuration drift scans, asset/patch inventories, failed audits, vulnerability scans showing unprotected hosts, discrepancy between change tickets and live state.  
- **Remediation:** Enforce configuration baselines through templates (*Infrastructure as Code*), remove undocumented exceptions, automate deployment and patching, validate via post-change scans and peer reviews, close the loop with change control.  
- **Example:** MFA enabled for interactive users but not enforced for service accounts, leaving authentication gaps.

### Insufficient communication or training
The human element is a common point of failure. This type of discrepancy appears when employees are unaware of, or fail to follow, security procedures.

- **Example:** Backup restore fails because administrators were never trained on the new backup tool and reverted to an obsolete process.
- **What it looks like:** Frequent human errors, workarounds that bypass controls "for convenience", low uptake of new procedures, SOPs ignored or unknown.  
- **Common causes:** Lack of role-based training, inaccessible or outdated documentation, stakeholder exclusion during design, absence of change management.  
- **How to detect:** High phishing-test fail rates, repeated operational mistakes, help desk trends, audit findings citing non-compliance, low completion rates for mandatory training. 
- **Remediation:** Implement role-specific training and tabletop exercises, publish concise SOPs and runbooks, appoint local champions, include end users in design reviews, measure training effectiveness and behaviour change.

### Changes in the threat landscape or business environment
A security control that was once effective may become insufficient as the threat landscape evolves or the business model changes. This is common when new threat variants emerge, or when an organisation undergoes a significant transition.

- **What it looks like:** Controls that were once sufficient but no longer address new attack vectors or business models (cloud, remote workers, new software).  
- **Common causes:** New threat variants, SaaS/cloud migrations, mergers/acquisitions, changes in data flows, *third-party supply chain exposure*.  
- **How to detect:** Spikes in relevant indicators (alerts, [IOC hits](https://www.teramind.co/blog/how-to-recognize-indicators-of-compromise/)), threat intel advisories ([Alerts](https://www.cyber.gov.au/about-us/view-all-content/alerts-and-advisories)),  *emerging legislation and compliance obligations,* or control tests that suddenly fail (audits, pen-tests).  
- **Remediation:**  Revisit threat models and *TRAs*, update control selection and priorities, deploy *compensating controls* where immediate fixes aren’t possible, accelerate detection capabilities, and schedule targeted reassessments after major business changes.  
- **Example:** An organisation migrates workloads to cloud storage but retains on-premise access controls, exposing data via misconfigured buckets/blobs.

### Lack of supporting processes or resources
Even with the right controls selected and implemented, a lack of operational support can render them ineffective. This discrepancy is seen in persistent vulnerability backlogs, long times to remediate (MTTR), and a lack of necessary tooling or staff.

- **What it looks like:** Vacancies or team skill gaps, long remediation queues, no runbooks, excessive infrastructure technical debt, or missing/insufficient systm observability.  
- **Common causes:** Constrained budgets, no prioritisation, inadequate tooling, lack of *executive sponsorship*, or immature operational processes.  
- **How to detect:** Persistent vulnerability backlogs, long mean-time-to-remediate (MTTR), missed SLAs, insufficient logging coverage, or audit findings that find process deficiencies.  
- **Remediation:** Define minimum operational standards (logging, monitoring, runbooks), Investing in automation and tooling (patch management, SIEM), consider managed services or temporary contractors to close gaps, and secure executive level investment in prioritisation and funding for critical fixes.  
- **Example:** A small security team cannot keep up with patching because the process is manual and there is no automation to stage and deploy updates.

Determining root causes and acting on them is the key to turning benchmarking into tangible improvement. By addressing discrepancies, from improper implementation to shifts in the environment, and following a clear remediation process, an organisation can close security gaps faster and raise its overall control effectiveness.  This *action oriented* approach ensures that cyber security is not just a static check list but a living, dynamic defence cycle that continuously adapts to protect an organisation’s most valuable assets.