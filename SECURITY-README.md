# Security & Privacy Implementation Guide

## 🛡️ Firebase Security Rules

### Deploying Rules to Firebase Console

1. **Navigate to Firebase Console**: https://console.firebase.google.com
2. **Select your project**: "carbooler"
3. **Go to Realtime Database > Rules**
4. **Copy and paste** the contents of `firebase-rules.json`
5. **Click Publish**

### What the Rules Do

- **Per-Event Isolation**: Each carpool event lives at `/events/{eventId}`
- **Collaborative Access**: Anyone with the event URL can read/write (for real-time collaboration)
- **Data Validation**: Enforces structure and limits (e.g., event names max 100 chars)
- **No Cross-Event Access**: Events are isolated - you can't access other events' data

### Future Enhancements (Optional)

For production apps with higher security needs, consider:
- **Firebase Authentication**: Require sign-in before editing
- **Owner-only deletion**: Only event creators can delete events
- **Rate limiting**: Add server-side rate limiting via Cloud Functions

---

## 📱 App Store Privacy Requirements

### Data Collection Declaration

When submitting to App Store Connect, declare the following:

#### Data Types Collected:
1. **Contact Info**
   - **Name**: User-provided participant names
   - **Purpose**: App Functionality
   - **Linked to User**: NO
   - **Used for Tracking**: NO

2. **Identifiers**
   - **Device/Other IDs**: Firebase anonymous identifiers
   - **Purpose**: App Functionality (real-time sync)
   - **Linked to User**: NO
   - **Used for Tracking**: NO

3. **Usage Data**
   - **Product Interaction**: Network requests
   - **Purpose**: App Functionality (collaborative editing)
   - **Linked to User**: NO
   - **Used for Tracking**: NO

### Privacy Policy

- **In-app link**: Footer link to `privacy-policy.html`
- **Hosted version**: Upload `privacy-policy.html` to your website or GitHub Pages
- **Update App Store**: Add the hosted URL to your App Store listing

---

## 🔒 Architecture Overview

### Event-Based Data Model

**Old (Unsafe)**:
```
/carpoolData
  ├── eventName: "Beach Trip"
  ├── cars: [...]
  └── unassignedPassengers: [...]
```
❌ Everyone writes to the same node → race conditions

**New (Safe)**:
```
/events
  ├── {uuid-1}
  │   ├── eventName: "Beach Trip"
  │   ├── cars: [...]
  │   ├── unassignedPassengers: [...]
  │   └── lastUpdated: 1699999999
  └── {uuid-2}
      ├── eventName: "Ski Trip"
      └── ...
```
✅ Each event has its own space → no conflicts

### How Sharing Works

1. **Create Event**: User visits app → generates UUID → URL becomes `?e={uuid}`
2. **Share Link**: User clicks "Share" → copies/shares URL
3. **Join Event**: Friend opens link → reads UUID from URL → connects to same event
4. **Real-time Sync**: Firebase listeners update everyone instantly

### Atomic Updates

```javascript
// ✅ Good: Field-level updates (prevents overwriting)
const updates = {};
updates[`events/${eventId}/cars`] = cars;
updates[`events/${eventId}/eventName`] = name;
database.ref().update(updates);

// ❌ Bad: Set entire object (overwrites concurrent changes)
dbRef.set({eventName, cars, passengers});
```

---

## ✅ Testing Checklist

- [ ] Deploy Firebase rules to console
- [ ] Test creating a new event (gets UUID in URL)
- [ ] Test sharing link (copy/paste works)
- [ ] Test joining event (pasting URL loads data)
- [ ] Test concurrent editing (open in 2 tabs, edit both)
- [ ] Test privacy policy link works
- [ ] Submit App Store with correct data declarations

---

## 📄 Files Modified

- ✅ `index.html` - Event-based Firebase architecture + Share button + Privacy link
- ✅ `privacy-policy.html` - Privacy policy page
- ✅ `firebase-rules.json` - Security rules (deploy to Firebase Console)

---

## 🚀 Next Steps

1. **Deploy Firebase Rules**: Copy `firebase-rules.json` to Firebase Console
2. **Host Privacy Policy**: Upload `privacy-policy.html` to your domain
3. **Update App Store**: Add privacy policy URL + data declarations
4. **Test Thoroughly**: Verify sharing and concurrent editing work

---

Made with ❤️ for safe carpooling!
