# Password Reset Implementation - Fixes & Verification

## 📋 Patch Summary

### Files Changed (5 files)

1. **`backend/utils/emailService.js`**
   - **Reason**: Enhanced email service to support MAIL_HOST, MAIL_PORT, MAIL_USER, MAIL_PASS env vars; improved error handling; fixed console mode logging format
   - **Changes**:
     - Added support for custom SMTP (MAIL_HOST, MAIL_PORT, MAIL_USER, MAIL_PASS)
     - Enhanced `verifyEmailConfig()` with detailed validation and startup warnings
     - Fixed console mode to log in format: "Reset link for <email>: <url>"
     - Improved error handling with detailed logging for email send failures
     - Added try-catch around `transporter.sendMail()` with detailed error info

2. **`backend/controllers/authController.js`**
   - **Reason**: Added proper error handling for email sending; ensured email is awaited
   - **Changes**:
     - Wrapped `sendPasswordResetEmail()` in try-catch block
     - Added logging for email send failures (without revealing user existence)
     - Ensured email sending is properly awaited

3. **`backend/server.js`**
   - **Reason**: Added email configuration validation on server startup
   - **Changes**:
     - Imported `verifyEmailConfig` from emailService
     - Called `verifyEmailConfig()` on server startup (both server paths)
     - Provides clear warnings if email env vars are missing

4. **`backend/models/User.js`** (No changes - already correct)
   - **Status**: ✅ Fields `passwordResetToken` and `passwordResetExpires` already exist

5. **`backend/routes/authRoutes.js`** (No changes - already correct)
   - **Status**: ✅ Routes `/api/auth/forgot-password` and `/api/auth/reset-password` already configured

---

## 🔧 Environment Variables

### Required for Console Mode (Development)
```env
FRONTEND_URL=http://localhost:5173
```

### Required for Email Sending (Production)

**Option A: Custom SMTP**
```env
MAIL_PROVIDER=nodemailer
MAIL_HOST=smtp.example.com
MAIL_PORT=587
MAIL_USER=your-email@example.com
MAIL_PASS=your-password
MAIL_FROM=your-email@example.com
FRONTEND_URL=http://localhost:5173
```

**Option B: Service-based (Gmail/Outlook)**
```env
MAIL_PROVIDER=nodemailer
MAIL_SERVICE=gmail
MAIL_USER=your-email@gmail.com
MAIL_PASS=your-app-password
MAIL_FROM=your-email@gmail.com
FRONTEND_URL=http://localhost:5173
```

**Note**: `MAIL_PASS` can also be `MAIL_PASSWORD` or `MAIL_API_KEY`

---

## 📝 Server Log Examples

### Console Mode (Development - No Email Config)

**On Server Startup:**
```
================================================================================
⚠️  EMAIL CONFIGURATION WARNING
================================================================================
Missing environment variables: MAIL_HOST, MAIL_PORT, MAIL_USER, MAIL_PASS, MAIL_FROM
Email service will use CONSOLE MODE (reset links logged to server console)

To enable email sending, configure:
  - MAIL_HOST (for custom SMTP) or MAIL_SERVICE (for Gmail/Outlook)
  - MAIL_PORT (for custom SMTP, default: 587)
  - MAIL_USER (your email/username)
  - MAIL_PASS or MAIL_PASSWORD (your email password/app password)
  - MAIL_FROM (sender email address)
  - FRONTEND_URL (frontend URL for reset links)
  - MAIL_PROVIDER=nodemailer (optional, defaults to console)
================================================================================
Email service: Using console mode (emails will be logged to server console)
```

**On Forgot Password Request:**
```
================================================================================
📧 PASSWORD RESET EMAIL (Console Mode)
================================================================================
Reset link for demo_admin@zynga.com: http://localhost:5173/reset-password?token=abc123...&id=507f1f77bcf86cd799439011
================================================================================
⚠️  Copy the Reset URL above and paste it in your browser to reset password!
================================================================================

[INFO] Reset link for demo_admin@zynga.com: http://localhost:5173/reset-password?token=abc123...&id=507f1f77bcf86cd799439011
[INFO] Password reset requested for: demo_admin@zynga.com (Admin), Token hash: 1a2b3c4d..., Expires: 2024-11-11T22:00:00.000Z, IP: ::1
```

### Email Mode (Production - With Email Config)

**On Server Startup:**
```
✅ Email service: Configuration verified
Email service: Nodemailer SMTP transporter initialized (smtp.gmail.com:587)
```

