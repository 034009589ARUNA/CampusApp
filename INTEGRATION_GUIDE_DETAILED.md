# Integration Guide - Implementation Details

## 📌 System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     FRONTEND (React Native)                      │
├─────────────────────────────────────────────────────────────────┤
│  StudentSettings.tsx                                             │
│  ├─ Delete Account Button                                        │
│  ├─ Confirmation Dialog                                          │
│  ├─ Password Modal                                               │
│  └─ API Call: DELETE /api/auth/delete-account                    │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      │ HTTP
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                    BACKEND (Express.js)                          │
├─────────────────────────────────────────────────────────────────┤
│  Auth Routes (auth.js)                                           │
│  ├─ POST /api/auth/signup                                        │
│  ├─ POST /api/auth/login                                         │
│  ├─ DELETE /api/auth/delete-account ← NEW                        │
│  └─ Other auth endpoints                                         │
│                                                                   │
│  Auth Controller (authController.js)                             │
│  ├─ signup()                                                     │
│  ├─ login()                                                      │
│  ├─ deleteAccount() ← NEW                                        │
│  └─ Other controllers                                            │
│                     ↓                                             │
│  ┌─ Cleanup Handlers                                             │
│  ├─ Delete from Attendance collection                            │
│  ├─ Delete from AssignmentSubmission collection                  │
│  ├─ Delete from Device collection                                │
│  ├─ Update Chat collection                                       │
│  └─ Delete from Message collection                               │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      │ Database Operations
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                    MONGODB COLLECTIONS                           │
├─────────────────────────────────────────────────────────────────┤
│  Students → Deleted                                              │
│  Lecturers → Unchanged (if lecturer deletes)                    │
│  Attendance → All student records deleted                       │
│  AssignmentSubmissions → All student submissions deleted        │
│  Devices → Student device unregistered                          │
│  Chats → Student removed from participants                      │
│  Messages → Student messages deleted                            │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Attendance Flow with Anti-Cheating

