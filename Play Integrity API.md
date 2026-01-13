# 🔐 Google Play Integrity API – Full Integration Guide (Fintech Safe)

> **Use case:**  
> Prevent fraud, rooted devices, emulators, tampered APKs, SIM-swap frameworks, and hooking tools in Android apps.

> **Mandatory for:**  
> AEPS, KYC, Wallet, Banking, UPI, Payout apps.

---

## 📌 Architecture Overview

```text
Flutter App
   ↓
Android (Play Integrity SDK)
   ↓
Integrity Token
   ↓
Backend Server
   ↓
Google Play Integrity REST API
   ↓
Verdict (Allow / Block / Re-Verify)
```

⚠️ **Never verify integrity on the client**  
⚠️ **Always verify on backend**

---

## 1️⃣ Enable Play Integrity API (Play Console)

### Steps

1. Open **Google Play Console**
    
2. Select your app
    
3. Go to  
    `Setup → App integrity`
    
4. Enable **Play Integrity API**
    
5. Link or create a **Google Cloud Project**
    

📌 **Note down**

- Package name
    
- Cloud Project ID
    

---

## 2️⃣ Enable API in Google Cloud

1. Go to: [https://console.cloud.google.com/](https://console.cloud.google.com/)
    
2. Select the **same Cloud Project**
    
3. Navigate to  
    `APIs & Services → Library`
    
4. Enable:
    
    ```
    Google Play Integrity API
    ```
    

---

## 3️⃣ Create Service Account (Backend Only)

1. Cloud Console → `APIs & Services → Credentials`
    
2. Create **Service Account**
    
3. Assign role:
    
    ```
    Play Integrity API User
    ```
    
4. Generate and download **JSON key**
    

⚠️ Store this **only on backend**, never in app.

---

## 4️⃣ Android Setup

### Dependency

**`android/app/build.gradle.kts`**

```kotlin
dependencies {
    implementation("com.google.android.play:integrity:1.3.0")
}
```

---

## 5️⃣ Android – Request Integrity Token (Kotlin)

### `PlayIntegrityService.kt`

```kotlin
package com.workbench

import android.content.Context
import android.util.Base64
import android.util.Log
import com.google.android.play.core.integrity.IntegrityManagerFactory
import com.google.android.play.core.integrity.IntegrityTokenRequest
import java.security.SecureRandom

object PlayIntegrityService {

    private const val TAG = "PlayIntegrityService"

    fun requestIntegrityToken(
        context: Context,
        callback: (String?) -> Unit
    ) {
        try {
            val integrityManager = IntegrityManagerFactory.create(context)

            val nonce = generateNonce()

            val request = IntegrityTokenRequest.builder()
                .setNonce(nonce)
                .build()

            integrityManager.requestIntegrityToken(request)
                .addOnSuccessListener { response ->
                    callback(response.token())
                }
                .addOnFailureListener {
                    callback(null)
                }

        } catch (e: Exception) {
            Log.e(TAG, "Integrity error", e)
            callback(null)
        }
    }

    private fun generateNonce(): String {
        val bytes = ByteArray(32)
        SecureRandom().nextBytes(bytes)
        return Base64.encodeToString(bytes, Base64.NO_WRAP)
    }
}
```

---

## 6️⃣ Expose to Flutter (MethodChannel)

### `MainActivity.kt`

```kotlin
MethodChannel(
    flutterEngine.dartExecutor.binaryMessenger,
    "device_info_channel"
).setMethodCallHandler { call, result ->
    when (call.method) {
        "getPlayIntegrityToken" -> {
            PlayIntegrityService.requestIntegrityToken(this) { token ->
                result.success(token)
            }
        }
        else -> result.notImplemented()
    }
}
```

---

## 7️⃣ Flutter Side

```dart
static const _channel = MethodChannel('device_info_channel');

Future<String?> getIntegrityToken() async {
  return await _channel.invokeMethod<String>('getPlayIntegrityToken');
}
```

---

## 8️⃣ Backend Verification (MANDATORY)

### REST Endpoint

```http
POST https://playintegrity.googleapis.com/v1/{packageName}:decodeIntegrityToken
```

Example:

```text
https://playintegrity.googleapis.com/v1/com.finrich.distributor:decodeIntegrityToken
```

---

### Backend Request (Conceptual)

```json
{
  "integrityToken": "TOKEN_FROM_APP"
}
```

Authorization:

```http
Authorization: Bearer <OAuth2 Access Token>
```

---

## 9️⃣ Verdict Fields to Check

### Example Response

```json
{
  "tokenPayloadExternal": {
    "appIntegrity": {
      "appRecognitionVerdict": "PLAY_RECOGNIZED"
    },
    "deviceIntegrity": {
      "deviceRecognitionVerdict": [
        "MEETS_DEVICE_INTEGRITY",
        "MEETS_BASIC_INTEGRITY"
      ]
    }
  }
}
```

---

## 🔒 REQUIRED VERDICTS (Fintech Rules)

Allow sensitive actions **ONLY IF**:

|Field|Required|
|---|---|
|`appRecognitionVerdict`|`PLAY_RECOGNIZED`|
|`MEETS_DEVICE_INTEGRITY`|✅|
|`MEETS_BASIC_INTEGRITY`|✅|

❌ If missing → **Block KYC / AEPS / Payout**

---

## 1️⃣0️⃣ When to Call Play Integrity

Call **only on sensitive events**:

- App launch
    
- Login
    
- KYC submission
    
- AEPS transaction
    
- Wallet payout
    
- SIM change detected
    

⚠️ API is **rate limited** — don’t spam.

---

## 1️⃣1️⃣ Combine With SIM Binding (Best Practice)

```text
Integrity Verdict OK
        ↓
SIM Environment Hash Match
        ↓
Allow Sensitive Action
```

If mismatch → OTP re-verification.

---

## 1️⃣2️⃣ Play Console Declaration (Copy-Paste)

> _We use the Google Play Integrity API to verify that the app is genuine and running on a trusted Android device. This helps prevent fraud, abuse, and unauthorized access. No personal or sensitive user data is collected._

---

## ❌ What NOT To Do (Guaranteed Rejection)

- ❌ Verify token on client
    
- ❌ Call REST API from Flutter
    
- ❌ Skip deviceIntegrity checks
    
- ❌ Store service account key in app
    

---

## ✅ Compliance Checklist

- ✔ Official Google API
    
- ✔ Backend verification
    
- ✔ No restricted permissions
    
- ✔ Fraud prevention purpose
    
- ✔ Used by banks & UPI apps
    

---

## 🏁 Final Notes

- Play Integrity **replaces SafetyNet**
    
- Mandatory for fintech apps in 2025
    
- Combine with **SIM binding + device binding**
    
- Client verdicts are **never trusted**
    

---

## 🚀 Next Steps

- Backend implementation (Node.js / Java / Spring Boot)
    
- SIM + Integrity combined fraud rules
    
- Cool-down logic after SIM swap
    
- Production vs UAT integrity handling