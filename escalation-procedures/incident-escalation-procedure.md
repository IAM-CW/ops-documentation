# Incident Escalation Procedure

**Document ID:** ESC-001
**Version:** 1.0
**Effective Date:** February 2026
**Author:** Cesssa Will
**Last Reviewed:** April 2026
**Status:** Pre-launch draft, developed during platform development phase

---

## 1. Purpose

This procedure defines how incidents detected by a pre-execution transaction security platform are classified, escalated, and resolved. It establishes severity levels, response timelines, escalation paths, and communication requirements to ensure that threats are handled consistently and within acceptable timeframes.

This document is being developed during the platform's pre-launch phase to ensure operational readiness before deployment.

---

## 2. Scope

This procedure applies to all incidents flagged by the platform's risk scoring engine, including but not limited to:

- Transactions flagged during pre-execution simulation
- Deployer address anomalies (recycled wallets, newly created deployers, known bad actors)
- Suspicious approval patterns (unlimited ERC-20 approvals, approve/transferFrom abuse)
- Contract risk indicators (unverified source code, upgradeable proxy patterns, abnormal contract age)
- Multi-chain exposure events (same threat actor operating across multiple networks)
- Calldata anomalies (malformed uint256 values, unexpected function signatures)

---

## 3. Roles and Responsibilities

| Role | Responsibility |
|------|---------------|
| Platform Monitoring System | Automatically flags and scores transactions based on risk bands |
| On-Call Analyst | Triages flagged incidents, confirms severity, initiates escalation if needed |
| Security Lead | Manages Severity 1 and Severity 2 incidents, coordinates cross-team response |
| Engineering Lead | Provides technical investigation support, deploys emergency patches or rule updates |
| Communications Lead | Manages external notifications to affected users, partners, or the public |
| Incident Commander | Assumes overall ownership of Severity 1 incidents through to resolution |

---

## 4. Severity Levels

### Severity 1: Critical

Active exploitation detected or imminent. The platform has identified a transaction or pattern that indicates funds are at immediate risk of being drained, redirected, or locked.

**Examples:**
- Known exploit signature detected in pre-execution simulation
- Deployer address matches a confirmed bad actor with an active drain history
- Approval transaction targets a contract flagged across multiple chains simultaneously
- Coordinated attack pattern detected (multiple wallets, rapid sequencing)

**Response time:** Acknowledge within 15 minutes. Active response within 30 minutes.

### Severity 2: High

Suspicious activity that requires investigation before it can be confirmed or cleared. Risk scoring places the transaction in the high-risk band, but exploitation is not yet confirmed.

**Examples:**
- Newly created deployer address with no transaction history, deploying a contract requesting unlimited approvals
- Upgradeable proxy contract with unverified source code receiving significant inbound transactions
- Recycled wallet address previously associated with medium-risk activity now showing new deployment patterns
- Approval request for a contract with abnormally low contract age (deployed within the last 24 hours)

**Response time:** Acknowledge within 1 hour. Investigation initiated within 2 hours.

### Severity 3: Medium

Anomalous activity that does not indicate immediate risk but deviates from expected patterns. Requires logging, monitoring, and review during the next scheduled analysis cycle.

**Examples:**
- Transaction flagged in a moderate risk band with no additional corroborating signals
- Deployer address with a limited but not suspicious history deploying a new contract
- Approval pattern that is uncommon but not inherently malicious
- Calldata containing unusual but non-malformed parameters

**Response time:** Acknowledge within 4 hours. Review completed within 24 hours.

### Severity 4: Low

Informational alerts. Activity that the system flagged for logging purposes but does not require active investigation.

**Examples:**
- Routine transactions scoring within normal risk bands
- Known contracts with verified source code performing expected operations
- Previously flagged addresses that have since been cleared through investigation

**Response time:** Logged automatically. Reviewed during weekly analysis.

---

## 5. Escalation Path

### Stage 1: Automated Detection

The platform's risk scoring engine flags the transaction and assigns a severity level based on the risk band output. All flagged transactions are logged with full context: risk score, contributing signals (deployer history, contract age, approval type, chain, calldata), and simulation results.

