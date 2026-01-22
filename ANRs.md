**ANR** means **Application Not Responding**.  
In simple terms: **your app freezes and Android gets impatient**.

### What actually causes an ANR

Android expects the **main (UI) thread** to stay responsive.

If the main thread is blocked for:

- ~5 seconds (UI input)
    
- ~10 seconds (broadcast receiver)
    
- ~20 seconds (service)
    

👉 Android shows the **ANR dialog** and may kill the app.

---

### Common real-world reasons

- Running network calls on the UI thread
    
- Heavy computation on main thread
    
- Waiting for location, payment, or biometric result synchronously
    
- Blocking Method Channel handlers
    
- Deadlocks or infinite loops
    

---

### Why ANRs are dangerous

- Terrible user experience
    
- App Store / Play Store penalties
    
- High uninstall rates
    
- Interview red flag if you don’t understand them
    

---

### One-line interview definition

> “ANR happens when the Android main thread is blocked for too long, causing the app to stop responding.”

Say that cleanly and move on.

---

### Flutter + ANR (important link)

In Flutter apps:

- Method Channel handlers run on Android’s **main thread**
    
- Blocking them = **guaranteed ANR**
    

That’s why threading discipline matters.

---

Bottom line:  
**ANR is Android telling you your app is stuck thinking instead of responding.**