```
┌─────────────────────────────────────────────────────────────────┐
│             STUDENT MARKS ATTENDANCE                             │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  POST /api/attendance/mark                                       │
│  Body: { code, latitude, longitude, accuracy, deviceId }        │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  VALIDATION LAYER 1: Basic Input                                │
│  ├─ code exists? ❌ Reject if missing                            │
│  ├─ coordinates valid? ❌ Reject if missing                      │
│  └─ deviceId provided? ❌ Reject if missing                      │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  VALIDATION LAYER 2: GPS Quality                                │
│  ├─ Accuracy <= 100m? ❌ Reject if too poor                     │
│  ├─ Coordinates valid? ❌ Reject if out of range                │
│  └─ Confidence level logged                                      │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  VALIDATION LAYER 3: Session Verification                       │
│  ├─ Code matches active session? ❌ Reject if invalid/expired   │
│  ├─ Venue location configured? ❌ Reject if missing             │
│  └─ Get session details (radius, location)                      │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  VALIDATION LAYER 4: Geofencing Check                           │
│  ├─ Within range? ❌ Reject if outside geofence                 │
│  ├─ Calculate distance from venue                                │
│  ├─ Adjust for GPS accuracy                                      │
│  └─ Log geofence data                                            │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  VALIDATION LAYER 5: Duplicate Prevention                       │
│  ├─ Already marked today? ❌ Reject if duplicate                │
│  └─ Check Attendance.find({ studentId, classSessionId, date })  │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  FRAUD DETECTION LAYER 1: Device Verification                   │
│  ├─ Device registered? ❌ Block if not registered               │
│  ├─ Device matches current? ❌ Block if different device         │
│  └─ Call: Student.populate('registeredDevice')                  │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  FRAUD DETECTION LAYER 2: Device Sharing Detection              │
│  ├─ Check: Device.find({ deviceId }) - multiple users?          │
│  ├─ ❌ Block if device registered to another student            │
│  └─ Log fraud attempt                                            │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  FRAUD DETECTION LAYER 3: Simultaneous Attendance               │
│  ├─ Check recent attendance (last 5 min)                        │
│  ├─ ❌ Block if marked at multiple sessions/locations            │
│  └─ Query: Attendance.find({ date: today, createdAt: last5min })│
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  FRAUD DETECTION LAYER 4: Proxy Attendance Detection            │
│  ├─ Check: Multiple students from same location                 │
│  ├─ Check: Deviations from historical patterns                  │
│  ├─ ⚠️  Flag for review (don't block)                            │
│  └─ Calculate risk level                                         │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  ANOMALY DETECTION: Location Spoofing                           │
│  ├─ Get previous attendance location                            │
│  ├─ Calculate speed between locations                           │
│  ├─ Check for rapid movement (>20 m/s)                          │
│  ├─ Check for GPS jumping (>1km in <10s)                        │
│  └─ ⚠️  Flag anomalies, record in fraudData                      │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  CALCULATE FRAUD SCORE (0-100)                                  │
│  ├─ + 20 for rapid movement                                      │
│  ├─ + 25 for GPS jumping                                         │
│  ├─ + 15 for suspicious location                                │
│  ├─ + 5 for poor GPS accuracy                                    │
│  ├─ + 5 for unusual time                                         │
│  └─ Assess risk level (LOW/MEDIUM/HIGH/CRITICAL)                │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  CREATE ATTENDANCE RECORD                                        │
│  ├─ Store location (lat, lon, accuracy)                         │
│  ├─ Store geofence data                                          │
│  ├─ Store anomaly flags                                          │
│  ├─ Store fraud score & assessment                               │
│  ├─ Store verification method (geofence)                         │
│  └─ Save to Attendance collection                                │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  UPDATE STATISTICS                                               │
│  ├─ Increment session.attendanceCount                            │
│  ├─ Update ClassSession                                          │
│  └─ Return success response with location data                   │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  RESPONSE TO CLIENT                                              │
│  ├─ Success with fraud score visibility (if flagged)             │
│  ├─ Location accuracy feedback                                   │
│  └─ Confirmation message                                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔐 Database Schema Changes

### Attendance Collection (Enhanced)

```javascript
{
  _id: ObjectId,
  studentId: ObjectId,           // ref: Student
  classSessionId: ObjectId,       // ref: ClassSession
  date: String,                   // YYYY-MM-DD
  
  // Location data
  location: {
    latitude: Number,
    longitude: Number,
    accuracy: Number              // GPS accuracy in meters
  },
  
  // Geofence data
  geofenceData: {
    venueLatitude: Number,
    venueLongitude: Number,
    distance: Number,
    withinRange: Boolean,
    geofenceRadius: Number,
    effectiveDistance: Number
  },
  
  // Verification metadata
  markedAt: Date,
  verified: Boolean,
  verificationMethod: String,     // 'geofence' | 'bluetooth' | 'qr_code' | etc
  
  // Bluetooth data (if BLE method)
  bluetoothData: {
    sessionId: String,
    rssi: Number,
    deviceId: String,
    secretKey: String,
    beaconDetectedAt: Date
  },
  
  // Anomaly flags
  anomalies: {
    rapidMovement: Boolean,
    suspiciousLocation: Boolean,
    gpsJumping: Boolean
  },
  
  // ← NEW: Fraud detection data
  fraudData: {
    fraudScore: Number,            // 0-100
    assessment: String,            // LOW_RISK | MEDIUM_RISK | HIGH_RISK | CRITICAL_RISK
    flaggedForReview: Boolean,
    reviewNotes: String
  },
  
  // Lecturer verification
  verifiedBy: ObjectId,            // ref: Lecturer
  verifiedAt: Date,
  remarks: String,
  
  // Timestamps
  createdAt: Date,
  updatedAt: Date,
  
  // Indexes
  // { studentId: 1, date: 1 }
  // { classSessionId: 1 }
  // { studentId: 1, classSessionId: 1 }
}
```

---

## 🔄 API Response Examples

### Success: Account Deletion
```json
{
  "success": true,
  "message": "Student account and all associated data deleted successfully"
}
```

### Success: Attendance Marked (Low Risk)
```json
{
  "success": true,
  "message": "Attendance marked successfully ✓",
  "data": {
    "attendanceId": "507f1f77bcf86cd799439011",
    "markedAt": "2026-02-01T10:15:30Z",
    "courseName": "Computer Science 101",
    "subject": "Introduction to Programming",
    "location": {
      "distance": 12,
      "accuracy": 8.5,
      "qualityMessage": "🟢 Excellent GPS signal (±8.5m)"
    },
    "fraudData": {
      "fraudScore": 5,
      "assessment": "LOW_RISK"
    }
  }
}
```

### Success: Attendance Marked (Flagged)
```json
{
  "success": true,
  "message": "Attendance marked successfully ✓",
  "data": {
    "attendanceId": "507f1f77bcf86cd799439012",
    "markedAt": "2026-02-01T14:30:45Z",
    "courseName": "Physics 101",
    "subject": "Quantum Mechanics",
    "location": {
      "distance": 45,
      "accuracy": 58,
      "qualityMessage": "🟡 Fair GPS signal (±58m)"
    },
    "fraudData": {
      "fraudScore": 35,
      "assessment": "MEDIUM_RISK",
      "flaggedForReview": true,
      "reviewNotes": "Multiple students from same location; poor GPS accuracy"
    }
  }
}
```

### Error: Device Not Registered
```json
{
  "success": false,
  "message": "Your device is not registered. Please register your device first."
}
```

### Error: Device Fraud Detected
```json
{
  "success": false,
  "message": "Device fraud detected. This device is registered to multiple accounts. Contact support."
}
```

### Error: Outside Geofence
```json
{
  "success": false,
  "message": "You are outside the attendance zone. Distance: 156m (Max: 50m)",
  "details": {
    "distance": 156,
    "maxDistance": 50,
    "bufferZone": -106,
    "accuracy": 12,
    "qualityMessage": "🟢 Excellent GPS signal (±12m)"
  }
}
```

### Error: GPS Quality Too Poor
```json
{
  "success": false,
  "message": "GPS quality is too poor to mark attendance",
  "details": {
    "issues": [
      "GPS accuracy is poor (>100m)",
      "Latitude out of range"
    ],
    "accuracy": 145,
    "qualityMessage": "🔴 Very Poor GPS signal (±145m)"
  }
}
```

---

## 🔌 Integration Checkpoints

### When Integrating into Production

#### 1. Database Indexes
```javascript
// Ensure these indexes exist in MongoDB
db.attendances.createIndex({ studentId: 1, date: 1 })
db.attendances.createIndex({ classSessionId: 1 })
db.attendances.createIndex({ studentId: 1, classSessionId: 1 })

