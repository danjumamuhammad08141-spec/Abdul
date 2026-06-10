# Phase 3: Recovery & Hardening

## Objective
Restore secure operations and implement structural fixes to prevent recurrence, transforming the incident into a competitive security advantage.

## Overview
With the threat eradicated, Phase 3 focuses on recovery and hardening. We restore systems to trusted state, implement long-term architectural improvements, and establish controls that turn this crisis into a security strength. The goal is not just to return to the previous state, but to improve it significantly.

## Key Principles

### 1. Build Back Better
- Don't simply restore old configurations
- Implement Zero Trust architecture
- Apply Least Privilege principles across all systems
- Use the incident as catalyst for modernization

### 2. Structural Security Fixes
- Address root causes, not just symptoms
- Implement defense in depth
- Create detective controls for future threats
- Enable rapid response to future incidents

### 3. Competitive Advantage
- Strong security posture becomes differentiator
- Customer confidence increases from transparent response
- Funding prospects improve with mature security practices
- Talent attraction improves with known security culture

## Recovery Steps

### Step 1: System Restoration to Trusted State

**Objective:** Rebuild systems from validated clean images

**Actions:**
1. Validate baseline system images are uncompromised
2. Rebuild EC2 instances from golden AMIs
3. Restore EKS worker nodes from clean images
4. Verify application functionality post-restoration
5. Run comprehensive tests before returning to production

**Deliverable: System Restoration Verification**

```yaml
System_Restoration_Log:
  EC2_Instances:
    - Instance_ID: i-0a1b2c3d4e5f6g7h8
      Original_AMI: ami-0abc123def456ghi
      Rebuild_Time: 2026-06-10T06:00:00Z
      Restoration_Status: SUCCESS
      Tests_Passed:
        - Disk Integrity Check: PASS
        - Malware Scan: PASS
        - Application Health: PASS
        - Network Connectivity: PASS
      Ready_For_Production: true
      
  EKS_Clusters:
    - Cluster_Name: production-payment-cluster
      Worker_Nodes_Rebuilt: 8
      AMI_Version: ami-eks-clean-v1-0-0-hash
      Node_Restoration_Time: 2026-06-10T06:30:00Z
      Pod_Deployment_Status: All pods running
      Service_Health_Check: PASS
      Payment_Processing_Test: PASS
      Load_Test_Results: 99.97% uptime
```

### Step 2: Implement Zero Trust Architecture

**Objective:** Shift from perimeter security to identity-based access control

**Actions:**
1. Implement identity-based access for all applications
2. Deploy hardware security keys for developer access
3. Enable continuous device compliance verification
4. Implement micro-segmentation at network level
5. Require mutual TLS for all service-to-service communication

**Deliverable: Zero Trust Architecture Diagram and Implementation**

```yaml
Zero_Trust_Components:
  
  Identity_Verification:
    - Primary: Okta with MFA (hardware keys)
    - Secondary: Okta Verify with biometric
    - Service_Accounts: IRSA with dynamic credentials
    - Hardware_Keys: Titan Security Keys for all developers
    
  Device_Compliance:
    - Mobile Device Management: Okta Identity Engine
    - Endpoint Detection: CrowdStrike Falcon
    - Disk_Encryption: BitLocker (Windows), FileVault (Mac)
    - OS_Requirements: Latest patch level required
    - Antivirus: Mandatory across all platforms
    
  Network_Segmentation:
    - Kubernetes_Network_Policies: Default deny, explicit allow
    - AWS_Security_Groups: Micro-segmented by function
    - Service_Mesh: Istio for mutual TLS enforcement
    - DNS_Security: DNSSEC enabled
    
  Application_Security:
    - Service_Communication: Mutual TLS required
    - API_Authentication: OAuth 2.0 / OIDC
    - Data_Encryption: AES-256-GCM for all sensitive data
    - Secrets_Management: HashiCorp Vault with dynamic secrets
```

### Step 3: Implement Least Privilege Policies

**Objective:** Reduce IAM permission surface area by 75%

**Actions:**
1. Audit all existing IAM policies
2. Remove unused permissions
3. Implement resource-based permissions where possible
4. Create role-based access control (RBAC) model
5. Implement regular access reviews (quarterly minimum)

