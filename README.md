# 🏥 MedRec Intake System

**HIPAA-Ready Medication Intake System with Zero Data Storage Architecture**

A secure, serverless medication intake form system designed for hospital IT review and production deployment. Built with security-first principles, HIPAA compliance requirements, and a zero-persistence architecture.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Security Architecture](#security-architecture)
- [Features](#features)
- [Technical Stack](#technical-stack)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Deployment](#deployment)
- [Usage](#usage)
- [Security Features](#security-features)
- [HIPAA Compliance](#hipaa-compliance)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [IT Security Review Checklist](#it-security-review-checklist)

---

## 🎯 Overview

The MedRec Intake System is a secure web application that allows healthcare staff to generate unique QR codes for patients to submit medication intake information. The system is designed with a **zero data storage** architecture—all form data is transmitted directly via secure email and never persisted to disk or database.

### Key Principles

1. **Zero Persistence**: No PHI is written to disk, database, or logs
2. **Direct Transmission**: Form data flows directly from patient → serverless function → email
3. **Ephemeral Processing**: Data exists only in memory during the request lifecycle
4. **Audit Without PHI**: Logging captures metadata only (timestamps, IP hashes, status codes)
5. **Defense in Depth**: Multiple layers of security controls

---

## 🔒 Security Architecture

```
┌─────────────────┐
│   Patient       │
│   Device        │
└────────┬────────┘
         │ HTTPS (TLS 1.3)
         │ Input Sanitized
         ▼
┌─────────────────┐
│   Vercel CDN    │
│   (Edge Network)│
└────────┬────────┘
         │ Rate Limited
         │ CORS Validated
         ▼
┌─────────────────┐
│ Serverless Func │
│ (api/submit.js) │
│                 │
│ - XSS Prevention│
│ - Validation    │
│ - Sanitization  │
└────────┬────────┘
         │ In-Memory Only
         │ No Persistence
         ▼
┌─────────────────┐
│   Resend API    │
│ (HIPAA-Compliant│
│  Infrastructure)│
└────────┬────────┘
         │ Encrypted Email
         ▼
┌─────────────────┐
│   Main Campus   │
│   Pharmacy      │
└─────────────────┘
```

### Data Flow

1. **Staff generates QR code** → Unique session token created (contains no PHI)
2. **Patient scans QR** → Opens form with session context
3. **Patient submits form** → Data sent via HTTPS POST to serverless function
4. **Serverless function** → Validates, sanitizes, and sends email immediately
5. **Email sent** → Data delivered to pharmacy inbox
6. **Memory cleared** → No trace of PHI remains in system

---

## ✨ Features

### For Healthcare Staff
- **QR Code Generation**: Create unique QR codes for each patient session
- **Professional Dashboard**: Clean, intuitive interface for staff workflow
- **Department Tracking**: Associate submissions with specific departments
- **Print & Download**: Export QR codes for physical distribution
- **Activity Log**: Client-side audit trail (no PHI)

### For Patients
- **Mobile-Friendly**: Responsive design works on any device
- **Accessible**: WCAG 2.1 AA compliant form design
- **Clear Instructions**: User-friendly interface with helpful hints
- **Privacy Transparency**: Clear privacy notices and consent mechanisms
- **Secure Submission**: Visual feedback on encryption and security

### Security Features
- ✅ **Zero Data Storage** - No database, no disk writes
- ✅ **Rate Limiting** - 5 submissions per IP per hour (configurable)
- ✅ **Input Sanitization** - XSS prevention via HTML escaping
- ✅ **CORS Protection** - Restricted to allowed origins only
- ✅ **HTTPS Enforcement** - TLS 1.3 with HSTS headers
- ✅ **Audit Logging** - Metadata only, no PHI in logs
- ✅ **CSP Headers** - Content Security Policy prevents injection attacks
- ✅ **Email Encryption** - TLS-encrypted transmission to Resend
- ✅ **Session Isolation** - Each QR code creates a unique, temporary session

---

## 🛠 Technical Stack

### Frontend
- **HTML5** - Semantic, accessible markup
- **CSS3** - Modern, responsive styling (no frameworks required)
- **Vanilla JavaScript** - Zero dependencies, maximum security

### Backend
- **Node.js 18+** - Modern JavaScript runtime
- **Vercel Serverless Functions** - Scalable, ephemeral compute
- **Resend API** - HIPAA-compliant email infrastructure

### Infrastructure
- **Vercel** - Edge network with automatic HTTPS
- **Resend** - Transactional email with BAA available

---

## 📦 Prerequisites

Before you begin, ensure you have:

1. **Node.js 18+** installed
   ```bash
   node --version  # Should be 18.0.0 or higher
   ```

2. **Vercel Account** (free tier works)
   - Sign up at [vercel.com](https://vercel.com)

3. **Resend Account** (free tier: 3,000 emails/month)
   - Sign up at [resend.com](https://resend.com)
   - Verify your sending domain
   - Generate an API key

4. **Domain with Email** (for receiving submissions)
   - Must be able to receive emails at `pharmacy@hospital.org`
   - Or configure a different recipient email

---

## 🚀 Installation

### 1. Clone or Download

```bash
# If using Git
git clone https://github.com/yourusername/medrec-intake-system.git
cd medrec-intake-system

# Or download and extract the ZIP file
```

### 2. Install Dependencies

```bash
npm install
```

This installs:
- `resend` - Email API client
- `vercel` - Deployment CLI (dev dependency)

### 3. Configure Environment

```bash
# Copy the example environment file
cp .env.example .env

# Edit .env and add your actual values
nano .env  # or use your preferred editor
```

**Required environment variables:**

```bash
RESEND_API_KEY=re_your_actual_api_key_here
RECIPIENT_EMAIL=pharmacy@hospital.org
```

**Optional environment variables** (with defaults):

```bash
RATE_LIMIT_MAX=5                    # Max submissions per IP per window
RATE_LIMIT_WINDOW_MS=3600000        # 1 hour in milliseconds
ALLOWED_ORIGINS=https://*.vercel.app # Comma-separated list
```

---

## ⚙️ Configuration

### Resend API Setup

1. **Sign up** at [resend.com](https://resend.com)

2. **Verify your domain**:
   - Go to Domains → Add Domain
   - Add your domain (e.g., `selectmedical.com`)
   - Add the required DNS records (MX, TXT, CNAME)
   - Wait for verification (usually 5-15 minutes)

3. **Generate API Key**:
   - Go to API Keys → Create API Key
   - Name it "MedRec Production"
   - Set permissions to "Sending access"
   - Copy the key (starts with `re_`)
   - **Never commit this key to version control!**

4. **Request BAA** (for HIPAA compliance):
   - Contact Resend support
   - Request a Business Associate Agreement
   - Provide your organization details

### Rate Limiting Configuration

Adjust based on your facility's needs:

```javascript
// Conservative (smaller facility)
RATE_LIMIT_MAX=3
RATE_LIMIT_WINDOW_MS=1800000  // 30 minutes

// Standard (default)
RATE_LIMIT_MAX=5
RATE_LIMIT_WINDOW_MS=3600000  // 1 hour

// Permissive (large facility)
RATE_LIMIT_MAX=10
RATE_LIMIT_WINDOW_MS=3600000  // 1 hour
```

### CORS Configuration

In production, restrict to your actual domain:

```bash
# Single domain
ALLOWED_ORIGINS=https://medrec.selectmedical.com

# Multiple domains
ALLOWED_ORIGINS=https://medrec.hospital.org,https://intake.hospital.org

# Development (allows Vercel preview URLs)
ALLOWED_ORIGINS=https://*.vercel.app
```

---

## 🚢 Deployment

### Option 1: Vercel CLI (Recommended)

```bash
# Install Vercel CLI globally
npm install -g vercel

# Login to your Vercel account
vercel login

# Deploy to production
vercel --prod
```

Follow the prompts:
- **Set up and deploy?** Yes
- **Which scope?** Select your account/team
- **Link to existing project?** No (first time) or Yes (updates)
- **What's your project's name?** medrec-intake-system
- **In which directory is your code located?** ./

### Option 2: Vercel Dashboard

1. Go to [vercel.com/dashboard](https://vercel.com/dashboard)
2. Click **Add New** → **Project**
3. Import your Git repository or upload files
4. Configure:
   - **Framework Preset**: Other
   - **Build Command**: (leave empty)
   - **Output Directory**: (leave empty)
5. Add environment variables:
   - Click **Environment Variables**
   - Add each variable from `.env.example`
   - Set for **Production** environment
6. Click **Deploy**

### Post-Deployment Steps

1. **Verify Deployment**:
   ```bash
   curl -I https://your-project.vercel.app
   ```
   Check for security headers (X-Frame-Options, CSP, etc.)

2. **Test Form Submission**:
   - Open the staff dashboard: `https://your-project.vercel.app`
   - Generate a QR code
   - Scan and submit a test form
   - Verify email arrives at recipient inbox

3. **Configure Custom Domain** (optional):
   - Vercel Dashboard → Project → Settings → Domains
   - Add your custom domain (e.g., `medrec.selectmedical.com`)
   - Update DNS records as instructed
   - Update `ALLOWED_ORIGINS` environment variable

4. **Enable Vercel Analytics** (optional but recommended):
   - Vercel Dashboard → Project → Analytics
   - Enable Web Analytics
   - No PHI is collected (only page views, performance metrics)

---

## 📖 Usage

### For Healthcare Staff

1. **Access Staff Dashboard**:
   ```
   https://your-project.vercel.app/
   ```

2. **Generate QR Code**:
   - Enter patient's MRN (optional)
   - Select department
   - Enter your staff ID
   - Click "Generate Secure QR Code"

3. **Provide to Patient**:
   - **Display on screen**: Show QR code directly to patient
   - **Print**: Click "Print" button to create physical copy
   - **Download**: Save as image to send via secure messaging

4. **Patient Submits Form**:
   - Patient scans QR code with phone camera
   - Opens form in browser
   - Fills out medication information
   - Submits securely

5. **Receive Submission**:
   - Email arrives at `pharmacy@hospital.org`
   - Contains all form data in formatted HTML
   - Reply-to address set to patient's email (if provided)

### For Patients

1. **Scan QR Code**:
   - Use phone camera or QR code reader app
   - Opens form automatically in browser

2. **Complete Form**:
   - Fill out all required fields (marked with *)
   - Review information carefully
   - Check consent checkbox

3. **Submit Securely**:
   - Click "Submit Securely" button
   - Wait for confirmation message
   - Form data sent directly to pharmacy

4. **Confirmation**:
   - Success message appears on screen
   - Optional: Confirmation email sent to patient's address

---

## 🔐 Security Features

### 1. Zero Data Storage

**Implementation**:
- No database connections
- No file system writes
- No session storage
- No cookies containing PHI
- Data exists only in memory during request processing

**Verification**:
```bash
# Check for database connections (should return nothing)
grep -r "database\|mongoose\|sequelize\|knex" api/

# Check for file writes (should return nothing)
grep -r "fs.write\|writeFile" api/
```

### 2. Input Sanitization

**XSS Prevention**:
```javascript
// All user input is HTML-escaped
function escapeHtml(text) {
  return String(text)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;');
}
```

**Validation**:
- Required fields checked
- Email format validated
- Phone numbers formatted
- Date inputs validated

### 3. Rate Limiting

**Implementation**:
- IP-based rate limiting
- 5 submissions per hour (configurable)
- IP addresses hashed before storage
- Rate limit store resets on function cold start

**Testing**:
```bash
# Test rate limit (should block after 5 requests)
for i in {1..7}; do
  curl -X POST https://your-project.vercel.app/api/submit \
    -H "Content-Type: application/json" \
    -d '{"patientName":"Test","dateOfBirth":"1990-01-01","medicationName":"Test","dosage":"10mg"}'
done
```

### 4. HTTPS Enforcement

**Headers**:
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

**Vercel Configuration**:
- Automatic HTTPS on all domains
- TLS 1.3 enabled by default
- HTTP → HTTPS redirect automatic

### 5. Security Headers

**Implemented Headers**:
```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin
Content-Security-Policy: default-src 'none'; frame-ancestors 'none'
Permissions-Policy: geolocation=(), microphone=(), camera=()
```

**Testing**:
```bash
curl -I https://your-project.vercel.app/api/submit
```

### 6. Audit Logging

**What's Logged** (safe):
- Timestamp of submission
- Hashed IP address
- Request ID
- HTTP status code
- User agent (truncated)
- Email delivery status

**What's NOT Logged** (PHI):
- Patient names
- Medical record numbers
- Medication names
- Any form field values

**Example Log Entry**:
```json
{
  "level": "INFO",
  "event": "FORM_SUBMITTED",
  "requestId": "req_1234567890_abc123",
  "timestamp": "2025-01-15T10:30:45.123Z",
  "ipHash": "a1b2c3d4",
  "userAgent": "Mozilla/5.0 (iPhone; CPU iPhone OS 15_0 like Mac O",
  "emailId": "resend_email_id_here",
  "remaining": 4
}
```

---

## 🏥 HIPAA Compliance

### Technical Safeguards

✅ **Access Control** (§164.312(a)(1)):
- HTTPS enforcement prevents unauthorized access
- Rate limiting prevents brute force attacks
- CORS restrictions limit access to authorized origins

✅ **Audit Controls** (§164.312(b)):
- All API requests logged with metadata
- No PHI in logs (compliant with minimum necessary)
- Logs available via Vercel dashboard

✅ **Integrity** (§164.312(c)(1)):
- Input sanitization prevents data corruption
- Validation ensures data accuracy
- Email format maintains data integrity

✅ **Transmission Security** (§164.312(e)(1)):
- TLS 1.3 encryption for all transmissions
- HSTS header enforces secure connections
- Resend uses encrypted SMTP for email delivery

### Administrative Safeguards

📋 **Required Actions**:

1. **Business Associate Agreement (BAA)**:
   - Sign BAA with Resend (your email provider)
   - Sign BAA with Vercel (if processing PHI on platform)
   - Maintain copies of all BAAs

2. **Security Risk Analysis**:
   - Document all technical controls (this README)
   - Identify potential vulnerabilities
   - Implement mitigation strategies
   - Review annually

3. **Workforce Training**:
   - Train staff on proper QR code generation
   - Train staff on recognizing phishing attempts
   - Train staff on incident reporting procedures

4. **Incident Response Plan**:
   - Define what constitutes a breach
   - Establish reporting procedures
   - Document incident response steps
   - Test plan annually

### Physical Safeguards

🏢 **Recommendations**:

1. **Workstation Security**:
   - Lock workstations when generating QR codes
   - Use privacy screens in public areas
   - Secure printed QR codes immediately

2. **Device Security**:
   - Require device passwords/biometrics
   - Use Mobile Device Management (MDM) if applicable
   - Encrypt devices storing QR code images

### Documentation Requirements

Create and maintain these documents:

1. **System Security Plan** - Overall security architecture
2. **Risk Analysis** - Identified risks and mitigations
3. **Policies and Procedures** - Use of the system
4. **Training Records** - Who was trained and when
5. **Incident Log** - Any security incidents
6. **BAAs** - Business associate agreements

---

## 🧪 Testing

### Unit Testing (Manual)

#### Test 1: Form Submission
```bash
curl -X POST https://your-project.vercel.app/api/submit \
  -H "Content-Type: application/json" \
  -d '{
    "patientName": "John Doe",
    "dateOfBirth": "1980-01-15",
    "medicationName": "Lisinopril",
    "dosage": "10mg",
    "mrn": "MRN123456",
    "phone": "(555) 123-4567",
    "frequency": "once-daily"
  }'
```

**Expected Result**:
- HTTP 200 status
- JSON response: `{"success": true, ...}`
- Email received at recipient address

#### Test 2: Missing Required Fields
```bash
curl -X POST https://your-project.vercel.app/api/submit \
  -H "Content-Type: application/json" \
  -d '{
    "patientName": "John Doe"
  }'
```

**Expected Result**:
- HTTP 400 status
- Error message listing missing fields

#### Test 3: Rate Limiting
```bash
# Run this script to exceed rate limit
for i in {1..7}; do
  echo "Request $i:"
  curl -X POST https://your-project.vercel.app/api/submit \
    -H "Content-Type: application/json" \
    -d '{"patientName":"Test","dateOfBirth":"1990-01-01","medicationName":"Test","dosage":"10mg"}'
  echo -e "\n"
done
```

**Expected Result**:
- First 5 requests: HTTP 200
- Requests 6+: HTTP 429 (Too Many Requests)

#### Test 4: XSS Prevention
```bash
curl -X POST https://your-project.vercel.app/api/submit \
  -H "Content-Type: application/json" \
  -d '{
    "patientName": "<script>alert(\"XSS\")</script>",
    "dateOfBirth": "1990-01-01",
    "medicationName": "Test",
    "dosage": "10mg"
  }'
```

**Expected Result**:
- HTTP 200 status
- Email received with escaped HTML: `&lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;`

#### Test 5: CORS Validation
```bash
curl -X POST https://your-project.vercel.app/api/submit \
  -H "Content-Type: application/json" \
  -H "Origin: https://malicious-site.com" \
  -d '{
    "patientName": "Test",
    "dateOfBirth": "1990-01-01",
    "medicationName": "Test",
    "dosage": "10mg"
  }'
```

**Expected Result**:
- HTTP 403 status
- Error: "Access denied from this origin"

### Integration Testing

1. **End-to-End Flow**:
   - Staff generates QR code
   - Scan with mobile device
   - Submit form
   - Verify email received
   - Check email formatting
   - Verify no PHI in logs

2. **Browser Compatibility**:
   - Test on Chrome, Firefox, Safari, Edge
   - Test on iOS Safari and Android Chrome
   - Test with various screen sizes

3. **Accessibility Testing**:
   - Run axe DevTools
   - Test with screen reader
   - Test keyboard navigation
   - Verify color contrast ratios

### Security Testing

```bash
# Run security headers check
curl -I https://your-project.vercel.app | grep -E "X-|Content-Security|Strict-Transport"

# Check for exposed environment variables
curl https://your-project.vercel.app/.env
# Should return 404

# Check for directory listing
curl https://your-project.vercel.app/api/
# Should return 404 or method not allowed
```

---

## 🔧 Troubleshooting

### Issue: "Failed to process submission"

**Possible Causes**:
1. Resend API key not set or invalid
2. Recipient email not verified in Resend
3. Rate limit exceeded

**Solutions**:
```bash
# Check environment variables
vercel env ls

# Verify Resend API key is set
vercel env pull

# Check Resend dashboard for API key status
# Verify domain is verified in Resend
```

### Issue: "CORS violation" or 403 error

**Cause**: Request origin not in allowed list

**Solution**:
```bash
# Update ALLOWED_ORIGINS environment variable
vercel env add ALLOWED_ORIGINS

# Enter your domain:
# https://your-actual-domain.com

# Redeploy
vercel --prod
```

### Issue: Email not received

**Checks**:
1. Check spam/junk folder
2. Verify domain is verified in Resend dashboard
3. Check Resend logs for delivery status
4. Verify recipient email address is correct

**Debugging**:
```bash
# Check Vercel logs
vercel logs

# Look for email delivery confirmation
# Should see: "emailId": "resend_..."
```

### Issue: Rate limit too restrictive

**Solution**:
```bash
# Increase rate limit
vercel env add RATE_LIMIT_MAX
# Enter new value (e.g., 10)

vercel env add RATE_LIMIT_WINDOW_MS
# Enter new window (e.g., 7200000 for 2 hours)

# Redeploy
vercel --prod
```

### Issue: QR code not working on older phones

**Cause**: URL too long for some QR readers

**Solution**:
- Shorten session token generation
- Use a URL shortener
- Provide manual link entry option

---

## ✅ IT Security Review Checklist

Use this checklist when presenting to your IT security team:

### Architecture & Design
- [ ] Zero data storage architecture explained
- [ ] Data flow diagram provided
- [ ] Threat model documented
- [ ] Attack surface minimized (no database, no file system)

### Authentication & Access Control
- [ ] HTTPS enforced on all endpoints
- [ ] CORS restrictions in place
- [ ] Rate limiting configured
- [ ] Session tokens are opaque (no PHI)

### Data Protection
- [ ] Input sanitization implemented (XSS prevention)
- [ ] Output encoding verified
- [ ] TLS 1.3 for all transmissions
- [ ] No PHI in logs or error messages

### Audit & Monitoring
- [ ] Audit logging implemented
- [ ] Logs contain only non-PHI metadata
- [ ] Request IDs for traceability
- [ ] Vercel analytics enabled (optional)

### HIPAA Compliance
- [ ] BAA with Resend obtained
- [ ] BAA with Vercel obtained (if required)
- [ ] Technical safeguards documented
- [ ] Administrative procedures defined
- [ ] Security risk analysis completed

### Incident Response
- [ ] Breach notification procedures defined
- [ ] Incident response plan created
- [ ] Contact information for support
- [ ] Rollback procedures documented

### Code Quality
- [ ] Comprehensive inline comments
- [ ] Security decisions explained
- [ ] Dependencies minimal and justified
- [ ] No known vulnerabilities (run `npm audit`)

### Testing
- [ ] Manual testing completed
- [ ] Rate limiting verified
- [ ] XSS prevention tested
- [ ] CORS validation tested
- [ ] End-to-end flow verified

### Documentation
- [ ] README.md complete
- [ ] Deployment guide provided
- [ ] Troubleshooting section included
- [ ] Security features documented

### Deployment
- [ ] Environment variables secured
- [ ] Production domain configured
- [ ] DNS records properly set
- [ ] Backup/rollback plan defined

---

## 📞 Support

### For Technical Issues

- **Vercel Support**: https://vercel.com/support
- **Resend Support**: support@resend.com
- **Documentation**: This README

### For Security Concerns

Contact your organization's IT Security team immediately if you:
- Suspect a data breach
- Discover a vulnerability
- Experience unusual system behavior
- Need to report an incident

### For Feature Requests

This system is designed for simplicity and security. Additional features should be carefully evaluated for:
- Impact on zero-storage architecture
- HIPAA compliance implications
- Security risk introduction
- Complexity vs. benefit tradeoff

---

## 📄 License

**UNLICENSED** - This software is proprietary to the Hospital System and is not licensed for external use, modification, or distribution.

---

## 🔄 Version History

### v1.0.0 (2025-01-15)
- Initial release
- Staff dashboard with QR generation
- Patient form with validation
- Serverless email submission
- Rate limiting and security headers
- Comprehensive documentation
- HIPAA-ready architecture

---

## 🙏 Acknowledgments

Built with security-first principles for healthcare environments. Designed to meet HIPAA technical safeguard requirements while maintaining simplicity and usability.

**Remember**: Security is not a feature—it's a requirement. This system is only as secure as your operational procedures. Train staff properly, maintain documentation, and regularly review security controls.

---

**Last Updated**: 2025-01-15  
**Maintained By**: Hospital IT Security Team  
**Review Date**: [To be scheduled annually]