**On Forgot Password Request:**
```
✅ Password reset email sent to demo_admin@zynga.com. MessageId: <message-id>
[INFO] Password reset requested for: demo_admin@zynga.com (Admin), Token hash: 1a2b3c4d..., Expires: 2024-11-11T22:00:00.000Z, IP: ::1
```

**On Email Send Error:**
```
❌ Error sending password reset email: {
  to: 'demo_admin@zynga.com',
  error: 'Invalid login: 535-5.7.8 Username and Password not accepted',
  code: 'EAUTH',
  command: 'AUTH PLAIN',
  response: '535-5.7.8 Username and Password not accepted'
}
[WARN] Failed to send password reset email to demo_admin@zynga.com, but continuing (security: don't reveal)
```

---

## ✅ Manual Test Checklist

### Prerequisites
- [ ] Backend server running on `http://localhost:5000`
- [ ] Frontend server running on `http://localhost:5173`
- [ ] MongoDB connected
- [ ] At least one user exists in database (admin or player)

### Test 1: Forgot Password - Valid Email (Console Mode)

**cURL Command:**
```bash
curl -X POST http://localhost:5000/api/forgot-password \
  -H "Content-Type: application/json" \
  -d '{"email":"demo_admin@zynga.com"}'
```

**Expected Response:**
```json
{
  "message": "If an account with that email exists, a password reset link has been sent."
}
```

**Expected Server Logs:**
```
================================================================================
📧 PASSWORD RESET EMAIL (Console Mode)
================================================================================
Reset link for demo_admin@zynga.com: http://localhost:5173/reset-password?token=<64-char-hex>&id=<userId>
================================================================================
⚠️  Copy the Reset URL above and paste it in your browser to reset password!
================================================================================
```

**Verification:**
- [ ] Response status: 200
- [ ] Generic success message returned
- [ ] Reset URL logged in console with format: "Reset link for <email>: <url>"
- [ ] URL contains `token` (64 hex chars) and `id` (MongoDB ObjectId)

---

### Test 2: Forgot Password - Invalid Email

**cURL Command:**
```bash
curl -X POST http://localhost:5000/api/forgot-password \
  -H "Content-Type: application/json" \
  -d '{"email":"nonexistent@example.com"}'
```

**Expected Response:**
```json
{
  "message": "If an account with that email exists, a password reset link has been sent."
}
```

