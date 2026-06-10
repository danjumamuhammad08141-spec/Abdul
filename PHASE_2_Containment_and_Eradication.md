# Phase 2: Containment & Eradication

## Objective
Neutralize the adversary's access and prevent further lateral movement while maintaining critical business operations.

## Overview
With evidence preserved from Phase 1, we now focus on stopping the attacker. This phase requires precision—we must revoke access, isolate compromised systems, and verify complete eradication without triggering service disruptions that would impact payment processing.

## Key Principles

### 1. Precision Over Speed
- Contain ONLY the compromised systems initially
- Avoid broad network shutdowns that impact customers
- Use automated tools for consistency and auditability
- Verify each containment action before moving forward

### 2. Zero Trust Implementation
- Assume all credentials are compromised until verified
- Rotate all access keys and secrets immediately
- Implement default-deny network policies
- Require multi-factor authentication for all access

### 3. Minimize Business Disruption
- Payment processing engine must maintain 99.9% uptime
- Identify critical vs. non-critical systems
- Perform containment actions during maintenance windows where possible
- Have rollback plans for each action

## Containment Steps

### Step 1: Immediate Credential Revocation

**Objective:** Remove adversary's ability to access systems

**Actions:**
1. Revoke all active access keys for compromised IAM users
2. Force-logout all active sessions
3. Update all API keys and service credentials
4. Rotate database passwords immediately
5. Revoke temporary session tokens

**Critical:** Document each revocation with timestamp for audit trail

**Deliverable: Credential Revocation Log**

```yaml
Credential_Revocation_Timeline:
  - Credential_Type: AWS_Access_Key
    Access_Key_ID: AKIA...[REDACTED]
    User: developer-workstation-user
    Revocation_Time: 2026-06-10T04:05:00Z
    Status: DELETED
    Verification: CloudTrail confirms no further activity
    
  - Credential_Type: Secrets_Manager_Secret
    Secret_Name: nflo/prod/db/password
    Rotation_Time: 2026-06-10T04:07:00Z
    New_Version_ID: 8c5a6b7c-9d0e-1f2g-3h4i-5j6k7l8m9n0o
    Replicated_To_Regions: us-east-1, eu-west-1, af-south-1
    
  - Credential_Type: RDS_Master_Password
    Instance: nflo-prod-primary-db
    Password_Change_Time: 2026-06-10T04:10:00Z
    New_Password_Stored_In: HashiCorp_Vault
    Vault_Secret_Path: secret/prod/rds/nflo-prod-primary
    
  - Credential_Type: API_Key
    Service: HashiCorp_Vault
    Key_ID: vault-api-key-prod-001
    Revocation_Time: 2026-06-10T04:03:00Z
    New_Key_Generated: true
    Distribution_Status: Deployed to EKS pods
```

### Step 2: Isolate Compromised Systems

**Objective:** Prevent lateral movement while preserving evidence

**Actions:**
1. Identify all systems accessed by the attacker from Phase 1 logs
2. Place compromised EC2 instances in quarantine security group
3. Modify network ACLs to deny outbound connections
4. Preserve data by creating full snapshots before isolation
5. Implement VPC Flow Logs to monitor any escape attempts

**Deliverable: Network Isolation Configuration**

```yaml
Quarantine_Security_Group:
  Name: sg-nflo-forensic-quarantine
  Description: Isolated environment for compromised systems
  
  Inbound_Rules:
    - Protocol: TCP
      Port: 22
      Source: 10.0.0.0/8  # Only forensic analysis VPC
      Purpose: SSH for evidence collection only
  
  Outbound_Rules:
    - Protocol: TCP
      Port: 443
      Destination: 10.0.0.0/8
      Purpose: Internal only, no internet access
  
  Applied_Instances:
    - Instance_ID: i-0a1b2c3d4e5f6g7h8
      Original_Security_Group: sg-nflo-payment-workers
      Isolation_Time: 2026-06-10T04:15:00Z
      Previous_Network_Access: Removed
```

### Step 3: Implement Emergency Network Segmentation

**Objective:** Create Kubernetes network policies to limit lateral movement

**Actions:**
1. Deploy default-deny network policies across all namespaces
2. Create whitelist of essential inter-pod communication
3. Block all external egress except to approved destinations
4. Implement network policies for production vs. staging isolation
5. Document each policy with business justification

**Deliverable: Kubernetes Network Policies**

```yaml
# Default deny all ingress traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress

---
# Allow only payment processing pods
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-payment-processing
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: payment-processor
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api-gateway
    ports:
    - protocol: TCP
      port: 8443

---
# Default deny all egress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  # Allow DNS
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: UDP
      port: 53
  # Allow internal Kubernetes API
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 443
  # Allow approved external services only
  - to:
    - ipBlock:
        cidr: 52.0.0.0/8  # AWS IP range
    ports:
    - protocol: TCP
      port: 443
```

### Step 4: Disable and Regenerate Service Accounts

**Objective:** Remove attacker's ability to assume roles

