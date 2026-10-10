# Firebase Setup Guide for AFAHJOY Portal

This document provides step-by-step instructions to configure your Firebase project for the AFAHJOY portal.

## Your Firebase Project Details

**Project ID:** `ais-website-547fa`  
**Auth Domain:** `ais-website-547fa.firebaseapp.com`

---

## Step 1: Enable Authentication

1. Go to [Firebase Console](https://console.firebase.google.com)
2. Select your project **`ais-website-547fa`**
3. Navigate to **Authentication** (left sidebar)
4. Click **Get Started**
5. Choose **Email/Password** as the sign-in method
6. Toggle **Enable** and click **Save**

✅ This allows users to sign in with email and password.

---

## Step 2: Create Firestore Database

1. In Firebase Console, go to **Firestore Database** (left sidebar)
2. Click **Create Database**
3. Choose location: **Select the region closest to your users** (e.g., `europe-west1` for Europe, `us-central1` for North America)
4. Start in **Production mode**
5. Click **Create**

✅ This creates the database that stores user profiles and account data.

---

## Step 3: Set Firestore Security Rules

1. In Firestore, go to **Rules** tab
2. Replace the default rules with this security configuration:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Allow signed-in users to read/write their own user document
    match /users/{userId} {
      allow read, write: if request.auth.uid == userId;
    }
    
    // Allow admins to read and manage all user documents
    match /users/{userId} {
      allow read, write: if 
        request.auth.uid == userId || 
        (get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin');
    }
    
    // Counter for auto-generating student IDs (admin only)
    match /meta/{document=**} {
      allow read, write: if
        exists(/databases/$(database)/documents/users/$(request.auth.uid)) &&
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
  }
}
```

3. Click **Publish**

✅ This ensures users can only access their own data (unless they're an admin).

---

## Step 4: Create Initial Admin Account (Manual)

### Option A: Via Firebase Console (Easiest)

1. In **Authentication** → **Users** tab
2. Click **Add User**
3. Enter:
   - **Email:** `admin@afahjoyintlschool.com`
   - **Password:** (Set a temporary strong password)
4. Click **Add User**
5. Copy the new user's **UID**
6. Go to **Firestore Database** → Collections
7. Create new collection: `users`
8. Create new document with ID = the UID you copied
9. Add fields:
   - `role` (string): `admin`
   - `name` (string): `Admin`
   - `email` (string): `admin@afahjoyintlschool.com`
   - `createdAt` (timestamp): Current time

✅ Admin account created and ready to use.

### Option B: Via Portal (After Enabling)

Once authentication and Firestore are live:
1. Deploy the portal
2. Sign in with the admin credentials you created in Option A
3. Use the admin dashboard to add teachers, students, and parents

---

## Step 5: Enable Firestore Offline Persistence (Optional)

This allows users to see cached data even if offline:

1. The portal JavaScript already includes offline persistence
2. No additional Firebase configuration needed
3. Works automatically once Firestore is enabled

---

## Step 6: Test the Setup

1. **Test Authentication:**
   - Open `login.html`
   - Try signing in with the admin account you created
   - You should be redirected to `admin.html`

2. **Test Firestore:**
   - On the admin dashboard, try creating a teacher, student, or parent account
   - Check Firestore in the console to verify the new account was created
   - The new user should be able to sign in

3. **Test Role-Based Redirects:**
   - Sign in as a teacher → should see `teacher.html`
   - Sign in as a student → should see `student.html`
   - Sign in as a parent → should see `parent.html`

---

## Step 7: Customize Authentication Settings (Optional)

### Email Verification
1. Go to **Authentication** → **Templates**
2. Customize the email verification template if desired

### Password Reset
The portal already includes a "Forgot password?" link that triggers Firebase's password reset flow.

### Session Persistence
- Users who check "Remember me" stay logged in longer
- Without it, users are logged out when the browser closes

---

## Step 8: Monitor Usage

1. Go to **Firestore Database** → **Usage**
2. Monitor reads, writes, and deletes to stay within the free tier limits
3. Go to **Authentication** → **Sign-in methods** for user metrics

---

## Troubleshooting

### "Permission denied" errors
- Check the Firestore security rules above
- Ensure the user's role is set correctly in the `users` collection

### Admin can't see accounts
- Verify the admin's document has `role: "admin"`
- Check browser console for JavaScript errors (F12 → Console tab)

### Login not working
- Ensure Authentication (Email/Password) is enabled
- Verify the email and password are correct
- Check if the user account exists in Firebase Console

### Account creation fails
- Ensure the admin is logged in
- Verify Firestore rules allow admin operations
- Check if the email is already in use

---

## Security Best Practices

1. ✅ Use strong passwords (min 12 characters, mix of uppercase, lowercase, numbers, symbols)
2. ✅ Enable 2FA on your Firebase Console account
3. ✅ Regularly review active users in **Authentication** → **Users**
4. ✅ Set up billing alerts in Firebase Console to avoid unexpected charges
5. ✅ Never commit Firebase credentials to version control (already safe in this repo)

---

## Next Steps

Once you enable the services above:

1. Test the portal by signing in and creating accounts
2. Customize dashboard content in `teacher.html`, `student.html`, `parent.html`, `admin.html`
3. Add more features (grades, attendance tracking, messaging, etc.)
4. Deploy the site to Firebase Hosting for production use

---

## Contact & Support

For Firebase issues:
- [Firebase Documentation](https://firebase.google.com/docs)
- [Firebase Support](https://firebase.google.com/support)

For portal issues:
- Check browser console (F12 → Console)
- Review the inline comments in `login.html` and dashboard files
