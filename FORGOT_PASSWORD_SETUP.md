# Forgot Password Setup Instructions

## Quick Setup Guide

### Step 1: Install Dependencies

```bash
cd CampusTrackBackend
npm install nodemailer express-rate-limit
```

### Step 2: Configure Environment Variables

Add the following to your `.env` file in `CampusTrackBackend`:

```env
# Email Configuration (Required for production)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
SMTP_FROM=noreply@campustrack.com

# Rate Limiting Configuration (Optional - defaults provided)
OTP_RATE_LIMIT_WINDOW=3600000  # 1 hour in milliseconds
OTP_RATE_LIMIT_MAX=3  # Maximum 3 OTP requests per hour

# Development Settings (Optional)
SKIP_RATE_LIMIT=false  # Set to 'true' to skip rate limiting in development
```

### Step 3: Gmail Setup (If using Gmail)

1. Enable 2-Step Verification on your Google account
2. Generate an App Password:
   - Go to Google Account → Security → 2-Step Verification → App passwords
   - Create a new app password for "Mail"
   - Use this password in `SMTP_PASS`

### Step 4: Test Email Service

The email service will automatically:
- ✅ Send emails in production (when SMTP is configured)
- ✅ Log OTP to console in development (when SMTP is not configured)
- ✅ Fall back gracefully if email sending fails

### Step 5: Verify Implementation

1. **Test OTP Request:**
   ```bash
   # Should send email (or log to console in dev)
   POST /api/auth/forgot-password
   Body: { "email": "test@example.com", "userType": "student" }
   ```

2. **Test Rate Limiting:**
   - Try requesting OTP more than 3 times in an hour
   - Should receive rate limit error

3. **Test Full Flow:**
   - Request OTP → Verify OTP → Reset Password
   - Should complete successfully

## Features Implemented

### ✅ Email Service
- Professional HTML email template
- Fallback to console logging in development
- Success notification email after password reset

### ✅ Rate Limiting
- 3 OTP requests per hour per IP
- 5 OTP verification attempts per 15 minutes
- 3 password reset attempts per hour

### ✅ Password Strength Indicator
- Visual strength bar (Weak/Medium/Strong)
- Color-coded feedback
- Real-time validation

### ✅ Security Features
- OTP expires in 10 minutes
- Reset token expires in 1 hour
- Generic error messages (security best practice)
- Account deactivation check

## Troubleshooting

### Email Not Sending

1. **Check SMTP Configuration:**
   - Verify all SMTP_* variables are set correctly
   - Test SMTP credentials

2. **Check Console Logs:**
   - OTP will be logged to console if email fails
   - Check for error messages

3. **Gmail Issues:**
   - Ensure App Password is used (not regular password)
   - Check if "Less secure app access" is enabled (if not using App Password)

### Rate Limiting Too Strict

- Adjust `OTP_RATE_LIMIT_MAX` in `.env`
- Set `SKIP_RATE_LIMIT=true` for development

### OTP Not Received

1. Check spam folder
2. Verify email address is correct
3. Check console logs (development mode)
4. Verify SMTP configuration

## Production Checklist

- [ ] SMTP credentials configured
- [ ] Rate limiting enabled
- [ ] Email template customized with branding
- [ ] Tested full password reset flow
- [ ] Monitored email delivery rates
- [ ] Set up email service monitoring
- [ ] Configured email bounce handling

## Alternative Email Services

You can use other email services by updating SMTP settings:

### SendGrid
```env
SMTP_HOST=smtp.sendgrid.net
SMTP_PORT=587
SMTP_USER=apikey
SMTP_PASS=your-sendgrid-api-key
```

### Mailgun
```env
SMTP_HOST=smtp.mailgun.org
SMTP_PORT=587
SMTP_USER=your-mailgun-username
SMTP_PASS=your-mailgun-password
```

### AWS SES
```env
SMTP_HOST=email-smtp.us-east-1.amazonaws.com
SMTP_PORT=587
SMTP_USER=your-aws-access-key
SMTP_PASS=your-aws-secret-key
```

