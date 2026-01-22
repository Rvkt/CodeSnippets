### SharedPreferences vs Secure Storage

|Aspect|SharedPreferences|Secure Storage (Keystore / Keychain)|
|---|---|---|
|**Purpose**|Store simple app settings|Store sensitive data securely|
|**Security level**|❌ Low|✅ High|
|**Encryption**|❌ No (plain text XML)|✅ Yes (OS-level encryption)|
|**Hardware-backed protection**|❌ No|✅ Yes (on supported devices)|
|**Accessible on rooted/jailbroken devices**|❌ Very easy|⚠️ Much harder (not impossible, but protected)|
|**Suitable for tokens**|❌ No|✅ Yes|
|**Suitable for passwords / secrets**|❌ Never|✅ Yes|
|**Performance**|✅ Very fast|⚠️ Slightly slower (encryption overhead)|
|**Ease of use**|✅ Very simple|⚠️ Slightly more complex|
|**Survives app restart**|✅ Yes|✅ Yes|
|**Survives app reinstall**|❌ No|❌ No (by design)|
|**Used for**|Flags, UI state, preferences|Tokens, API keys, sensitive identifiers|
|**Flutter access**|Direct via plugin|Via OS secure APIs through plugin|
|**Compliance & best practices**|❌ Not compliant for sensitive data|✅ Industry standard|

---

### How to explain this in one sentence (interview-safe)

> “SharedPreferences is for non-sensitive configuration data, while Secure Storage uses OS-level encryption and hardware protection and is required for storing tokens or secrets.”

---

### Simple rule to remember

- **If leaking it is harmless → SharedPreferences**
    
- **If leaking it is dangerous → Secure Storage**
    

That rule alone prevents most security mistakes.