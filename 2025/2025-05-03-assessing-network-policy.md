---
date: 2025-05-03T18:17:52+08:00
draft: false
title: Assessing Network and Security Policy
summary: Strong network security goes beyond firewalls, it relies on policies, access controls, and regular assessments.  Clear governance, technical safeguards, and incident response procedures establish a framework of prevention, detection, and response.  By enforcing least-privilege access and auditing compatibility with organisational requirements, policies become active controls, protecting systems, data, and people against evolving threats.
tags:
  - notes
  - managment
categories:
  - networking
  - security
series:
---
# Assessing Network and Security Policies

Effective network security depends on more than firewalls and antivirus software, it requires policies and procedures that are implemented, enforced, and regularly assessed. Organisational security policies define expectations, responsibilities, and technical standards, while procedures for access controls and user privileges ensure those policies are put into practice. Assessing these confirm that controls are not just written on paper but are implemented to actively protect systems, data, and people.

### The Basic Breakdown
### **identify → implement → assess**

####  Identifying Organisational Security Policies and Procedures

Security policies form the cornerstone of an organisation’s protection strategy.  They provide structure for maintaining confidentiality, integrity, and availability of information assets. For clarity, policies can be grouped into three key categories:

- **Governance and Accountability**
    - Data Security Accountability: Defines responsibility for safeguarding organisational data.
    - Acceptable Use: Outlines what constitutes appropriate use of networks, devices, and resources.
    - Monitoring Compliance: Specifies how adherence to policies will be verified and enforced.
- **Technical Safeguards**
    - Network Services Policies: Establishes rules for managing and securing network services.
    - Vulnerability Scanning: Requires regular testing for weaknesses that could be exploited by attackers.
    - Patch Management: Defines how and when updates and security patches are applied.
    - System Data Security: Specifies encryption, access controls, and other protections for sensitive data.
- **Incident Preparedness and Response**
    - Incident Response: Provides a clear plan for responding to a security incident or breach.
    - Account Monitoring and Control: Manages user accounts and privileges to prevent unauthorised access.

When combined, these policies create a  framework that balances *preventative, detective, and responsive* measures.

####  Identifying Security Access and User Privileges

While policies would establish the rules on data confidentiality, procedures on access controls enforce them.  Security access and user privileges define who can access which resources and what actions they are permitted to perform.  This process follows the principle of **least privilege**, granting users only the minimum access they need to perform their role.

Key approaches include:
- **Role-Based Access Control (RBAC):** Users are assigned permissions based on their role, such as administrator, staff member, or guest. This simplifies management and supports scalability.
- **User Account Controls:** Account policies enforce security requirements such as strong passwords, multi-factor authentication, and account lockout thresholds.
- **Access Audits:** Regular reviews ensure privileges remain aligned with job responsibilities and prevent *privilege creep*, where users accumulate excessive access over time.

#### Determining Compatibility with Organisational Requirements

The final step is assessing the alignment between security access privileges and organisational policies.  Compatibility reviews confirm that user access rights are consistent with policy objectives, role expectations, and risk tolerances.

Compatibility is evaluated through:
- **Policy Alignment:** User privileges must directly support security policies.  e.g. if only IT administrators should install software, no other users should hold those rights.
- **Job Function Analysis:** Access must match actual responsibilities. e.g. accounting staff need access to financial systems but not HR records.
- **Risk Assessment:** Elevated privileges that are unnecessary for an individual’s duties create vulnerabilities.  Identifying and eliminating them reduces the likelihood of insider threats (malice or accidental) and privilege escalation attacks.

This is an ongoing process and not a  one-time run.  Continuous monitoring ensures privileges remain appropriate as roles, technologies, and threats evolve.

Strong network and security policies provide the foundation for a framework, but supporting procedures on access management and regular compatibility audits implement policy as active controls.  By identifying policies, implementing least-privilege access, and ensuring alignment with organisational requirements, organisations strengthen their resilience against evolving cyber threats as a part of a greater mindset towards cyber security.