// Optional but recommended for performance
db.attendances.createIndex({ "fraudData.fraudScore": 1 })
db.attendances.createIndex({ createdAt: 1 })
```

#### 2. Environment Variables
```env
# Make sure these are set
JWT_SECRET=your_secret_key
NODE_ENV=production
MONGODB_URI=your_connection_string
API_PORT=5000
CORS_ORIGIN=your_frontend_url
```

#### 3. API Base URL (Frontend)
```typescript
// Update in StudentSettings.tsx:
// Change: http://192.168.1.100:5000
// To: https://your-production-api.com or process.env.API_URL
```

#### 4. Rate Limiting
```javascript
// Account deletion is rate-limited
// Prevent abuse: max 1 delete per user per hour
// Configure in middleware/rateLimiter.js if needed
```

#### 5. Logging
```javascript
// Ensure these are logged for audit trail:
- Account deletion attempts (success and failure)
- All fraud detection events
- Device verification failures
- Geofence violations
```

---

## 🧪 Test Cases

### Account Deletion Tests
```javascript
describe('DELETE /api/auth/delete-account', () => {
  test('Should fail without password', () => {
    // Expect 400: Password required
  })
  
  test('Should fail with wrong password', () => {
    // Expect 401: Password incorrect
  })
  
  test('Should succeed with correct password', () => {
    // Expect 200: Account deleted
    // Verify: User no longer exists
    // Verify: Attendance records deleted
    // Verify: Device unregistered
  })
  
  test('Should fail if already deleted', () => {
    // Expect 404: User not found
  })
  
  test('Should clean up all related data', () => {
    // Verify: Attendance.find({ studentId }) returns empty
    // Verify: AssignmentSubmission.find({ studentId }) returns empty
    // Verify: Device.find({ student: userId }) returns empty
    // Verify: Chat participants no longer include user
  })
})
```

### Anti-Cheating Tests
```javascript
describe('POST /api/attendance/mark - Fraud Detection', () => {
  test('Should block unregistered device', () => {
    // Device not in Device collection
    // Expect 403: Device not registered
  })
  
  test('Should block shared device', () => {
    // Device registered to 2 different students
    // Expect 403: Device fraud detected
  })
  
  test('Should block simultaneous attendance', () => {
    // Mark attendance 2x in 5 minutes
    // Expect 403: Simultaneous attendance
  })
  
  test('Should flag proxy attendance', () => {
    // Multiple students from same location
    // Expect 200: Marked but fraudScore > 40
  })
  
  test('Should block impossible speeds', () => {
    // Previous location: 0,0; Current: 0.1,0.1; Time: 1s
    // Speed: ~11km/s (impossible)
    // Expect: Flagged or blocked
  })
  
  test('Should accept normal attendance', () => {
    // All checks pass
    // Expect 200: Attendance marked successfully
    // fraudScore < 20
  })
})
```

---

## 🚀 Deployment Steps

### Step 1: Backup Database
```bash
# Backup MongoDB before deployment
mongodump --uri="mongodb://your-connection-string" --out=./backup
```

### Step 2: Update Backend
```bash
cd CampusTrackBackend
git pull origin master
npm install  # In case new packages added
npm test     # Run tests
npm start    # Start server
```

### Step 3: Update Frontend
```bash
cd CampusTrack
git pull origin master
npm install  # In case new packages added
npm test     # Run tests
# Build for production
eas build --platform android
```

### Step 4: Verify Endpoints
```bash
# Test account deletion endpoint
curl -X DELETE http://localhost:5000/api/auth/delete-account \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"password": "yourpassword"}'

