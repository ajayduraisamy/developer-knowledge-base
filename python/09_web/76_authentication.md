# Authentication - JWT, OAuth2, session vs token, password hashing

## Introduction
Authentication is the process of verifying the identity of a user or system. In web applications, authentication answers the question "Who are you?" while authorization answers "What are you allowed to do?" Python developers have several authentication strategies at their disposal: session-based authentication (server-side sessions with cookies), token-based authentication (client-side tokens like JWT), OAuth2 flows (delegated authorization), and password hashing (secure credential storage). Each approach has distinct trade-offs regarding security, scalability, state management, and user experience. Understanding these mechanisms is essential for building secure web applications and APIs.

## JWT

### What It Is
JSON Web Token (JWT) is a compact, URL-safe token format for representing claims between parties. A JWT consists of three Base64url-encoded segments (header, payload, signature) separated by dots. The header specifies the signing algorithm, the payload contains claims (user ID, roles, expiration), and the signature cryptographically verifies the token's integrity.

### Why It Is Important
JWTs enable stateless authentication: the server does not need to store session data because all necessary information is encoded in the token itself. This makes JWTs ideal for distributed systems, microservices, and mobile apps where sharing server-side sessions is impractical. They are also used for single sign-on (SSO), API authentication (Bearer tokens), and secure information exchange.

### How It Works Internally
When a user logs in, the server creates a JWT containing user claims and an expiration time, then signs it with a secret key (HMAC) or a private key (RSA/ECDSA). The token is returned to the client, which stores it (localStorage, cookie, mobile secure storage) and sends it in the `Authorization: Bearer <token>` header for subsequent requests. The server verifies the signature and expiration on each request without any database lookup.

### Syntax
```python
import jwt
from datetime import datetime, timedelta
from typing import Dict, Optional

SECRET_KEY = "your-secret-key-keep-it-safe"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

def create_access_token(data: Dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=15))
    to_encode.update({"exp": expire, "iat": datetime.utcnow()})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def verify_access_token(token: str) -> Dict:
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except jwt.ExpiredSignatureError:
        raise ValueError("Token has expired")
    except jwt.InvalidTokenError:
        raise ValueError("Invalid token")

# Usage
token = create_access_token({"sub": "user123", "role": "admin"})
payload = verify_access_token(token)
print(payload["sub"])  # "user123"
```

### Intermediate Examples
```python
import jwt
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel
from datetime import datetime, timedelta

app = FastAPI()
security = HTTPBearer()
SECRET_KEY = "super-secret-key"
ALGORITHM = "HS256"

class TokenResponse(BaseModel):
    access_token: str
    token_type: str = "bearer"

class UserPayload(BaseModel):
    sub: str
    role: str
    exp: datetime

def create_token(user_id: str, role: str = "user") -> TokenResponse:
    expire = datetime.utcnow() + timedelta(hours=1)
    payload = {"sub": user_id, "role": role, "exp": expire}
    token = jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)
    return TokenResponse(access_token=token)

def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)) -> dict:
    try:
        payload = jwt.decode(
            credentials.credentials,
            SECRET_KEY,
            algorithms=[ALGORITHM]
        )
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token expired"
        )
    except jwt.InvalidTokenError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid token"
        )

@app.post("/login")
def login(username: str, password: str):
    user = authenticate_user(username, password)
    if not user:
        raise HTTPException(401, "Invalid credentials")
    return create_token(user["id"], user["role"])

@app.get("/protected")
def protected_route(current_user: dict = Depends(get_current_user)):
    return {"message": "Access granted", "user": current_user["sub"]}

# Refresh token pattern
REFRESH_SECRET_KEY = "different-secret-for-refresh"

def create_refresh_token(user_id: str) -> str:
    expire = datetime.utcnow() + timedelta(days=30)
    payload = {"sub": user_id, "exp": expire, "type": "refresh"}
    return jwt.encode(payload, REFRESH_SECRET_KEY, algorithm=ALGORITHM)

@app.post("/refresh")
def refresh_token(refresh_token: str = Body(...)):
    try:
        payload = jwt.decode(refresh_token, REFRESH_SECRET_KEY, algorithms=[ALGORITHM])
        if payload.get("type") != "refresh":
            raise HTTPException(400, "Invalid token type")
        return create_token(payload["sub"])
    except jwt.ExpiredSignatureError:
        raise HTTPException(401, "Refresh token expired")
```

