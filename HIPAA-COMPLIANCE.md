# HIPAA Compliance Checklist

## MedRec Secure Intake - Path to Full HIPAA Compliance

This document outlines the steps required to bring the MedRec Secure Intake system into full HIPAA compliance for production use in a healthcare setting.

---

## Phase 1: Legal & Administrative (CRITICAL)

### 1.1 Business Associate Agreements (BAAs)

A BAA is a **legal requirement** under HIPAA when a Business Associate (BA) handles PHI on behalf of a Covered Entity (CE).

**Required BAAs:**

| Vendor | BAA Available | Action Required |
|--------|--------------|-----------------|
| Resend (Email) | ✅ Yes | Sign BAA before production |
| Vercel (Hosting) | ✅ Yes | Sign BAA or self-host |

**How to Request a BAA:**

**Resend:**
1. Contact support@resend.com
2. Request HIPAA BAA for enterprise account
3. May require upgrade to paid plan

**Vercel:**
1. Contact enterprise@vercel.com
2. Request HIPAA BAA
3. Requires Enterprise plan ($$$)

**Alternative:** Self-host on hospital-owned infrastructure (no BAA needed for Vercel)

### 1.2 Covered Entity Authorization

- [ ] Hospital Legal reviews and approves system
- [ ] Hospital IT Security approves architecture
- [ ] Pharmacy Department head approves workflow
- [ ] Document approval in compliance file

### 1.3 Risk Assessment

Conduct a formal HIPAA Security Risk Assessment (SRA):

- [ ] Identify all PHI touchpoints
- [ ] Assess threats and vulnerabilities
- [ ] Document current safeguards
- [ ] Identify gaps
- [ ] Create remediation plan
- [ ] Document risk acceptance (for residual risks)

**Tools:**
- HHS Security Risk Assessment Tool: https://www.healthit.gov/topic/privacy-security-and-hipaa/security-risk-assessment

---

## Phase 2: Technical Safeguards

### 2.1 Access Control (§ 164.312(a))

| Requirement | Current Status | Action Needed |
|-------------|---------------|---------------|
| Unique user identification | ⚠️ Partial | Add staff authentication |
| Emergency access procedure | ❌ Missing | Document emergency access |
| Automatic logoff | ❌ Missing | Implement session timeout |
| Encryption and decryption | ✅ Yes | TLS 1.2+ in place |

**Actions:**
- [ ] Add staff login to portal (currently no auth)
- [ ] Implement session timeout (15 min idle)
- [ ] Document emergency access procedures
- [ ] Add audit logging for staff actions

### 2.2 Audit Controls (§ 164.312(b))

| Requirement | Current Status | Action Needed |
|-------------|---------------|---------------|
| Record activity | ⚠️ Partial | Log staff actions |
| Hardware/software procedures | ❌ Missing | Document procedures |

**Actions:**
- [ ] Log all QR code generations (timestamp, staff ID)
- [ ] Log all form submissions (metadata only)
- [ ] Create audit log retention policy (6 years)
- [ ] Document audit review procedures

### 2.3 Integrity Controls (§ 164.312(c))

| Requirement | Current Status | Action Needed |
|-------------|---------------|---------------|
| Mechanisms to authenticate PHI | ✅ Yes | Input validation in place |
| Mechanisms to protect against alteration | ✅ Yes | No persistent storage |

### 2.4 Person/Entity Authentication (§ 164.312(d))

| Requirement | Current Status | Action Needed |
|-------------|---------------|---------------|
| Verify person/entity seeking access | ⚠️ Partial | Add staff authentication |

**Actions:**
- [ ] Implement staff authentication for portal
- [ ] Document identity verification procedures

### 2.5 Transmission Security (§ 164.312(e))

| Requirement | Current Status | Action Needed |
|-------------|---------------|---------------|
| Integrity controls | ✅ Yes | TLS ensures integrity |
| Encryption | ✅ Yes | TLS 1.2+ enforced |

---

## Phase 3: Physical Safeguards

### 3.1 Facility Access Controls

Since this is a web application:

- [ ] Document that server access is controlled by Vercel/Resend
- [ ] Verify Vercel/Resend physical security (SOC2 reports)
- [ ] Document workstation security for staff using portal

### 3.2 Workstation Security

- [ ] Policy: Lock screens when away
- [ ] Policy: No PHI on printed QR codes visible to others
- [ ] Policy: Secure disposal of printed QR codes

### 3.3 Device and Media Controls

- [ ] Document that no PHI is stored on devices
- [ ] Policy for secure disposal of devices (if applicable)

---

## Phase 4: Administrative Safeguards

### 4.1 Security Management Process

- [ ] Designate Security Officer
- [ ] Document security policies
- [ ] Conduct annual risk assessments
- [ ] Sanction policy for violations
- [ ] Information system activity review

### 4.2 Assigned Security Responsibilities

- [ ] Name Security Officer: _______________
- [ ] Name Privacy Officer: _______________
- [ ] Document roles and responsibilities

### 4.3 Workforce Security

- [ ] Authorization procedures for staff access
- [ ] Clearance procedures (background checks)
- [ ] Termination procedures (access revocation)

### 4.4 Information Access Management

- [ ] Access authorization procedures
- [ ] Access establishment/modification
- [ ] Access termination