# Test attendance with fraud detection
curl -X POST http://localhost:5000/api/attendance/mark \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "ABC123",
    "latitude": 40.7128,
    "longitude": -74.0060,
    "accuracy": 12,
    "deviceId": "device-uuid"
  }'
```

### Step 5: Monitor Logs
```bash
# Watch for fraud detection alerts
tail -f /var/log/campusapp/fraud-detection.log

# Watch for account deletions
tail -f /var/log/campusapp/account-deletion.log

# General application logs
tail -f /var/log/campusapp/app.log
```

---

## 📊 Monitoring & Analytics

### Key Metrics to Track

```javascript
// Fraud Detection Metrics
- Total fraud attempts: fraud_attempts_total
- Blocked by device: fraud_device_blocked_total
- Blocked by simultaneous: fraud_simultaneous_blocked_total
- Flagged for review: fraud_flagged_total
- Average fraud score: fraud_score_average
- High-risk percentage: fraud_high_risk_percentage

// Account Deletion Metrics
- Total deletions: account_deletions_total
- Failed deletions: account_deletion_errors_total
- Data cleanup time: account_deletion_cleanup_time_ms
- Recovery attempts: account_deletion_recovery_attempts

// Attendance Metrics
- Total marked: attendance_marked_total
- Successful: attendance_success_total
- Blocked: attendance_blocked_total
- Geofence violations: geofence_violations_total
```

---

**Integration complete and production-ready!** ✅
