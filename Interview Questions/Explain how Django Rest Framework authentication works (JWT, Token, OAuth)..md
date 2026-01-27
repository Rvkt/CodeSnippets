

**L3 answer (clear, structured, interview-ready):**

> “In Django Rest Framework, authentication is handled at the API layer using pluggable mechanisms like Token, JWT, and OAuth. The backend validates credentials, issues a token, and every request is authenticated by verifying that token before permission checks are applied.”

**How each works (quickly):**

- **Token Authentication**  
    After login, the server issues a random token stored in the database.  
    The client sends this token with every request, and DRF matches it against the DB.  
    Simple, but not ideal for large-scale systems because it requires DB lookup on every request.
    
- **JWT (JSON Web Token)**  
    After login, the server issues a signed token containing user info and expiry.  
    The client sends it on each request, and the server verifies the signature—no DB lookup needed.  
    This is **stateless, fast, and scalable**, which is why it’s preferred for mobile apps.
    
- **OAuth (OAuth2)**  
    Used when third-party access is involved (Google login, enterprise SSO).  
    The user authorizes access, an access token is issued, and DRF validates it for API access.  
    Best for delegated access, not simple username-password auth.
    

**One-line closer (strong signal):**

> “Token auth is simple, JWT is stateless and scalable for mobile apps, and OAuth is used when third-party or delegated authentication is required.”


> The best security approach is backend-driven, uses standard protocols like HTTPS and JWT, stores secrets in OS secure storage, assumes the app can be reverse engineered, and relies on layered defenses instead of trusting the client.


> Security is not about hiding things.  It’s about limiting trust and controlling impact.