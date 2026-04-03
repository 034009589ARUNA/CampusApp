# Deployment & Testing Checklist - Profile Image Persistence Fix

## ✅ Backend Changes Completed

- [x] Updated `authController.js` - Login endpoint to return profileImage
- [x] Updated `authController.js` - Student signup to return profileImage
- [x] Updated `authController.js` - Lecturer signup to return profileImage
- [x] Added logging for debugging

**Files Changed:**
- `CampusTrackBackend/controllers/authController.js` (3 functions)

**No Database Migration Needed**: ✅ All fields already exist in schema

---

## 🚀 Deployment Steps

### Step 1: Verify Files Are Updated
```bash
cd CampusTrackBackend
grep -n "profileImage: user.profileImage" controllers/authController.js
# Should find: Line ~214 (in login function)
```

### Step 2: Restart Backend Server
```bash
# Option A: If using npm/nodemon
npm restart
# or
npm stop
npm start

# Option B: If using PM2
pm2 restart app.js

# Option C: Docker
docker restart campustrack-backend
```

### Step 3: Verify Backend Is Running
```bash
# Test endpoint
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "password",
    "userType": "student"
  }'

# Should see "profileImage" in response
```

---

## 🧪 Testing Scenarios

### Test Case 1: Upload → Logout → Login (CRITICAL)
**Expected**: Profile image persists

**Steps:**
1. [ ] Open CampusTrack app
2. [ ] Login with valid credentials
3. [ ] Navigate to Dashboard
4. [ ] Tap on profile picture button
5. [ ] Choose "Choose from Gallery"
6. [ ] Select any image and upload
7. [ ] See success message "Profile picture updated successfully!"
8. [ ] Verify image appears in profile button
9. [ ] Go to Settings/Profile
10. [ ] Tap "Logout"
11. [ ] Confirm logout
12. [ ] Login again with same credentials
13. [ ] Navigate to Dashboard
14. [ ] Verify profile image is still visible ✅

**What to Check:**
- [ ] Backend logs: `✅ Login successful for user: ... with profileImage: yes`
- [ ] App logs: `🔄 Dashboard focus effect - image refreshed: http://...`
- [ ] Profile picture displays immediately (not blank)

---

### Test Case 2: Cross-Device Login (IMPORTANT)
**Expected**: Profile image available on new device

**Setup:**
- Device A: CampusTrack already installed
- Device B: CampusTrack needs to be installed
- Same account for both devices

**Steps:**
1. [ ] Device A: Login and upload profile image
2. [ ] Device A: Verify image appears
3. [ ] Device A: Do NOT logout yet
4. [ ] Device B: Install CampusTrack
5. [ ] Device B: Login with SAME email as Device A
6. [ ] Device B: Navigate to Dashboard
7. [ ] Device B: Verify profile image appears ✅
8. [ ] Device B: Profile image should be identical to Device A

**What to Check:**
- [ ] Backend returns image URL for both logins
- [ ] Image URLs are identical (proof from same database)
- [ ] Both devices display same image

---

### Test Case 3: App Restart with Cached Session
**Expected**: Image available even if offline

**Steps:**
1. [ ] Login to app
2. [ ] Upload profile image
3. [ ] Navigate to Dashboard (verify image shows)
4. [ ] Close app completely (swipe away)
5. [ ] Turn off WiFi/Mobile Data (go offline)
6. [ ] Reopen app
7. [ ] Navigate to Dashboard
8. [ ] Verify image still displays from cache ✅
9. [ ] Turn WiFi/Data back on
10. [ ] Image should refresh from server

**What to Check:**
- [ ] Image loads instantly from cache
- [ ] No blank profile picture
- [ ] AsyncStorage properly caching user data

---

### Test Case 4: Multiple Users on Same Device
**Expected**: Each user's image persists independently

**Setup:**
- Account A: student@example.com
- Account B: another@example.com

**Steps:**
1. [ ] Login as Account A
2. [ ] Upload profile image (Image A)
3. [ ] Verify displays
4. [ ] Logout
5. [ ] Login as Account B
6. [ ] Upload profile image (Image B - different)
7. [ ] Verify Image B displays
8. [ ] Logout
9. [ ] Login as Account A
10. [ ] Verify Image A displays again ✅ (not Image B)
11. [ ] Logout
12. [ ] Login as Account B
13. [ ] Verify Image B displays again ✅

**What to Check:**
- [ ] Each account has separate AsyncStorage entry
- [ ] No image mixing between accounts
- [ ] Each account retrieves correct image on re-login

---

### Test Case 5: Verify API Response
**Expected**: Backend returns profileImage field

**Setup:**
- Terminal/Postman access
- Valid test account credentials

**Steps:**
1. [ ] Open Postman or Terminal
2. [ ] Create POST request to `http://localhost:5000/api/auth/login`
3. [ ] Headers: `Content-Type: application/json`
4. [ ] Body:
   ```json
   {
     "email": "student@example.com",
     "password": "password123",
     "userType": "student"
   }
   ```