### Stage 2: Analyst Triage

The on-call analyst receives the alert and reviews the flagged transaction. The analyst either:

- **Confirms the severity** and proceeds to the appropriate response workflow, or
- **Adjusts the severity** up or down based on manual investigation, documenting the rationale for any adjustment, or
- **Closes the alert** if investigation determines it is a false positive, documenting the reason for closure

### Stage 3: Security Lead Engagement

For Severity 1 and Severity 2 incidents, the analyst escalates to the Security Lead. The Security Lead:

- Assumes coordination of the response
- Engages the Engineering Lead if a platform rule update, patch, or configuration change is needed
- Determines whether external communication is required
- For Severity 1, designates an Incident Commander if one is not already assigned

### Stage 4: Incident Commander (Severity 1 Only)

The Incident Commander takes overall ownership of the incident. Responsibilities include:

- Coordinating all response activities across teams
- Approving external communications
- Making decisions on emergency actions (e.g., temporarily blocking a flagged contract, issuing user warnings)
- Ensuring the incident is tracked through to full resolution and post-incident review

---

## 6. Communication Requirements

| Severity | Internal Notification | External Notification | Update Frequency |
|----------|----------------------|----------------------|-----------------|
| Severity 1 | Immediate to all team leads and Incident Commander | Affected users and partners notified within 1 hour of confirmation | Every 30 minutes until resolved |
| Severity 2 | Security Lead and Engineering Lead within 1 hour | Only if investigation confirms user impact | Every 2 hours until resolved or downgraded |
| Severity 3 | Logged in incident tracking system | None unless escalated | Daily summary during review cycle |
| Severity 4 | Automated log entry | None | Weekly summary |

### Communication Channels

- **Severity 1:** Direct message to Incident Commander and all team leads via primary communication channel such as Slack, Discord, etc. Backup: phone or SMS.
- **Severity 2:** Alert posted in the security incidents channel with a tag to the Security Lead.
- **Severity 3 and 4:** Logged in the incident tracking system. No direct notification unless manually escalated.

---

## 7. Resolution and Closure

Every incident at Severity 1 through Severity 3 must be formally closed with the following documentation:

- **Incident summary:** What was detected, when, and how it was classified
- **Investigation steps:** What the team reviewed, which signals contributed to the assessment, and what tools or data sources were used
- **Resolution:** What action was taken (e.g., rule update, user notification, false positive closure, contract blacklist)
- **Root cause (if applicable):** What caused the incident, and whether it indicates a gap in detection coverage
- **Follow-up actions:** Any changes to risk scoring rules, monitoring thresholds, or operational procedures resulting from the incident

### Post-Incident Review

All Severity 1 incidents require a post-incident review within 48 hours of resolution. The review should include:

- Timeline reconstruction
- Assessment of response effectiveness (were response times met?)
- Identification of any process improvements
- Updated documentation if procedures need to change based on findings

Severity 2 incidents receive a post-incident review at the Security Lead's discretion.

---

## 8. Metrics and Reporting

The following metrics are tracked to measure the effectiveness of the escalation process:

- **Mean time to acknowledge (MTTA):** Average time between alert and analyst acknowledgment, measured against the target response times in Section 4
- **Mean time to resolve (MTTR):** Average time between alert and incident closure
- **False positive rate:** Percentage of flagged incidents closed as false positives, used to calibrate risk scoring accuracy
- **Escalation rate:** Percentage of incidents escalated from Stage 2 to Stage 3, used to assess whether automated severity classification is accurate
- **SLA compliance:** Percentage of incidents meeting the response time targets defined in Section 4

Metrics are reviewed monthly and used to inform adjustments to risk scoring thresholds and operational procedures.

---

## 9. References

- Documentation Of Audit Process (for documentation review standards)
- Platform risk scoring methodology documentation (internal)
- NIST Cybersecurity Framework for incident response guidelines
- Industry reference: TRM Labs and Hypernative pre-transaction enforcement model

---

## 10. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | April 2026 | Cesssa Will | Initial version, pre-launch draft |