## OAuth2 flow

### What It Is
OAuth2 is an authorization framework that enables applications to obtain limited access to user accounts on an HTTP service. It separates the role of the resource owner (user), the client application, the authorization server, and the resource server. OAuth2 defines several grant types for different use cases: authorization code (web apps), implicit (SPAs, deprecated), client credentials (server-to-server), password (legacy), and device code (TVs, CLI tools).

### Why It Is Important
OAuth2 is the industry standard for delegated authorization. It allows users to grant third-party applications access to their resources without sharing credentials. Services like Google, GitHub, Facebook, and Twitter use OAuth2 for their authentication APIs. Understanding OAuth2 is essential for integrating with these providers and building secure authorization systems.

### How It Works Internally (Authorization Code Flow)
1. The client redirects the user to the authorization server with `client_id`, `redirect_uri`, `scope`, and `state`.
2. The user authenticates and consents to the requested permissions.
3. The authorization server redirects back with an authorization code.
4. The client exchanges the code (plus `client_secret`) for an access token and optionally a refresh token.
5. The client uses the access token to call the resource server's APIs.

```python
from fastapi import FastAPI, HTTPException, Request
from fastapi.responses import RedirectResponse
import httpx
import secrets

app = FastAPI()

# OAuth2 configuration
CLIENT_ID = "your-client-id"
CLIENT_SECRET = "your-client-secret"
REDIRECT_URI = "http://localhost:8000/callback"
AUTHORIZATION_URL = "https://github.com/login/oauth/authorize"
TOKEN_URL = "https://github.com/login/oauth/access_token"
USERINFO_URL = "https://api.github.com/user"
SCOPES = ["read:user", "user:email"]

# In-memory state store (use Redis in production)
state_store: dict = {}

@app.get("/login/github")
async def login_github():
    state = secrets.token_urlsafe(32)
    state_store[state] = True
    params = {
        "client_id": CLIENT_ID,
        "redirect_uri": REDIRECT_URI,
        "scope": " ".join(SCOPES),
        "state": state,
    }
    url = f"{AUTHORIZATION_URL}?{'&'.join(f'{k}={v}' for k, v in params.items())}"
    return RedirectResponse(url)

@app.get("/callback")
async def callback(code: str = None, state: str = None, error: str = None):
    if error:
        raise HTTPException(400, f"Authorization failed: {error}")
    if not state or state not in state_store:
        raise HTTPException(400, "Invalid state (CSRF detected)")
    del state_store[state]

    # Exchange code for token
    async with httpx.AsyncClient() as client:
        token_response = await client.post(
            TOKEN_URL,
            data={
                "client_id": CLIENT_ID,
                "client_secret": CLIENT_SECRET,
                "code": code,
                "redirect_uri": REDIRECT_URI,
            },
            headers={"Accept": "application/json"},
        )
        token_data = token_response.json()

        if "access_token" not in token_data:
            raise HTTPException(400, "Failed to get access token")

        # Fetch user info
        user_response = await client.get(
            USERINFO_URL,
            headers={"Authorization": f"Bearer {token_data['access_token']}"}
        )
        user_data = user_response.json()

    # Create local session/JWT
    return {
        "user": user_data["login"],
        "email": user_data.get("email"),
        "token": token_data["access_token"],
    }
```

### OAuth2 with FastAPI's Security Utilities
```python
from fastapi import FastAPI, Depends, HTTPException
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel
from datetime import datetime, timedelta

app = FastAPI()
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = authenticate_user(form_data.username, form_data.password)
    if not user:
        raise HTTPException(401, "Incorrect username or password")
    access_token = create_access_token(data={"sub": user.username})
    return {"access_token": access_token, "token_type": "bearer"}

@app.get("/users/me")
async def read_users_me(token: str = Depends(oauth2_scheme)):
    payload = verify_token(token)
    user = get_user(payload["sub"])
    return user
```