**Verification:**
- [ ] Response status: 200
- [ ] Same generic message (security: no account enumeration)
- [ ] No reset URL logged (user doesn't exist)

---

### Test 3: Reset Password - Valid Token

**Prerequisites:** Copy reset URL from Test 1

**Extract token and userId from URL:**
- URL format: `http://localhost:5173/reset-password?token=<token>&id=<userId>`
- Extract `token` (64 hex characters)
- Extract `id` (MongoDB ObjectId)

**cURL Command:**
```bash
curl -X POST http://localhost:5000/api/reset-password \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "<userId-from-url>",
    "token": "<token-from-url>",
    "newPassword": "NewPass123"
  }'
```

**Expected Response:**
```json
{
  "message": "Password reset successful. Please login with your new password."
}
```

**Verification:**
- [ ] Response status: 200
- [ ] Success message returned
- [ ] User can login with new password
- [ ] Old password no longer works

---

### Test 4: Reset Password - Invalid Token

**cURL Command:**
```bash
curl -X POST http://localhost:5000/api/reset-password \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "507f1f77bcf86cd799439011",
    "token": "invalidtoken123",
    "newPassword": "NewPass123"
  }'
```

**Expected Response:**
```json
{
  "message": "Invalid or expired reset token"
}
```

**Verification:**
- [ ] Response status: 400
- [ ] Error message returned
- [ ] Password not changed

---

### Test 5: Reset Password - Expired Token

**Steps:**
1. Request password reset
2. Wait for token to expire (or set `PASSWORD_RESET_TOKEN_EXPIRE_MS=60000` for 1 minute)
3. Try to reset with expired token

**cURL Command:**
```bash
curl -X POST http://localhost:5000/api/reset-password \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "<userId>",
    "token": "<expired-token>",
    "newPassword": "NewPass123"
  }'
```

**Expected Response:**
```json
{
  "message": "Invalid or expired reset token"
}
```

**Verification:**
- [ ] Response status: 400
- [ ] Error message returned
- [ ] Expired token cleared from database

---

### Test 6: Reset Password - Invalid Password Format

**cURL Command:**
```bash
curl -X POST http://localhost:5000/api/reset-password \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "<valid-userId>",
    "token": "<valid-token>",
    "newPassword": "weak"
  }'
```

**Expected Response:**
```json
{
  "message": "Password must be at least 6 characters long"
}
```
OR
```json
{
  "message": "Password must contain at least one uppercase letter, one lowercase letter, and one number"
}
```

**Verification:**
- [ ] Response status: 400
- [ ] Appropriate validation error returned
- [ ] Password not changed

---

### Test 7: Rate Limiting - Forgot Password

**cURL Command (Run 6 times):**
```bash
for i in {1..6}; do
  curl -X POST http://localhost:5000/api/forgot-password \
    -H "Content-Type: application/json" \
    -d '{"email":"demo_admin@zynga.com"}'
  echo ""
done
```

**Expected Response (6th request):**
```json
{
  "message": "Too many password reset requests. Please try again later."
}
```

**Verification:**
- [ ] First 5 requests: 200 OK
- [ ] 6th request: 429 Too Many Requests (or rate limit message)
- [ ] Rate limit resets after 1 hour

---

### Test 8: Frontend Integration - Forgot Password

**Steps:**
1. Navigate to `http://localhost:5173/forgot-password`
2. Enter valid email
3. Submit form

**Verification:**
- [ ] Form submits to `/api/forgot-password`
- [ ] Generic success message displayed
- [ ] Redirects to login after 3 seconds
- [ ] Server console shows reset URL

---

### Test 9: Frontend Integration - Reset Password

**Steps:**
1. Get reset URL from Test 1 or Test 8
2. Navigate to reset URL: `http://localhost:5173/reset-password?token=<token>&id=<userId>`
3. Enter new password (meeting requirements)
4. Confirm password
5. Submit form

**Verification:**
- [ ] Page reads `token` and `id` from URL query params
- [ ] Form submits to `/api/reset-password` with `{userId, token, newPassword}`
- [ ] Success message displayed
- [ ] Redirects to login after 2 seconds
- [ ] Can login with new password

---

### Test 10: Email Sending (If Configured)

**Prerequisites:** Configure email env vars (see Environment Variables section)

**cURL Command:**
```bash
curl -X POST http://localhost:5000/api/forgot-password \
  -H "Content-Type: application/json" \
  -d '{"email":"demo_admin@zynga.com"}'
```

**Verification:**
- [ ] Response status: 200
- [ ] Server logs: "✅ Password reset email sent to <email>. MessageId: <id>"
- [ ] Email received in inbox
- [ ] Email contains reset link with token and userId
- [ ] Reset link works (see Test 3)

---

## 🔍 Troubleshooting

### Issue: Reset URL not appearing in console
**Solution:** Check server terminal/console (not browser console). URL is logged to server stdout.

### Issue: "Invalid or expired reset token"
**Possible Causes:**
- Token has expired (default 1 hour)
- Token was already used (one-time use)
- Token format is incorrect (must be 64 hex characters)
- userId is incorrect

**Solution:** Request a new password reset.

### Issue: Email not sending (production)
**Check:**
1. Email env vars are set correctly
2. `MAIL_PASS` is correct (for Gmail, use App Password, not regular password)
3. Server logs for email send errors
4. SMTP server allows connections from your IP

### Issue: Rate limiting too strict
**Solution:** Adjust `forgotPasswordLimiter` in `backend/middleware/rateLimiter.js` (currently 5/hour per IP)

---

## ✅ Verification Checklist

After implementation, verify:

- [x] Server startup validates email config and shows warnings if missing
- [x] Console mode logs reset links in format: "Reset link for <email>: <url>"
- [x] Email sending is properly awaited and errors are caught
- [x] Token generation uses `crypto.randomBytes(32).toString('hex')`
- [x] Tokens are hashed (SHA-256) before storing in database
- [x] Token expiry is configurable via `PASSWORD_RESET_TOKEN_EXPIRE_MS`
- [x] Reset password endpoint validates token hash and expiry
- [x] Frontend correctly reads token and id from URL
- [x] Frontend correctly sends userId, token, newPassword to API
- [x] No raw tokens leaked in responses (only in dev logs if enabled)
- [x] Generic success messages prevent account enumeration
- [x] Rate limiting prevents abuse (5 requests/hour per IP)

---

**Implementation Date**: 2024-11-11
**Status**: ✅ Complete and Ready for Testing

