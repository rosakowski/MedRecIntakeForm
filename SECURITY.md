# Security Documentation

## MedRec Secure Intake - Security Overview

**Version:** 1.0.0  
**Classification:** Internal - For IT Security Review  
**Prepared for:** Hospital IT Security Team  

---

## Executive Summary

MedRec Secure Intake is a zero-storage medication intake system designed to minimize data exposure while enabling patients to submit home medication information to hospital pharmacy departments.

**Key Security Principles:**
- No persistent storage of Protected Health Information (PHI)
- Data exists only in memory during processing
- End-to-end encryption for all data transmission
- Defense in depth with multiple security layers

---

## System Architecture

### Data Flow

```
Patient Input → Browser Validation → HTTPS POST → Server Sanitization 
→ Memory Processing → Email Transmission → Immediate Disposal
```

### Components

| Component | Purpose | Data Handling |
|-----------|---------|---------------|
| Staff Portal (index.html) | Generate QR codes | No PHI handled |
| Patient Form (form.html) | Collect patient data | PHI in browser memory only |
| API Endpoint (api/submit.js) | Process submissions | PHI in memory, never persisted |
| Email Service (Resend) | Deliver to pharmacy | Encrypted transmission |

---

## Security Controls

### 1. Data Protection

#### Zero-Storage Architecture
- **No Database**: No SQL/NoSQL database connections
- **No File Storage**: No filesystem writes containing PHI
- **No Session Storage**: Server maintains no session state
- **Memory-Only Processing**: Data processed and immediately discarded

```javascript
// Example: Data never leaves this scope
async function handler(req, res) {
    const formData = sanitize(req.body);  // Sanitize
    await sendEmail(formData);             // Send
    // formData goes out of scope → garbage collected
    // Nothing persisted anywhere
}
```

#### Input Sanitization
- HTML entity encoding for special characters
- Control character removal
- Length limiting (1000 chars max per field)
- Type validation for all fields

### 2. Access Control

#### Rate Limiting
- 10 submissions per IP per hour
- In-memory tracking (resets on cold start)
- Prevents spam and brute force

#### CORS Policy
- Whitelist of allowed origins
- Preflight request handling
- No wildcard (`*`) origins in production

### 3. Encryption

#### In Transit
- TLS 1.2+ required (enforced by Vercel)
- HTTPS-only API endpoints
- HSTS headers

#### At Rest (Email)
- Resend uses TLS for email transmission
- SOC2 Type II certified infrastructure
- No PHI stored on email servers (pass-through only)

### 4. Security Headers

| Header | Value | Purpose |
|--------|-------|---------|
| X-Content-Type-Options | nosniff | Prevent MIME sniffing |
| X-Frame-Options | DENY | Prevent clickjacking |
| X-XSS-Protection | 1; mode=block | XSS protection |
| Referrer-Policy | strict-origin-when-cross-origin | Limit referrer leakage |
| Content-Security-Policy | default-src 'self'... | Prevent XSS injections |

### 5. Audit & Logging

**What We Log (NO PHI):**
```javascript
{
  timestamp: "2026-02-09T17:30:00Z",
  event: "form_submitted",
  medicationCount: 5,
  ipHash: "192.168.xxx",  // Partial IP
  userAgent: "Mozilla/5.0..."  // Truncated
}
```

**What We DON'T Log:**
- Patient names
- Dates of birth
- Medication names
- Pharmacy information
- Phone numbers
- Any other PHI

---

## Infrastructure Security

### Hosting (Vercel)
- SOC2 Type II certified
- Automatic HTTPS
- DDoS protection
- Edge network with 100+ locations
- No server management required

### Email Provider (Resend)
- SOC2 Type II certified
- HIPAA-eligible infrastructure
- DKIM/SPF/DMARC enabled
- Encrypted email transmission

### Domain Security
- DNSSEC recommended
- DMARC policy: `p=quarantine`
- SPF record for email authentication

---

## Threat Model

### Identified Threats & Mitigations

| Threat | Risk | Mitigation |
|--------|------|------------|
| Data Breach (Server) | Low | Zero-storage architecture |
| Man-in-the-Middle | Low | TLS 1.2+ enforced |
| XSS Injection | Low | Input sanitization, CSP headers |
| SQL Injection | N/A | No database used |
| Session Hijacking | N/A | No session state |
| Rate Limit Bypass | Medium | IP-based limiting |
| Email Interception | Low | TLS-encrypted email |
| Insider Threat | Medium | No data access for developers |

### Attack Scenarios

#### Scenario 1: Server Compromise
**Attack:** Attacker gains access to server infrastructure  
**Impact:** LOW - No PHI stored on server  
**Mitigation:** Zero-storage means no data to steal

#### Scenario 2: Network Sniffing
**Attack:** Attacker intercepts network traffic  
**Impact:** LOW - All traffic encrypted  
**Mitigation:** TLS encryption, HSTS headers

#### Scenario 3: XSS Attack
**Attack:** Malicious script injection  
**Impact:** LOW - Multiple defenses  
**Mitigation:** Input sanitization, CSP headers, output encoding

---

## Compliance Considerations

### HIPAA Technical Safeguards

| Safeguard | Implementation | Status |
|-----------|---------------|--------|
| Access Control | Rate limiting, CORS | ✓ |
| Audit Controls | Metadata-only logging | ✓ |
| Integrity | Input validation | ✓ |
| Person/Entity Authentication | N/A (public form) | N/A |
| Transmission Security | TLS 1.2+ | ✓ |

### Required for Full Compliance

1. **Business Associate Agreement (BAA)**
   - Required between hospital and Resend
   - Required between hospital and Vercel (if not self-hosted)

2. **Risk Assessment**
   - Hospital IT must conduct formal risk assessment
   - Document acceptable use policies

3. **Security Policies**
   - Staff training on QR code handling
   - Incident response procedures
   - Access logging for staff portal

---

## Deployment Checklist

### Pre-Deployment
- [ ] Review code with IT security team
- [ ] Conduct penetration testing
- [ ] Verify all security headers
- [ ] Test rate limiting
- [ ] Validate input sanitization
- [ ] Review Resend BAA requirements

### Deployment
- [ ] Use dedicated domain (not shared)
- [ ] Enable DNSSEC
- [ ] Configure DMARC/SPF/DKIM
- [ ] Set up monitoring/alerts
- [ ] Document incident response

### Post-Deployment
- [ ] Quarterly security reviews
- [ ] Monitor for suspicious activity
- [ ] Keep dependencies updated
- [ ] Review access logs

---

## Incident Response

### Data Breach Response

Since the system stores no PHI, the impact of a breach is minimal. However:

1. **Immediate Actions**
   - Disable form submission
   - Preserve logs
   - Notify IT security team

2. **Investigation**
   - Review access logs
   - Check for unauthorized API calls
   - Verify email delivery logs

3. **Recovery**
   - Rotate API keys
   - Update CORS origins
   - Review and patch code

---

## Contact

For security questions or to report vulnerabilities:

- **Project Lead:** Ross + Astra
- **Hospital IT:** [Your IT Security Contact]

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-09 | Astra | Initial security documentation |