## Session-based auth

### What It Is
Session-based authentication stores user authentication state on the server. When a user logs in, the server creates a session (a unique identifier stored in a database or cache), returns the session ID to the client (typically as a cookie), and validates it on each subsequent request.

### Why It Is Important
Session-based auth is simpler to implement for traditional server-rendered applications. The server has full control over sessions (can revoke immediately), and session data is not exposed to the client. It remains the standard approach for Django, Flask, and server-side rendered applications.

### Syntax
```python
from flask import Flask, session, redirect, url_for, request, flash
from functools import wraps
import secrets

app = Flask(__name__)
app.secret_key = "your-secret-key"

# Login
@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        username = request.form["username"]
        password = request.form["password"]
        user = authenticate(username, password)
        if user:
            session["user_id"] = user["id"]
            session["role"] = user["role"]
            session["csrf_token"] = secrets.token_hex(32)
            return redirect(url_for("dashboard"))
        flash("Invalid credentials")
    return render_template("login.html")

# Logout
@app.route("/logout")
def logout():
    session.clear()
    return redirect(url_for("login"))

# Auth decorator
def login_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if "user_id" not in session:
            return redirect(url_for("login"))
        return f(*args, **kwargs)
    return decorated_function

@app.route("/dashboard")
@login_required
def dashboard():
    return render_template("dashboard.html", user_id=session["user_id"])
```

### Session vs Token auth

| Aspect | Session-based | Token-based (JWT) |
|--------|---------------|-------------------|
| State | Server-side (DB/cache) | Client-side (token) |
| Scalability | Needs shared session store | Stateless, scales horizontally |
| Revocation | Instant (delete session) | Must wait for expiry or use blocklist |
| Mobile/cross-platform | Difficult (cookie handling) | Easy (Bearer header) |
| CSRF protection | Required (cookie-based) | Not needed (header-based) |
| Payload size | Small (session ID only) | Larger (all claims in token) |
| Common frameworks | Django, Flask (default) | FastAPI, DRF + JWT |

## Token-based auth

### What It Is
Token-based authentication uses self-contained tokens (typically JWT) that the client presents with each request. The server validates the token's signature and expiration without accessing a database. This contrasts with session-based auth where the server must look up session state.

### Implementation
```python
from fastapi import FastAPI, Depends, HTTPException
from fastapi.security import HTTPBearer
import jwt
from datetime import datetime, timedelta

app = FastAPI()
security = HTTPBearer()

class TokenAuth:
    def __init__(self, secret_key: str, algorithm: str = "HS256"):
        self.secret_key = secret_key
        self.algorithm = algorithm

    def encode(self, payload: dict, expires_in: int = 3600) -> str:
        payload.update({
            "exp": datetime.utcnow() + timedelta(seconds=expires_in),
            "iat": datetime.utcnow(),
        })
        return jwt.encode(payload, self.secret_key, algorithm=self.algorithm)

    def decode(self, token: str) -> dict:
        try:
            return jwt.decode(token, self.secret_key, algorithms=[self.algorithm])
        except jwt.ExpiredSignatureError:
            raise HTTPException(401, "Token expired")
        except jwt.InvalidTokenError:
            raise HTTPException(401, "Invalid token")

auth = TokenAuth("secret-key")

@app.post("/api/login")
def login(username: str, password: str):
    user = verify_credentials(username, password)
    if not user:
        raise HTTPException(401, "Invalid credentials")
    token = auth.encode({
        "sub": user["id"],
        "role": user["role"],
        "permissions": user["permissions"],
    })
    return {"access_token": token, "token_type": "bearer"}

@app.get("/api/protected")
def protected(token: str = Depends(security)):
    payload = auth.decode(token.credentials)
    return {"user_id": payload["sub"], "role": payload["role"]}
```

