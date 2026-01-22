> “I assume the app can be reverse engineered, so I reduce what an attacker can learn and limit what they can abuse.”

**In practice, I do this:**

- **Obfuscation & shrinking**  
    I obfuscate code (names, logic flow) and remove unused code to make reverse engineering harder and noisy.
    
- **No secrets in the app**  
    I never store API keys, tokens, or business rules in the app. All sensitive logic lives on the backend.
    
- **Secure runtime checks**  
    I detect tampering, rooting/jailbreak, debuggers, and emulators, and respond with restricted behavior.
    
- **Backend-enforced security**  
    Even if APIs are discovered, the backend enforces auth, rate limits, device/app binding, and role checks.
    
- **Short-lived tokens & device binding**  
    Tokens expire quickly and are bound to app/device signals, so stolen values are useless.
    

---

### One-line interview version

> “I protect apps by obfuscating code, keeping all secrets and decisions on the backend, detecting tampering at runtime, and making stolen app logic or tokens useless.”

---

### Bottom line

You don’t _stop_ reverse engineering.  
You **make it expensive, unreliable, and pointless**.


----

Let’s explain the diagram **like you’re explaining it to a non-technical person**.

![Image](https://miro.medium.com/v2/resize%3Afit%3A1276/1%2AZRVvF37mrQLSjJEqNlsaRw.jpeg)



![Image](https://epic.blog/assets/2020-07-27-reverse-engineering-android-app/android-app/compilation.png)

---

## What this diagram is showing (in one line)

It shows **how someone can take an Android app and slowly open it up to see how it works inside**.

---

## Step 1: The app (APK file)

An Android app is just a file called an **APK**.

When someone installs the app:

- the APK sits on the phone
    
- anyone can copy that APK file
    

So the app is **not locked** once it’s installed.

---

## Step 2: Opening the APK

The APK is like a zip file.

People can open it and see:

- app settings
    
- permissions
    
- images and text
    
- the app’s internal structure
    

At this stage, they already learn:

- what the app does
    
- what permissions it needs
    
- how it’s organized
    

---

## Step 3: Reading the app logic

Inside the APK is the app’s logic in a compiled form.

Tools can:

- convert this compiled code
    
- turn it back into readable code
    

Now someone can:

- read how login works
    
- see how APIs are called
    
- understand app rules
    

It’s not perfect code, but **easy enough to understand**.

---

## Step 4: Running and watching the app

People don’t just read the code.

They also:

- run the app
    
- watch how it talks to servers
    
- see what data is sent and received
    
- change behavior while the app is running
    

This helps them understand the app even better.

---

## Step 5: Misusing what they learned

Once they understand the app, they can:

- fake app requests
    
- bypass UI limits
    
- automate actions
    
- abuse APIs
    

This is how fraud, cheating, and cloning happen.

---

## What the diagram is really teaching

The main message is:

> **You cannot hide app logic from users.**

So:

- anything inside the app can be seen
    
- frontend checks are not security
    
- secrets inside apps are already leaked
    

---

## Simple takeaway

The diagram explains why:

- apps must not trust themselves
    
- important checks must be on the server
    
- apps should assume they will be inspected
    

---

## One-line explanation (interview-friendly)

> “The diagram shows how an Android app can be unpacked, read, and analyzed, which is why security should never rely on hiding code inside the app.”

---

### Bottom line

An app is like a glass box.  
You can decorate it, but you can’t make it invisible.

Real security lives **outside the app**, not inside it.