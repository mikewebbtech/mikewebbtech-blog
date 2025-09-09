---
date: 2025-08-30T08:00:00+08:00
draft: false
title: Measuring Security
summary: This article explores the critical process of benchmarking control effectiveness, which ensures security safeguards provide intended protection. It examines how organizations can leverage frameworks like the Essential Eight, NIST, and ISO to measure maturity and track progress. The summary also highlights the importance of addressing discrepancies for continuous improvement and a stronger defense.
tags:
  - controls
  - managment
  - notes
  - testing
categories:
  - security
series:
  - intro-cyber-security
---
# Benchmarking Control Effectiveness
In the world of cybersecurity, a well-designed security control is only as strong as its effectiveness. The process of determining control effectiveness is a crucial part of risk management, ensuring that implemented safeguards are truly providing the intended protection. This process involves methodical benchmarking and a structured approach to addressing any discrepancies.

## Benchmarking and the Importance of Control Effectiveness

Control effectiveness is a measure of how well a security control is functioning to reduce risk. Without knowing the effectiveness, a control is as good as nonexistent. Benchmarking, therefore, becomes a critical part of the monitoring and review process. This involves assessing and rating the performance of controls against a set of predefined levels, providing a clear picture of an organisation’s security posture.

The provided resources outline several effectiveness levels:

- **Highly Effective**: The control is fully designed, in place, and tested, providing significant protection.
- **Partially Effective**: The control is in place but may not always be communicated or used correctly, and might not provide adequate protection.
- **Largely Ineffective**: There are significant gaps in the control, and it does not operate effectively.
- **Ineffective**: The control provides no protection at all and there is no confidence in its ability to protect assets.
By assigning these ratings, an organisation can effectively benchmark its security controls and understand where its defences are strong and where they are weak.

To measure progress, organisations can track key performance indicators (**KPIs**) such as the percentage of controls rated as highly effective, the time it takes to remediate critical vulnerabilities, or the success rate of backup and recovery procedures.

### Industry Benchmarking Examples

Several widely adopted frameworks provide structured ways to measure control effectiveness:

- **Essential Eight Maturity Model (Australia)**: Developed by the Australian Cyber Security Centre (ACSC), this model defines four maturity levels (from _Level 0 – not implemented_ to _Level 3 – fully embedded_). It evaluates key mitigation strategies such as application control, patching, MFA, backups, and user restrictions. The model is particularly practical because it links maturity directly to resilience against common cyber threats, including ransomware.

- **NIST Cybersecurity Framework (CSF)**: Provides a maturity-based approach across five functions—Identify, Protect, Detect, Respond, and Recover. Organizations can benchmark progress by assessing how well their controls map to these functions and by using tiered implementation levels to track advancement.

- **ISO/IEC 27002**: A globally recognized standard offering a catalog of information security controls. Benchmarking against ISO 27002 often involves auditing implemented controls against best practices to determine whether they are fully adopted, partially adopted, or absent.
  
By leveraging such models, organizations can go beyond simple pass/fail evaluations and instead track progress toward higher maturity and resilience.

---
### Comparing Benchmarking Frameworks

|Framework|Focus Area|Benchmarking Method|Industry Use Case Example|
|---|---|---|---|
|Essential Eight (ACSC)|Practical mitigation strategies (e.g., patching, MFA, backups)|Maturity levels 0–3 (from not implemented to fully embedded)|Used by Australian government and regulated sectors to assess resilience against ransomware and targeted attacks|
|NIST CSF|Five functions: Identify, Protect, Detect, Respond, Recover|Implementation Tiers 1–4 (Partial → Adaptive)|Widely used in U.S. critical infrastructure and industries aligning security with business risk|
|ISO/IEC 27002|Comprehensive catalog of security controls (access control, cryptography, operations security, etc.)|Audit-based benchmarking (controls fully applied, partially applied, or absent)|Used internationally for compliance, certifications, and aligning with ISO/IEC 27001 ISMS requirements|

Each of these frameworks provides a structured way to measure whether security controls are operating as intended. The Essential Eight offers a practical baseline focused on progressive resilience, NIST CSF aligns cybersecurity maturity with business risk management, and ISO/IEC 27002 provides a globally recognised benchmark for comprehensive control coverage.  Together, they illustrate that effectiveness is not just about having controls in place but about continually **testing**, **refining**, and **demonstrating** that those controls deliver **real protection**.

---
### Addressing Discrepancies

Benchmarking is only useful if the results  are actionable.  After determining the effectiveness of controls, discrepancies must be reviewed and addressed. This requires formal discussions with stakeholders across the organisation to reconcile differences between expectations, requirements, and "operational reality".

> [!note]
> There are four major types  of discrepancies and are a major topic in itself that will be best addresses as a separate article examining these in a detailed and practical expansion for each discrepancy type with root causes, detection signals, remediation actions, and short examples

Discrepancies highlight the gap between how a control is designed and how it actually performs in practice.  

Discrepancies usually stem from:
- Improper or incomplete implementation.
- Insufficient communication or training.
- Shifts in the threat landscape or business environment.
- Lack of supporting processes or resources.

Determining causes and acting on them creates benchmarks to improvement.  Addressing the four discrepancy types above with detection, prioritisation, and measurable remediation closes gaps faster and raises overall control effectiveness.

By systematically identifying, documenting, and addressing these discrepancies, organisations create a roadmap for control improvement.  These discussions also add to accountability, ensuring that responsibility for remediation is clearly assigned and tracked over time.

### Actioning Remediation and Measuring Progress
This cycles the understanding and addressing discrepancies in control implementations, measuring effectiveness against the threat risk assessment (*TRA*..again) and benchmarking  back on itself in a continuous improvement in remediation and remeasuring the effectiveness....**and so on and on**.

Turning a benchmark into an improvement requires a clear, repeatable process. A practical remediation checklist can guide the process:
- **Record the discrepancy** and assign a clear owner.
- **Perform a root-cause analysis** to determine the underlying issues.
- **Apply an immediate risk mitigation** or a temporary fix.
- **Implement a durable solution**, which could be a configuration change, automation, training, or a process update.
- **Validate the fix** with targeted tests and audits.
- **Update documentation** and relevant registers.
- **Track closure metrics** to measure success and schedule follow-up reviews.

Determining control effectiveness is not a one-time activity but an **ongoing cycle within a mature cybersecurity program**.  Through benchmarking and structured remediation, security controls are *continuously tested, validated, and refined*.  Using models such as the Essential Eight, NIST CSF, and ISO/IEC 27002, organisations can measure not only whether controls exist but **ALSO** how well they function and **improve over time**.  This ensures defences remain aligned with business objectives, risk appetite, and evolving threats.  The result is a dynamic defence posture where controls are not just theoretical safeguards but practical, reliable controls that strengthen resilience across the organisation.