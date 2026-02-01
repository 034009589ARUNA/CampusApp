# Forgot Password Implementation Guide

## Current Status ✅

The forgot password feature is **already implemented** with a complete 3-step flow:

1. **Email Step** - User enters email and selects user type
2. **Verify Step** - User enters 6-digit OTP code
3. **Reset Step** - User sets new password

## What's Working ✅

- ✅ Backend API endpoints (forgot-password, verify-reset-code, reset-password)
- ✅ Frontend 3-step flow with progress indicator
- ✅ OTP generation and validation
- ✅ Password reset token system
- ✅ Resend OTP timer (60 seconds)
- ✅ User type selection (Student/Lecturer)
- ✅ Form validation
- ✅ Error handling

## What Needs Implementation 🔧

### 1. **Email Service Integration** (CRITICAL)

**Current State:** OTP is only logged to console in development mode

**What to Implement:**
- Install email service package (nodemailer recommended)
- Configure SMTP settings
- Create email template for password reset
- Send actual email with OTP code

**Files to Create/Update:**
- `CampusTrackBackend/utils/emailService.js` (new)
- `CampusTrackBackend/controllers/authController.js` (update)
- `.env` (add email configuration)

### 2. **Rate Limiting** (SECURITY)

**Current State:** No rate limiting on OTP requests

**What to Implement:**
- Limit OTP requests per email/IP (e.g., 3 requests per hour)
- Prevent brute force attacks on OTP verification
- Add cooldown period after failed attempts

**Files to Create/Update:**
- `CampusTrackBackend/middleware/rateLimiter.js` (new)
- `CampusTrackBackend/routes/auth.js` (update)

### 3. **Enhanced Password Validation** (SECURITY)

**Current State:** Only checks minimum length (6 characters)

**What to Implement:**
- Password strength requirements (uppercase, lowercase, numbers, special chars)
- Password strength indicator in UI
- Check against common passwords

**Files to Update:**
- `CampusTrack/src/screens/Auth/ForgotPassword.tsx` (update)
- `CampusTrackBackend/controllers/authController.js` (update)

### 4. **OTP Auto-fill Support** (UX)

**Current State:** Manual OTP entry

**What to Implement:**
- SMS OTP auto-fill (if using SMS)
- Email OTP auto-fill support
- Better OTP input UX (separate boxes for each digit)

**Files to Update:**
- `CampusTrack/src/screens/Auth/ForgotPassword.tsx` (update)

### 5. **Email Template** (UX)

**Current State:** No email template (if email service is added)

**What to Implement:**
- Professional HTML email template
- Branded email design
- Clear instructions and OTP display

**Files to Create:**
- `CampusTrackBackend/templates/email/passwordReset.html` (new)

## Implementation Steps

### Step 1: Install Email Service

```bash
cd CampusTrackBackend
npm install nodemailer
```

### Step 2: Create Email Service Utility

Create `CampusTrackBackend/utils/emailService.js` with SMTP configuration

### Step 3: Add Rate Limiting

Install express-rate-limit and implement rate limiting middleware

### Step 4: Enhance Password Validation

Add password strength requirements and UI indicators

### Step 5: Improve OTP Input UX

Add separate input boxes for better OTP entry experience

## Environment Variables Needed

Add to `.env`:

```env
# Email Configuration
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
SMTP_FROM=noreply@campustrack.com

# Rate Limiting
OTP_RATE_LIMIT_WINDOW=3600000  # 1 hour in ms
OTP_RATE_LIMIT_MAX=3  # Max 3 requests per hour
```

## Security Best Practices

1. ✅ OTP expires in 10 minutes (already implemented)
2. ✅ Reset token expires in 1 hour (already implemented)
3. ⚠️ Add rate limiting (needs implementation)
4. ⚠️ Add email verification (needs implementation)
5. ✅ Generic error messages (already implemented - security best practice)
6. ⚠️ Password strength requirements (needs enhancement)

## Testing Checklist

- [ ] OTP is sent via email (when email service is configured)
- [ ] OTP expires after 10 minutes
- [ ] Reset token expires after 1 hour
- [ ] Rate limiting prevents abuse
- [ ] Password strength validation works
- [ ] Resend OTP timer works correctly
- [ ] User can complete full flow
- [ ] Error messages are user-friendly
- [ ] Works for both Student and Lecturer accounts

