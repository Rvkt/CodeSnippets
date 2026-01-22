
> I used Method Channels to bridge Flutter with Android native code for device info retrieval, biometric KYC via RD Service apps, and payment app integration using Intents and Pending Intent callbacks.

---

### Real use case: Integrating native Android features in a Flutter app

In one of my projects, I worked on a Flutter application that needed **deep access to Android system features**, which are **not fully available in Flutter by default**. So I implemented **Android native code using Method Channels**.

---

### 1. Fetching device information (security + analytics use case)

The backend required **device-level details** for:

- fraud detection
    
- user identification
    
- compliance logging
    

From Flutter, I called Android native code to fetch:

- **Device ID** (secure Android ID)
    
- **Device model**
    
- **Manufacturer**
    
- **Latitude & longitude**
    

Why native?  
Flutter cannot reliably access:

- low-level device identifiers
    
- accurate location permissions handling
    
- system APIs needed for secure device fingerprinting
    

How it worked:

- Flutter sent a request using a Method Channel
    
- Android used system APIs like `Build`, `Settings.Secure`, and `LocationManager`
    
- Native code returned clean, structured data back to Flutter
    

Flutter treated it as a normal `await` call, but internally it was native execution.

---

### 2. Calling RD Service apps for KYC (biometric flow)

For **KYC verification**, we had to interact with **RD Service apps** installed on the device (used for biometric authentication like fingerprint).

This is something Flutter **cannot do directly**.

Implementation:

- Flutter triggered a Method Channel call
    
- Android launched the RD Service app using an **Intent**
    
- The RD Service app performed biometric capture
    
- Result was returned to our app via **Intent callback**
    
- Android parsed the response and sent only the required data back to Flutter
    

This ensured:

- secure biometric flow
    
- zero exposure of raw biometric data in Flutter
    
- compliance with KYC guidelines
    

---

### 3. Calling payment apps using Intent & PendingIntent

For payments (UPI / bank apps):

- Flutter initiated the payment request
    
- Android launched external payment apps using **Intent**
    
- We used **PendingIntent** to safely receive the payment status callback
    
- Success / failure / cancellation was handled on the native side
    
- Final result was sent back to Flutter
    

Why this approach:

- Payment apps expect **native Android Intents**
    
- Callbacks are asynchronous and OS-controlled
    
- Flutter alone cannot manage this lifecycle reliably
    

---

### Why Method Channels were the right choice

- Flutter handled UI and business logic
    
- Android handled **system-level responsibilities**
    
- Clear separation of concerns
    
- Secure, stable, and production-ready