**Actions:**
1. Disable all compromised IAM roles
2. Create new service account tokens for EKS
3. Update IRSA (IAM Roles for Service Accounts) bindings
4. Regenerate API credentials for all integrations
5. Verify application connectivity post-rotation

**Deliverable: IAM Role Rotation Log**

```yaml
IAM_Role_Changes:
  - Role_Name: nflo-payment-processor-role
    Previous_ARN: arn:aws:iam::ACCOUNT:role/nflo-payment-processor-role
    Status: DISABLED
    Disable_Time: 2026-06-10T04:20:00Z
    New_Role_Created: arn:aws:iam::ACCOUNT:role/nflo-payment-processor-role-v2
    Trust_Relationship_Updated: true
    
  - Role_Name: nflo-eks-worker-role
    Previous_ARN: arn:aws:iam::ACCOUNT:role/eks-nflo-NodeInstanceRole
    IRSA_Bindings_Updated: 12
    New_Service_Account_Tokens_Generated: true
    
  - Role_Name: nflo-rds-enhanced-monitoring-role
    Previous_ARN: arn:aws:iam::ACCOUNT:role/rds-enhanced-monitoring-role
    Inline_Policies_Reviewed: 5
    Attached_Policies_Reviewed: 8
    Least_Privilege_Implemented: true
```

### Step 5: Verify Eradication

**Objective:** Confirm the adversary has no remaining access

**Actions:**
1. Re-scan all systems for malware/backdoors
2. Verify all unauthorized access keys are deleted
3. Check for persistence mechanisms (cron jobs, systemd units, etc.)
4. Review AWS Config for unauthorized resource creation
5. Hunt for lateral movement artifacts in logs

**Deliverable: Eradication Verification Report**

```markdown
## Eradication Verification Report

### Malware Scan Results
- **EC2 Instances Scanned:** 127
- **Malware Detected:** 0 ✓
- **Suspicious Files Found:** 0 ✓
- **Scan Timestamp:** 2026-06-10T05:00:00Z
- **Scanner Tool:** CrowdStrike Falcon, OSSEC

### Unauthorized Credentials Verification
- **Access Keys Checked:** 1,247
- **Unauthorized Keys Found:** 0 ✓
- **Keys Rotated:** All
- **Verification Method:** AWS IAM credential report

### Persistence Mechanism Hunt
- **Cron Jobs Reviewed:** 1,892
- **Suspicious Jobs Found:** 0 ✓
- **Systemd Units Reviewed:** 2,147
- **Backdoor Accounts Found:** 0 ✓

### AWS Config Compliance Check
- **Unauthorized IAM Users Created:** 0 ✓
- **Unauthorized Security Groups Created:** 0 ✓
- **Unauthorized S3 Buckets Created:** 0 ✓
- **Configuration Changes Reverted:** All

### Final Eradication Status
✓ All malware removed  
✓ All unauthorized credentials deleted  
✓ No persistence mechanisms detected  
✓ All attacker-created resources removed  
✓ Ready for Recovery phase  
```

## Constraints & Requirements

### NIST SP 800-61 Containment Phase
- Implement short-term containment to limit impact
- Establish escalation procedures for decision-making
- Maintain capability to expand containment if needed
- Document all containment decisions for post-incident review

### Maintaining Business Continuity
- Payment gateway must maintain 99.9% uptime
- No interruption to transaction processing during containment
- Use blue-green deployment for credential rotation
- Implement automatic failover for any system outages

### Regulatory Compliance
- PCI-DSS requires evidence of "secure deletion" of compromised credentials
- Document all changes with timestamps for audit trail
- Maintain separation between forensic and operational systems
- Preserve all logs for potential regulatory review

## Guidance & Tips

### DO ✓
- Follow the NIST Incident Response Life Cycle for consistency
- Use Terraform/Infrastructure as Code for all IAM changes (auditability)
- Test containment actions in staging environment first
- Implement automatic rollback triggers in case of service degradation
- Use AWS Systems Manager Parameter Store for credential rotation orchestration
- Coordinate with legal team before any actions that might affect evidence

### DON'T ✗
- Do NOT delete compromised IAM users immediately; preserve for forensic analysis
- Do NOT perform unplanned RDS reboots that could impact customer transactions
- Do NOT manually edit security groups; use Infrastructure as Code only
- Do NOT skip verification steps—containment must be complete before eradication
- Do NOT ignore staging environment; attackers often maintain backdoors there

## Success Criteria for Phase 2

You have successfully completed Phase 2 when:

✓ All compromised credentials revoked with documented timeline  
✓ Compromised systems isolated without service interruption  
✓ Kubernetes network policies limit lateral movement  
✓ All new service accounts generated and distributed  
✓ Eradication verification confirms zero remaining access  
✓ Payment processing maintains 99.9% uptime throughout  
✓ All containment actions logged and auditable  

---

## Next Steps

Once Phase 2 is complete and eradication is verified, proceed to **Phase 3: Recovery & Hardening** to restore secure operations and implement long-term security improvements.
