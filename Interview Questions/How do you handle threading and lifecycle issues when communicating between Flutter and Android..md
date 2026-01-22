> I treat Method Channel calls as asynchronous, make sure heavy work never runs on the main thread, and always bind native calls to the Android lifecycle so Flutter doesn’t receive callbacks after the activity is destroyed.


I keep Method Channel handlers lightweight, run heavy work off the UI thread, and make all callbacks lifecycle-aware so Flutter never receives responses from a destroyed Activity.


> “I handle Method Channel calls asynchronously, making sure heavy or blocking work runs off the Android main thread. I also make all callbacks lifecycle-aware by checking Activity state and handling results only when Flutter is active, which prevents UI freezes and lifecycle-related crashes.”


### Rule 1: Never block the main thread

Inside the Method Channel handler:

- I **immediately move heavy work** to:
    - background thread
    - executor
    - coroutine (`Dispatchers.IO`)
        

Flutter gets a response **only after work finishes**.

Conceptually:

- Method Channel → receive call
    
- Background thread → do work
    
- Main thread → send result back
    

This keeps:

- Flutter UI smooth
    
- Android responsive
    
- ANRs away


# [[ANRs]]