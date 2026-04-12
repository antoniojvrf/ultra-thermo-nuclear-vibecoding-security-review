# Firebase Security

## Firestore Security Rules

### DANGEROUS Patterns
```javascript
// ❌ CRITICAL — allows anyone to read/write everything
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if true;
    }
  }
}

// ❌ CRITICAL — allows any authenticated user to read/write everything
match /{document=**} {
  allow read, write: if request.auth != null;
}
```

### SECURE Patterns
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users can only access their own profile
    match /users/{userId} {
      allow read, update: if request.auth != null && request.auth.uid == userId;
      allow create: if request.auth != null && request.auth.uid == userId;
      allow delete: if false; // admin only via Admin SDK
    }

    // Organization documents — members only
    match /orgs/{orgId}/documents/{docId} {
      allow read: if request.auth != null &&
        exists(/databases/$(database)/documents/orgs/$(orgId)/members/$(request.auth.uid));
      allow write: if request.auth != null &&
        get(/databases/$(database)/documents/orgs/$(orgId)/members/$(request.auth.uid)).data.role == 'admin';
    }

    // Validate data shape on write
    match /posts/{postId} {
      allow create: if request.auth != null
        && request.resource.data.authorId == request.auth.uid
        && request.resource.data.keys().hasAll(['title', 'content', 'authorId'])
        && request.resource.data.title is string
        && request.resource.data.title.size() <= 200;
    }
  }
}
```

## Firebase Auth
```javascript
// ❌ WRONG — trusting client-side auth state for server decisions
const user = firebase.auth().currentUser;
fetch('/api/admin', { headers: { uid: user.uid } }); // attacker can fake this

// ✅ CORRECT — verify ID token server-side
// Client:
const idToken = await firebase.auth().currentUser.getIdToken();
fetch('/api/admin', { headers: { Authorization: `Bearer ${idToken}` } });

// Server:
const decodedToken = await admin.auth().verifyIdToken(idToken);
const uid = decodedToken.uid;
```

## Storage Rules
```javascript
service firebase.storage {
  match /b/{bucket}/o {
    // Users can only access their own folder
    match /users/{userId}/{allPaths=**} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }

    // Validate file type and size
    match /uploads/{fileId} {
      allow write: if request.auth != null
        && request.resource.size < 5 * 1024 * 1024  // 5MB max
        && request.resource.contentType.matches('image/.*');
    }
  }
}
```

## Checklist
- [ ] No wildcard `{document=**}` rules with `allow: if true`
- [ ] Rules scope access by `request.auth.uid`
- [ ] Data validation in security rules (field types, required fields, size limits)
- [ ] ID tokens verified server-side with Admin SDK
- [ ] Storage rules restrict by user and validate file type/size
- [ ] Admin SDK operations use service account (NOT client-side)
- [ ] Custom claims used for roles (not client-provided values)