### 4.5 Security Awareness and Training

- [ ] Security reminders
- [ ] Protection from malicious software
- [ ] Log-in monitoring
- [ ] Password management

**Training Topics:**
- How to generate QR codes
- How to verify patient identity before giving QR
- What to do if patient has trouble
- Security incident reporting

### 4.6 Security Incident Procedures

Create incident response plan:

- [ ] Reporting procedures
- [ ] Response procedures
- [ ] Mitigation procedures

### 4.7 Contingency Plan

- [ ] Data backup plan (N/A - no data stored)
- [ ] Disaster recovery plan
- [ ] Emergency mode operation plan
- [ ] Testing and revision procedures
- [ ] Applications and data criticality analysis

### 4.8 Evaluation

- [ ] Perform periodic technical evaluations
- [ ] Review system activity
- [ ] Test security measures

### 4.9 Business Associate Contracts

- [ ] BAA with Resend (email)
- [ ] BAA with Vercel (if applicable)
- [ ] Review BA compliance annually

---

## Phase 5: Documentation

### Required Policies and Procedures

Create the following documents:

- [ ] Security Policies and Procedures
- [ ] Privacy Policies
- [ ] Incident Response Plan
- [ ] Disaster Recovery Plan
- [ ] Workforce Training Program
- [ ] Risk Assessment Report
- [ ] Business Associate Agreements
- [ ] System Security Plan
- [ ] Configuration Management Plan
- [ ] Audit Log Review Procedures

---

## Phase 6: Testing & Validation

### 6.1 Security Testing

- [ ] Penetration testing
- [ ] Vulnerability scanning
- [ ] Code review
- [ ] Third-party security audit (optional but recommended)

### 6.2 Functional Testing

- [ ] Test form submission end-to-end
- [ ] Test email delivery
- [ ] Test QR code generation
- [ ] Test expiration timers
- [ ] Test rate limiting
- [ ] Test input validation

### 6.3 User Acceptance Testing

- [ ] Pharmacy staff test workflow
- [ ] IT staff test admin functions
- [ ] Document feedback and changes

---

## Phase 7: Deployment

### Pre-Production

- [ ] All BAAs signed
- [ ] Risk assessment completed
- [ ] Policies documented
- [ ] Security testing passed
- [ ] Training completed
- [ ] Incident response plan ready

### Go-Live

- [ ] Soft launch (pilot with small team)
- [ ] Monitor logs for issues
- [ ] Gather feedback
- [ ] Adjust as needed
- [ ] Full rollout

---

## Maintenance

### Ongoing Requirements

| Task | Frequency | Responsible |
|------|-----------|-------------|
| Review audit logs | Weekly | Security Officer |
| Update policies | Annually | Privacy Officer |
| Risk assessment | Annually | Security Officer |
| Security testing | Annually | IT Security |
| BAA review | Annually | Compliance |
| Training | Annually + new hires | HR |
| Vulnerability scans | Quarterly | IT Security |
| Access review | Quarterly | Manager |

---

## Quick Reference: Compliance Status

| Requirement | Status | Priority |
|-------------|--------|----------|
| Resend BAA | ⏳ Needed | CRITICAL |
| Vercel BAA or Self-host | ⏳ Needed | CRITICAL |
| Risk Assessment | ⏳ Needed | CRITICAL |
| Staff Authentication | ⚠️ Partial | HIGH |
| Audit Logging | ✅ Metadata only | MEDIUM |
| Encryption | ✅ Yes | - |
| Access Controls | ⚠️ Partial | HIGH |
| Policies | ⏳ Needed | HIGH |
| Training | ⏳ Needed | MEDIUM |
| Incident Response | ⏳ Needed | MEDIUM |

---

## Cost Estimate

| Item | Cost | Notes |
|------|------|-------|
| Resend BAA | ~$20-100/mo | Paid plan required |
| Vercel Enterprise | ~$2000+/mo | Or self-host for free |
| Self-Hosting | Free | Requires server |
| Security Audit | $5,000-20,000 | One-time |
| Policy Development | Internal labor | 20-40 hours |
| Training | Internal labor | 4 hours/staff |

**Budget Option:** Self-host on hospital infrastructure → Cost: $0 + labor

---

## Timeline Estimate

| Phase | Duration | Dependencies |
|-------|----------|--------------|
| BAAs | 1-2 weeks | Legal review |
| Risk Assessment | 2-3 weeks | BAAs signed |
| Policy Development | 2-4 weeks | Risk assessment |
| Technical Changes | 1-2 weeks | Requirements defined |
| Testing | 1-2 weeks | Code complete |
| Training | 1 week | Policies final |
| **Total** | **8-14 weeks** | - |

---

## Summary

**Current State:** System implements strong technical safeguards but lacks administrative and legal components.

**Path to Compliance:**
1. Sign BAAs (Resend required, Vercel optional if self-hosted)
2. Conduct formal risk assessment
3. Develop policies and procedures
4. Add staff authentication
5. Train workforce
6. Deploy with monitoring

**Lowest-Cost Option:** Self-host on hospital infrastructure, sign Resend BAA only.

---

**Document Owner:** Hospital Compliance Officer  
**Last Updated:** 2026-02-09  
**Next Review:** [Date + 1 year]