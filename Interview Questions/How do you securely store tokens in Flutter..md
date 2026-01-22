## Core principle (say this first)

> “Tokens should never be stored in plain Flutter memory or local files; they must be protected using platform-level secure storage.”

That sets the tone: you respect security boundaries.

---

## How secure token storage works in Flutter (simple English)

Flutter itself **does not have a secure vault**.

So the correct approach is:

- Flutter **delegates storage** to the OS
    
- Android uses **Keystore**
    
- iOS uses **Keychain**
    

Flutter is just the messenger.

---

## The correct storage strategy (production-grade)

### 1. Use OS-backed secure storage

Tokens are stored via:

- Android → Keystore-encrypted storage
    
- iOS → Keychain
    

This gives:

- hardware-backed encryption (on most devices)
    
- protection against rooting / jailbreaking
    
- automatic OS-level access control
    

Flutter never sees the raw encryption key.

---

### 2. Store only what is necessary

You store:

- access token
    
- refresh token (if needed)
    
- token expiry metadata
    

You do **not** store:

- user passwords
    
- encryption keys
    
- biometric data
    

Tokens are treated as **temporary credentials**, not identity.

---

### 3. Keep tokens out of memory

Rules I follow:

- never log tokens
    
- never keep tokens in global variables
    
- clear tokens on logout
    
- refresh tokens just-in-time
    

Flutter memory is volatile and inspectable — treat it as unsafe.

---

### 4. Bind tokens to device/app context

For higher security:

- backend binds tokens to:
    
    - device signals
        
    - app install identity
        
- token misuse on another device is rejected
    

So even if a token leaks, it’s less useful.

---

### 5. Short-lived access tokens, controlled refresh

Security is not just storage — it’s lifecycle.

Best practice:

- short-lived access tokens
    
- refresh tokens stored securely
    
- automatic refresh via interceptor
    
- revoke tokens on logout or suspicious activity
    

This limits blast radius.

---

### 6. Handle edge cases safely

I also handle:

- app reinstall → tokens wiped
    
- device lock changes → storage invalidation
    
- biometric or PIN reset → re-auth required
    

OS handles most of this automatically if you use secure storage correctly.

---

## What NOT to do (instant red flags)

- ❌ SharedPreferences
    
- ❌ SQLite
    
- ❌ plain files
    
- ❌ environment variables
    
- ❌ hardcoded tokens
    
- ❌ “encrypted” Dart code
    

If Flutter can read it easily, attackers can too.

---

## One-line L2/L3 interview answer

> “I store tokens using OS-level secure storage backed by Android Keystore and iOS Keychain, keep them short-lived, and never expose them in Flutter memory or logs.”

That line is strong and safe.

---

### Bottom line

Flutter **orchestrates** security.  
The OS **enforces** it.

If you respect that boundary, your token storage is solid.