## Password hashing (bcrypt)

### What It Is
Password hashing converts a plaintext password into a one-way cryptographic hash that cannot be reversed. Bcrypt is the most widely recommended password hashing algorithm. It incorporates a salt (random data to prevent rainbow table attacks) and a configurable work factor (cost) that makes brute-force attacks increasingly expensive as hardware improves.

### Why It Is Important
Storing passwords in plaintext is one of the most critical security vulnerabilities. Even hashing with fast algorithms like MD5 or SHA-256 is insufficient because they can be brute-forced at billions of hashes per second. Bcrypt's adaptive cost factor ensures that password hashing remains slow even as hardware improves.

### How It Works Internally
Bcrypt uses the Blowfish cipher to create a hash. It takes the password, a randomly generated salt (16 bytes), and a cost factor (typically 10-14). The cost factor determines the number of key expansion rounds (2^cost). Each round is computationally expensive, making brute-force attacks impractical. The output is a 60-character string containing the algorithm identifier, cost, salt, and hash.

### Syntax
```python
import bcrypt

def hash_password(password: str) -> str:
    # Generate salt and hash
    salt = bcrypt.gensalt(rounds=12)  # Cost factor: 12
    hashed = bcrypt.hashpw(password.encode("utf-8"), salt)
    return hashed.decode("utf-8")

def verify_password(password: str, hashed: str) -> bool:
    return bcrypt.checkpw(
        password.encode("utf-8"),
        hashed.encode("utf-8")
    )

# Usage
hashed = hash_password("user_password123")
print(hashed)  # $2b$12$...

is_valid = verify_password("user_password123", hashed)
print(is_valid)  # True

is_valid = verify_password("wrong_password", hashed)
print(is_valid)  # False
```

### Intermediate Examples
```python
import bcrypt
from flask import Flask, request, jsonify
from functools import wraps

app = Flask(__name__)

# User model with hashed password
class User:
    def __init__(self, username: str, password: str):
        self.username = username
        self.password_hash = self._hash_password(password)
        self.failed_attempts = 0
        self.locked_until = None

    def _hash_password(self, password: str) -> str:
        return bcrypt.hashpw(
            password.encode("utf-8"),
            bcrypt.gensalt(rounds=12)
        ).decode("utf-8")

    def check_password(self, password: str) -> bool:
        return bcrypt.checkpw(
            password.encode("utf-8"),
            self.password_hash.encode("utf-8")
        )

# Account lockout after failed attempts
users_db = {}

@app.route("/register", methods=["POST"])
def register():
    data = request.get_json()
    if data["username"] in users_db:
        return jsonify({"error": "User exists"}), 409
    users_db[data["username"]] = User(data["username"], data["password"])
    return jsonify({"message": "Created"}), 201

@app.route("/login", methods=["POST"])
def login():
    data = request.get_json()
    user = users_db.get(data["username"])
    if not user:
        return jsonify({"error": "Invalid credentials"}), 401

    # Check account lockout
    if user.locked_until and datetime.utcnow() < user.locked_until:
        return jsonify({"error": "Account locked"}), 423

    if user.check_password(data["password"]):
        user.failed_attempts = 0
        return jsonify({"token": create_token(user.username)})

    # Increment failed attempts
    user.failed_attempts += 1
    if user.failed_attempts >= 5:
        user.locked_until = datetime.utcnow() + timedelta(minutes=15)
        return jsonify({"error": "Account locked for 15 minutes"}), 423

    return jsonify({"error": "Invalid credentials"}), 401

# Password validation policy
def validate_password_strength(password: str) -> tuple:
    errors = []
    if len(password) < 8:
        errors.append("At least 8 characters")
    if not any(c.isupper() for c in password):
        errors.append("At least one uppercase letter")
    if not any(c.islower() for c in password):
        errors.append("At least one lowercase letter")
    if not any(c.isdigit() for c in password):
        errors.append("At least one digit")
    if not any(c in "!@#$%^&*" for c in password):
        errors.append("At least one special character")
    if errors:
        return False, errors
    return True, []
```