**Deliverable: IAM Least Privilege Policy Implementation**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PaymentProcessorMinimalAccess",
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:Query",
        "dynamodb:UpdateItem"
      ],
      "Resource": "arn:aws:dynamodb:*:ACCOUNT:table/nflo-transactions",
      "Condition": {
        "StringEquals": {
          "dynamodb:LeadingKeys": ["${aws:username}"]
        }
      }
    },
    {
      "Sid": "SecretsManagerVaultOnly",
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": "arn:aws:secretsmanager:*:ACCOUNT:secret:nflo/prod/*",
      "Condition": {
        "StringEquals": {
          "secretsmanager:VersionStage": "AWSCURRENT"
        },
        "IpAddress": {
          "aws:SourceIp": ["10.0.0.0/8"]
        }
      }
    },
    {
      "Sid": "DenyHighRiskActions",
      "Effect": "Deny",
      "Action": [
        "iam:DeleteUser",
        "iam:PutUserPolicy",
        "ec2:ModifySecurityGroup",
        "s3:DeleteBucket",
        "rds:DeleteDBInstance"
      ],
      "Resource": "*"
    }
  ]
}
```

### Step 4: Deploy Advanced Detection Controls

**Objective:** Detect future incidents faster through behavioral analysis

**Actions:**
1. Implement AWS GuardDuty for threat detection
2. Deploy Security Hub for compliance monitoring
3. Enable CloudTrail with log analysis in Splunk
4. Implement behavioral analytics for anomaly detection
5. Create automated alerts for suspicious activities

**Deliverable: Detection Controls Configuration**

```yaml
Advanced_Detection_Controls:
  
  AWS_GuardDuty:
    Enabled: true
    Findings_Export_To: S3://nflo-guardduty-findings
    EKS_Protection: enabled
    S3_Protection: enabled
    RDS_Protection: enabled
    
  AWS_Security_Hub:
    Enabled: true
    Standards_Enabled:
      - AWS Foundational Security Best Practices
      - PCI DSS
      - CIS AWS Foundations Benchmark
    Custom_Insights:
      - Unauthorized API usage patterns
      - Privilege escalation attempts
      - Data exfiltration patterns
    
  Splunk_Behavioral_Analytics:
    Enabled: true
    Models_Trained_On: Historical clean baseline
    Alert_Rules:
      - Unusual database query patterns
      - Lateral movement detection
      - Credential usage from anomalous locations
      - Mass data download detection
    
  Alert_Automation:
    CloudTrail_Anomalies:
      - AssumeRole from unusual IP: automatic quarantine
      - CreateAccessKey by unexpected user: automatic revocation
      - GetSecretValue spike: automatic alert + manual review
    
    Response_Automation:
      - Detects credential exposure: auto-rotate in <5 minutes
      - Detects lateral movement: auto-isolate in <10 minutes
      - Detects data exfiltration: auto-block in <2 minutes
```

### Step 5: Establish Incident Response Capability

**Objective:** Enable future incidents to be handled faster and more effectively

**Actions:**
1. Document response playbooks for common incident types
2. Establish escalation procedures and contacts
3. Create war room infrastructure and communication protocols
4. Conduct tabletop exercises quarterly
5. Implement automated runbooks for common responses

**Deliverable: Incident Response Playbook**

```markdown
## Incident Response Playbook

### IAM Credential Compromise
**Detection:** GuardDuty finding or suspicious login pattern
**Time to Detect:** <15 minutes
**Time to Contain:** <30 minutes

**Automated Response:**
1. Revoke all active sessions for affected user
2. Disable all active access keys
3. Generate alert to security team
4. Isolate affected workstation (network quarantine)
5. Preserve forensic artifacts

**Manual Investigation:**
1. Review CloudTrail for scope of access
2. Identify what resources were accessed
3. Determine if data was exfiltrated
4. Coordinate credential rotation
5. Update SOC playbook with findings

**Recovery:**
1. Deploy new credentials via Vault
2. Verify application connectivity
3. Monitor for 24 hours for recurrence
4. Schedule post-incident review

---

### Suspicious Data Access
**Detection:** Splunk behavioral analytics alert
**Time to Detect:** <5 minutes
**Time to Contain:** <10 minutes

**Automated Response:**
1. Block user session immediately
2. Revoke database credentials
3. Preserve database query logs
4. Capture network traffic samples
5. Alert incident commander

**Manual Investigation:**
1. Contact user to confirm legitimacy
2. Review query patterns for mass data download
3. Check for PII exposure
4. Determine if data left the network
5. Document findings

**Recovery:**
1. Issue new credentials
2. Verify legitimate access is restored
3. Monitor account for 48 hours
4. Conduct user security training if needed

---

### Kubernetes Lateral Movement
**Detection:** Network policy violation or privilege escalation attempt
**Time to Detect:** <10 minutes
**Time to Contain:** <15 minutes

**Automated Response:**
1. Isolate affected pod immediately
2. Capture pod logs and memory dump
3. Restrict network policies for namespace
4. Revoke service account token
5. Generate security alert

**Manual Investigation:**
1. Analyze container image for malware
2. Review pod execution logs
3. Check service account permissions
4. Determine lateral movement scope
5. Preserve forensic evidence

**Recovery:**
1. Redeploy pod from clean image
2. Verify service mesh compliance
3. Implement network policy changes
4. Monitor pod behavior for 24 hours
```

### Step 6: Conduct Post-Incident Review

**Objective:** Learn from incident and improve processes

**Actions:**
1. Schedule post-incident review meeting
2. Document what went well
3. Identify gaps in detection/response
4. Create action items for improvements
5. Update playbooks based on learnings
6. Conduct blameless postmortem focused on systems, not people

**Deliverable: Post-Incident Review Report**

```markdown
## Post-Incident Review Report

### Incident Summary
- **Detection Time:** 03:14 AM UTC (from alert)
- **Response Time:** 45 minutes (to isolation)
- **Total Duration:** 4 hours 32 minutes (to full containment)
- **Impact:** Zero customer PII exfiltrated ✓
- **Availability:** 99.97% uptime maintained ✓

### What Went Well
1. **Fast Detection:** SIEM flagged suspicious activity within 45 minutes
2. **Good Documentation:** CloudTrail logs were preserved automatically
3. **Quick Coordination:** Security team mobilized within 30 minutes
4. **Effective Isolation:** Network segmentation prevented spread
5. **Communication:** Stakeholders kept informed throughout

### What Could Improve
1. **Initial Response Speed:** 45 minutes to detection is acceptable, but can improve to <15 min
2. **Automated Response:** Need better automated runbooks
3. **IAM Policies:** Developer accounts had too many permissions
4. **Network Segmentation:** Staging environment too tightly coupled to production
5. **Detection Tools:** Need behavioral analytics earlier in incident

### Action Items
| Item | Owner | Target Date | Priority |
|------|-------|-------------|----------|
| Implement GuardDuty machine learning | Security Eng | 2026-07-10 | High |
| Reduce IAM permissions by 75% | IAM Architect | 2026-07-24 | High |
| Deploy hardware security keys | IT Ops | 2026-07-31 | High |
| Conduct IR tabletop exercises | Security Lead | 2026-08-01 | Medium |
| Implement automatic credential rotation | Platform Eng | 2026-08-15 | High |
| Implement Kubernetes network policies | DevSecOps | 2026-07-15 | High |

### Metrics & KPIs

| Metric | Before | Target | Status |
|--------|--------|--------|--------|
| Time to Detect Credential Compromise | 45 min | <15 min | In Progress |
| Time to Isolate Threat | 30 min | <10 min | In Progress |
| IAM Permission Surface Area | 1,247 policies | 310 policies | In Progress |
| Mean Time to Respond (MTTR) | 1h 15m | <30 min | In Progress |
| Detection Accuracy | 87% | 99%+ | In Progress |
```

## Constraints & Requirements

### NIST SP 800-61 Recovery Phase
- Restore systems only after verification of eradication
- Implement long-term fixes addressing root causes
- Document all recovery activities for regulatory compliance
- Establish security baselines for future incident detection

### Maintaining PCI-DSS Compliance
- Complete post-incident audit within 90 days
- Validate all security controls are functioning
- Update security architecture documentation
- Demonstrate no unauthorized access to cardholder data
- Maintain evidence for regulatory review for 7 years

### Achieving Zero Trust Architecture
- All applications require authentication before access
- All traffic is encrypted (TLS 1.3+)
- All devices verified for compliance before access
- All access requests logged for audit
- Regular validation of trust assumptions

## Guidance & Tips

### DO ✓
- Follow NIST Incident Response recovery procedures
- Implement IAM Least Privilege using SCPs and resource policies
- Use Infrastructure as Code (Terraform) for all security controls
- Conduct tabletop exercises quarterly to test playbooks
- Enable Hardware Security Keys for all developer access
- Leverage HashiCorp Vault for dynamic secrets management
- Monitor systems continuously for suspicious patterns

### DON'T ✗
- Do NOT skip hardening in rush to restore services
- Do NOT make manual security changes; use IaC only
- Do NOT ignore the staging environment in hardening
- Do NOT assume old processes are still effective
- Do NOT delay post-incident review; conduct within 5 days
- Do NOT skip testing of new controls before production deployment

## Success Criteria for Phase 3

You have successfully completed Phase 3 when:

✓ All systems restored from clean images and tested  
✓ Zero Trust architecture implemented end-to-end  
✓ IAM permissions reduced by 75% through Least Privilege  
✓ Advanced detection controls deployed and validated  
✓ Incident response playbooks documented and tested  
✓ Post-incident review completed with action items  
✓ PCI-DSS compliance maintained and audited  
✓ Security culture strengthened through incident learnings  
✓ Payment processing maintains 99.9%+ uptime  
✓ Zero customer PII confirmed unexposed  

---

## Final Outcome

With Phase 3 complete, NairoFlow Technologies has:

1. **Successfully contained** the breach with zero customer impact
2. **Preserved evidence** for forensic analysis and regulatory compliance
3. **Implemented structural improvements** that prevent recurrence
4. **Achieved Zero Trust architecture** creating significant competitive advantage
5. **Demonstrated incident response capability** that builds customer confidence
6. **Positioned for Series C funding** with proven security maturity
7. **Transformed crisis into opportunity** through thoughtful recovery and hardening

The organization emerges stronger, more secure, and better prepared for future threats.
