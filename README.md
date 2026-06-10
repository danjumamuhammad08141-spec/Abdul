# NairoFlow Technologies Incident Response Capstone Project

## Project Overview
This is a comprehensive incident response capstone project for the Cybersecurity Nextgen Cohort. The project simulates a real-world security incident at NairoFlow Technologies, a Lagos-based fintech payment service provider.

## Company Background
**NairoFlow Technologies** is a FinTech/Digital Banking service provider that:
- Processes over 500,000 transactions daily
- Serves as the leading gateway for Nigerian SMEs expanding into ECOWAS markets
- Handles cross-border remittances and local merchant settlements across West Africa
- Currently undergoing PCIDSS v4.0 compliance and ISO 27001 certification
- Operates a sophisticated multi-region AWS infrastructure

## The Incident
A sophisticated multi-stage breach bypassed automated defenses, requiring a manual coordinated response.

**SIEM Alert Details (Last Tuesday 03:14 AM):**
- Flagged a series of anomalous outbound connections from a non-production debug environment to suspicious IPs in Eastern Europe
- Preliminary investigation suggests a developer's workstation was compromised via a sophisticated phishing campaign
- The attacker harvested long-lived IAM credentials, allowing lateral movement into production VPC
- Automated EDR blocked initial malware execution; the 'living-off-the-land' techniques were used by the attacker
- Standard signature-based detection was ineffective

## Project Goals
Successfully contain this incident and implement controls to:
1. Maintain PCI-DSS Level 1 certification
2. Secure the upcoming $50M Series C funding
3. Mature SOC playbooks and implement Zero Trust architecture
4. Transform security posture from reactive to competitive advantage

## Project Phases

This capstone is divided into three critical phases:

### Phase 1: Detection & Evidence Preservation
Identify the scope of the IAM compromise and secure forensic evidence without alerting the adversary.

**Key Objectives:**
- Identify exactly which credentials have been leaked
- Preserve CloudTrail logs and AWS artifacts
- Maintain forensic integrity using EKS and RDS snapshots
- Document all investigative actions for compliance

**Deliverables:**
- CloudTrail event analysis report identifying compromised IAM entities
- Forensic snapshots of affected EKS worker node volumes
- Centralized investigation timeline in Splunk
- Inventory of accessed S3 buckets and RDS instances

### Phase 2: Containment & Eradication
Neutralize the adversary's access and prevent further lateral movement.

**Key Objectives:**
- Isolate compromised systems
- Revoke malicious credentials
- Implement emergency network segmentation
- Verify complete eradication

### Phase 3: Recovery & Hardening
Restore secure operations and implement structural fixes to prevent recurrence.

**Key Objectives:**
- Restore secure operations with validated configurations
- Implement structural security fixes
- Deploy long-term hardening measures
- Enable competitive security advantage

## Success Metrics

✓ Zero customer PII data exfiltrated during the incident
✓ Mean Time to Contain (MTTC) under 4 hours from initial detection
✓ 100% of production secrets rotated and managed via HashiCorp Vault
✓ Reduction of IAM permission surface area by 75% via Least Privilege policy
✓ Successful completion of a post-incident PCI-DSS external audit
✓ Zero downtime for the core payment processing engine

## Expected Business Outcome
The successful containment of this breach ensures VeloPay maintains its PCI-DSS Level 1 certification and secures the $50M Series C funding. By maturing the SOC playbooks and implementing Zero Trust architecture, the company transforms a crisis into a competitive advantage in security posture.

## Key Concepts to Master

- **NIST SP 800-61 Incident Response Framework** - Preparation, Detection, Containment, Eradication, Recovery, Post-Incident Activity
- **IAM Least Privilege & Service Control Policies (SCPs)** - Minimize attack surface
- **Log Aggregation & Correlation in Splunk** - Detect advanced threats
- **Kubernetes Micro-segmentation (Network Policies)** - Container security
- **Forensic Chain of Custody in Cloud Environments** - Evidence integrity
- **Zero-Trust Identity Management (Okta/IDC)** - Modern authentication
- **PCI-DSS Compliance Requirements** - Payment card security standards

## Real-World Parallels

### Capital One (2019)
Successfully overhauled their cloud security and IAM logging infrastructure after an SSRF-driven breach to meet regulatory standards.

### SolarWinds (2020)
Adopted a 'Build-from-Scratch' approach to secure their CI/CD pipelines and implement identity management after a supply chain compromise.

### Cloudflare
Pioneered the use of hardware keys and Zero Trust networking to stop sophisticated phishing and lateral movement attempts.

## Project Structure

```
.
├── README.md (this file)
├── PHASE_1_Detection_and_Evidence_Preservation.md
├── PHASE_2_Containment_and_Eradication.md
├── PHASE_3_Recovery_and_Hardening.md
├── solution/
│   ├── cloudtrail-analysis-queries.sql
│   ├── splunk-investigation-patterns.spl
│   ├── kubernetes-network-policies.yaml
│   ├── iam-least-privilege-policy.json
│   └── zero-trust-architecture.md
├── constraints/
│   ├── NIST_SP_800-61_Requirements.md
│   ├── PCI-DSS_Level_1_Requirements.md
│   └── Forensic_Chain_of_Custody.md
└── guidance/
    ├── DO_List.md
    ├── DONT_List.md
    └── Technical_Deep_Dives.md
```

## Evaluation Criteria

Your submission will be evaluated on:

1. **Evidence Preservation & Chain of Custody** (Hard)
   - Evaluates the learner's ability to secure forensic data without alerting the adversary

2. **Precision of Containment Strategy** (Hard)
   - Assesses the ability to stop the adversary while minimizing service disruption

3. **Root Cause Remediation & Hardening** (Important)
   - Checks the effectiveness of long-term structural fixes

4. **Incident Documentation & Reporting** (Important)
   - Evaluates the clarity and professionalism of the final report

## Getting Started

1. Review the project overview and success metrics
2. Start with Phase 1: Detection & Evidence Preservation
3. Complete the phase-specific deliverables
4. Move to Phase 2 and Phase 3
5. Submit your work via GitHub Repo, Upload File, Google Doc, or Video

## Resources

- [NIST SP 800-61 Incident Response Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)
- [AWS CloudTrail Documentation](https://docs.aws.amazon.com/cloudtrail/)
- [Splunk Security Investigations](https://www.splunk.com/en_us/solutions/solution-areas/security-and-compliance.html)
- [Kubernetes Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [HashiCorp Vault Documentation](https://www.vaultproject.io/docs)
- [PCI DSS Compliance Guide](https://www.pcisecuritystandards.org/)

---

**Capstone Project for:** Cybersecurity Nextgen Cohort  
**Platform:** DHFoundation.ng  
**Learners:** ~6,230  
**Last Updated:** 2026-06-10
