**RPC = Remote Procedure Call**

Big words. Simple meaning:

> You call a function, but the function runs **somewhere else**.

You _feel_ like you’re calling a normal method, but in reality:

- your app sends a message
    
- another system runs the code
    
- you get the result back
    

---

### Local call vs RPC call

#### Normal function call (local)

```dart
int sum = add(2, 3);
```

• Runs in the same program  
• Instant  
• Same memory

---

#### RPC-style call

```dart
final result = await channel.invokeMethod("getBatteryLevel");
```

• The code runs **outside Flutter**  
• Flutter sends a **request message**  
• Native code executes  
• Result comes back as a **response message**

Looks like a method call.  
Behaves like messaging.

That’s RPC.

---

### Why “style” and not pure RPC?

Classic RPC systems (gRPC, Thrift, etc.) have:

- strict schemas
    
- strong typing
    
- auto-generated code
    

Flutter Method Channels are **lighter**:

- string method names
    
- dynamic data
    
- manual handling
    

So we say **RPC-style**, not full RPC.

---

### What actually happens

When Flutter “calls” a method:

1. Flutter writes a message:
    
    > “Run `getBatteryLevel` with these inputs”
    
2. Native side reads the message
    
3. Native code runs
    
4. Native side replies:
    
    > “Here’s the result” or “Here’s an error”
    
5. Flutter receives the reply and continues
    

Request → response.  
One round trip.

---

### Key rule (important)

RPC-style calls are:

- **Slow compared to local calls**
    
- **Always async**
    
- **Failure-prone** (no handler, crash, timeout)
    

That’s why Flutter uses `Future` and `await`.

---

### One-line explanation (interview gold)

> “RPC-style messaging means Flutter looks like it’s calling a method, but it’s actually sending a request to native code and waiting for a reply.”

Say that calmly. Stop. Let it land.

---

### Simple real-world analogy

You email HR:

> “Can you approve my leave?”

You don’t approve it yourself.  
You wait.

They reply:

> “Approved” or “Rejected”.

That email exchange is RPC-style messaging.

---

Bottom line:  
**Method Channels pretend to be method calls, but they are really message exchanges across system boundaries.**