### Advanced Examples - Password Hashing with Werkzeug
```python
from werkzeug.security import generate_password_hash, check_password_hash

# Werkzeug uses pbkdf2:sha256 by default (configurable)
hashed = generate_password_hash("my_password")
# Format: pbkdf2:sha256:260000$salt$hash

is_valid = check_password_hash(hashed, "my_password")
print(is_valid)  # True

# Using passlib (supports bcrypt, argon2, scrypt)
from passlib.context import CryptContext

pwd_context = CryptContext(
    schemes=["bcrypt", "argon2", "pbkdf2_sha256"],
    default="bcrypt",
    bcrypt__rounds=12,
)

hashed = pwd_context.hash("my_password")
is_valid = pwd_context.verify("my_password", hashed)

# Automatic hash upgrade (rehash if scheme changes)
def verify_and_upgrade(password: str, hashed: str) -> tuple:
    valid = pwd_context.verify(password, hashed)
    needs_rehash = pwd_context.needs_update(hashed)
    return valid, needs_rehash
```

### Real-World Use Cases
- **Web application authentication**: Login systems for Flask, Django, FastAPI apps.
- **API authentication**: JWT Bearer tokens for REST APIs.
- **Single sign-on (SSO)**: OAuth2 with providers (Google, GitHub, Azure AD).
- **Mobile app authentication**: Token-based auth with refresh tokens.
- **Third-party API access**: OAuth2 client credentials for service accounts.

### Common Mistakes
- Storing passwords in plaintext or with weak hashes (MD5, SHA-1).
- Using low bcrypt cost (rounds < 10), making brute-force feasible.
- Not using HTTPS (credentials sent in clear text).
- Storing JWTs in localStorage (vulnerable to XSS).
- Token expiration set too long (weeks or months without refresh).
- Not implementing account lockout or rate limiting on login.
- Missing CSRF protection for cookie-based session auth.

### Best Practices
- Always use bcrypt (or argon2) for password hashing, never SHA/MD5.
- Set bcrypt cost factor high enough for your hardware (12-14 target ~250ms).
- Use JWT access tokens with short expiration (15-60 minutes).
- Implement refresh tokens for long-lived sessions.
- Store refresh tokens securely (HTTP-only cookies, not localStorage).
- Use HTTPS exclusively for all authentication endpoints.
- Implement rate limiting on login endpoints.
- Use CSRF tokens for session-based auth with cookies.
- Validate all input and never trust client-side data.

### Performance Considerations
- Bcrypt cost factor 12 takes ~250ms; factor 14 takes ~1 second. Choose based on traffic.
- JWT verification is fast (~1-5 microseconds) since it's cryptographic signature verification.
- Token blocklists (for immediate revocation) require database/cache lookups, compromising statelessness.
- Session stores should use Redis/memcached for fast lookups, not SQL databases.

### Interview Questions
1. What is the difference between session-based and token-based authentication?
2. How does bcrypt work and why is it better than SHA-256 for passwords?
3. Explain the OAuth2 authorization code flow.
4. What is a JWT and what are its three parts?
5. How do you handle token refresh and revocation?
6. What is the difference between authentication and authorization?
7. How do you protect against CSRF attacks in session-based auth?

### Coding Challenges
1. **JWT Auth System**: Build a complete authentication system with login, protected routes, token refresh, and role-based access control.
2. **OAuth2 Integration**: Implement GitHub OAuth2 login in a FastAPI application with session persistence.
3. **Password Manager**: Create a password hashing utility with strength validation, bcrypt hashing, and automatic rehashing when cost factor increases.

### Related Topics
- Flask-Login (session-based auth for Flask)
- FastAPI Security utilities (OAuth2, JWT, API keys)
- OAuth2 scopes and permissions
- OpenID Connect (identity layer on top of OAuth2)
- WebAuthn/Passkeys (passwordless authentication)
