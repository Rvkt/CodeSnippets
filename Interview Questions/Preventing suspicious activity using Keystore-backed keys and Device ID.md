
To prevent suspicious activities such as **multiple account usage, rapid re-logins, or app reinstalls**, we combine **Keystore-backed cryptographic keys** with a **device identifier (Android ID)**.

A **Keystore-backed key** creates a secure, app-scoped identity that proves the authenticity of a specific app installation. This key is hardware-protected, non-exportable, and cannot be cloned, making it reliable for detecting abuse within the same app install.

The **device ID** acts as a continuity signal across reinstalls. While weaker than a Keystore key, it helps the backend recognize repeated activity from the same physical device even when the app is reinstalled.

By correlating:

- install identities (Keystore keys),
    
- device continuity (Android ID),
    
- login behavior and frequency,
    

the backend can **identify patterns of misuse** such as account hopping or reinstall-based evasion.

This approach:

- avoids forbidden identifiers (IMEI, MAC)
    
- respects user privacy and Play Store policies
    
- relies on risk scoring instead of hard assumptions
    
- enables step-up security (OTP, biometric re-verification) when needed
    

---

### One-line interview version

> “We prevent suspicious activity by combining a Keystore-backed install identity with Android ID for continuity, allowing the backend to detect account abuse patterns across installs in a privacy-compliant way.”

---

### Bottom line

Security here isn’t about tracking users — it’s about **detecting abnormal behavior through trusted signals and correlation**.