5. [ ] Send request
6. [ ] Check response JSON:
   ```json
   {
     "success": true,
     "message": "Login successful",
     "token": "...",
     "user": {
       "id": "507f...",
       "profileImage": "http://localhost:5000/uploads/profile_507f.jpg",  ✅ MUST BE HERE
       "phoneNumber": "...",
       "address": "...",
       "fullName": "...",
       ...
     }
   }
   ```
7. [ ] Verify `profileImage` field exists
8. [ ] Verify value is URL string or null (not undefined)

**What to Check:**
- [ ] `profileImage` field is present
- [ ] Value is either valid URL or null (not undefined)
- [ ] URL format is correct: `http://...`
- [ ] Other fields (phoneNumber, address, etc.) also present

---

### Test Case 6: No Image Upload Yet
**Expected**: Handles null gracefully

**Steps:**
1. [ ] Create new test account (no image uploaded yet)
2. [ ] Login with new account
3. [ ] Navigate to Dashboard
4. [ ] Verify app doesn't crash
5. [ ] Verify profile picture shows placeholder/default ✅
6. [ ] Upload image now
7. [ ] Verify image displays
8. [ ] Logout/Login again
9. [ ] Verify image persists

**What to Check:**
- [ ] App handles `profileImage: null` gracefully
- [ ] No crashes or console errors
- [ ] Placeholder/default image shown
- [ ] Can still upload image after

---

## 📊 Verification Checklist

### Backend Verification
- [ ] File `authController.js` has been updated with git diff
- [ ] Backend server restarted successfully
- [ ] No error messages on backend startup
- [ ] Backend responding to requests (test with curl)

### Frontend Verification
- [ ] App connects to updated backend
- [ ] No console errors on login
- [ ] Profile image field properly handled
- [ ] Images load from correct URLs

### Database Verification
- [ ] MongoDB has profile images stored (if applicable)
- [ ] User records have profileImage field
- [ ] Query: `db.students.findOne({email:"..."}).profileImage` returns URL

### Logging Verification
- [ ] Backend logs show: `✅ Login successful for user: ... with profileImage: yes`
- [ ] Frontend logs show: `🔄 Dashboard focus effect - image refreshed: ...`

---

## 🔧 Troubleshooting

### Issue: Profile image still disappears after logout
**Diagnostics:**
- [ ] Check backend logs for profileImage in response
- [ ] Run test case 5 (API response test)
- [ ] Verify backend was restarted (not just code edited)
- [ ] Check AsyncStorage isn't cleared by user manually

**Solution:**
1. Verify authController.js has the changes
2. Restart backend: `npm stop && npm start`
3. Clear app cache: Settings → Apps → CampusTrack → Clear Data
4. Test again

### Issue: Image URL not loading
**Diagnostics:**
- [ ] Check if image file exists: `ls CampusTrackBackend/uploads/`
- [ ] Verify URL format in response
- [ ] Test URL in browser: `http://localhost:5000/uploads/profile_xxx.jpg`

**Solution:**
1. Check uploads folder has files
2. Verify backend serving static files correctly
3. Check CORS settings if cross-origin

### Issue: Backend logs don't show profileImage
**Diagnostics:**
- [ ] Code change not deployed properly
- [ ] Backend not restarted
- [ ] Using old backend process

**Solution:**
1. Kill all node processes: `killall node`
2. Restart backend: `npm start`
3. Check console logs in real-time
4. Retry login

---

## ✅ Final Checklist Before "Done"

- [ ] Backend code updated (authController.js)
- [ ] Backend server restarted
- [ ] Test case 1 passed (Upload → Logout → Login)
- [ ] Test case 2 passed (Cross-device)
- [ ] Test case 3 passed (App restart)
- [ ] Test case 4 passed (Multiple users)
- [ ] Test case 5 passed (API response)
- [ ] Test case 6 passed (No image)
- [ ] Backend logs show profileImage: yes
- [ ] App logs show image refresh
- [ ] No console errors
- [ ] Multiple users tested
- [ ] Different image formats tested (JPG, PNG)
- [ ] Large images tested (5MB+)

---

## 🎯 Success Criteria

All of these must be true:

✅ User can upload image
✅ Image shows in Dashboard
✅ User can logout completely
✅ User can login again
✅ Image still shows after re-login
✅ Image available on different device (same account)
✅ Backend returns profileImage in login response
✅ No errors in console
✅ Works offline (cached)
✅ Works with multiple accounts

---

## 📝 Git Commands

```bash
# Check what changed
git diff CampusTrackBackend/controllers/authController.js

# Commit the changes
git add CampusTrackBackend/controllers/authController.js
git commit -m "Fix: Include profileImage in auth responses for session persistence"

# Push to main branch
git push origin master
```

---

## 🚀 Go Live

Once all tests pass:

1. [ ] Run all test cases in production-like environment
2. [ ] Get team approval
3. [ ] Deploy to production server
4. [ ] Monitor backend logs for errors
5. [ ] Have rollback plan ready (just restart with old code)
6. [ ] Announce feature to users

---

## 📞 Support

If issues arise during testing:

1. Check backend logs: `journalctl -u campustrack-backend -f`
2. Check app logs via Android Studio / Xcode
3. Test API directly with Postman
4. Review this document's troubleshooting section
5. Check git diff to ensure changes are correct

---

**Status**: Ready for Testing ✅

