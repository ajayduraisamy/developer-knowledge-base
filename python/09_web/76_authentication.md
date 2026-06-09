# Authentication - JWT, OAuth2, session vs token, password hashing

## Introduction

Authentication is the process of verifying the identity of a user, system, or entity attempting to access a protected resource. In modern web applications and APIs, authentication is typically implemented using tokens (JWT), session cookies, OAuth2 flows, or API keys. Python provides robust libraries for implementing authentication — PyJWT for JSON Web Tokens, passlib/bcrypt/argon2 for password hashing, Authlib for OAuth2, and framework-specific solutions like FastAPI's OAuth2 support or Flask-Login. A secure authentication system is the foundation of any application that handles user data.

## Why It Is Important

Authentication is the first line of defense for any application. Weak authentication leads to account takeover, data breaches, and compliance violations. Modern authentication must balance security (strong password hashing, token expiration, multi-factor authentication) with user experience (social login, single sign-on, remember-me). Understanding the differences between session-based and token-based auth, OAuth2 flows, JWT structure and validation, and proper password storage is essential for building secure applications. Authentication also directly impacts scalability — stateless JWT tokens scale better than server-side sessions.

## Syntax

### JWT Token Creation and Verification

```python
import jwt
from datetime import datetime, timedelta

SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"

def create_token(user_id, role="user"):
    payload = {
        "sub": str(user_id),
        "role": role,
        "iat": datetime.utcnow(),
        "exp": datetime.utcnow() + timedelta(hours=1)
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

def verify_token(token):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except jwt.ExpiredSignatureError:
        return None
    except jwt.InvalidTokenError:
        return None
```

### Password Hashing with bcrypt

```python
import bcrypt

password = b"secure_password"
salt = bcrypt.gensalt()
hashed = bcrypt.hashpw(password, salt)

bcrypt.checkpw(password, hashed)
```

## Examples

### Basic JWT Authentication

```python
from flask import Flask, jsonify, request
from functools import wraps
import jwt
from datetime import datetime, timedelta

app = Flask(__name__)
app.config["SECRET_KEY"] = "your-secret-key"

def token_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        token = request.headers.get("Authorization", "").replace("Bearer ", "")
        if not token:
            return jsonify({"error": "Token is missing"}), 401
        try:
            data = jwt.decode(token, app.config["SECRET_KEY"], algorithms=["HS256"])
            request.user = data
        except jwt.ExpiredSignatureError:
            return jsonify({"error": "Token has expired"}), 401
        except jwt.InvalidTokenError:
            return jsonify({"error": "Token is invalid"}), 401
        return f(*args, **kwargs)
    return decorated

@app.route("/login", methods=["POST"])
def login():
    auth = request.get_json()
    if auth and auth.get("username") == "admin" and auth.get("password") == "secret":
        token = jwt.encode({
            "sub": "admin", "role": "admin",
            "exp": datetime.utcnow() + timedelta(hours=1)
        }, app.config["SECRET_KEY"])
        return jsonify({"token": token, "type": "bearer"})
    return jsonify({"error": "Invalid credentials"}), 401

@app.route("/protected")
@token_required
def protected():
    return jsonify({"message": "Access granted", "user": request.user})
```

## Beginner Examples

### 1. Simple Password Hashing

```python
import bcrypt

passwords = ["password123", "secure_pass!", "hello123"]

for pwd in passwords:
    hashed = bcrypt.hashpw(pwd.encode(), bcrypt.gensalt())
    print(f"Original: {pwd}")
    print(f"Hashed: {hashed}")
    print(f"Verify: {bcrypt.checkpw(pwd.encode(), hashed)}")
    print("---")
```

### 2. JWT Token with Claims

```python
import jwt
from datetime import datetime, timedelta

SECRET = "my-secret"
ALGO = "HS256"

payload = {
    "sub": "user_123",
    "name": "Alice",
    "role": "admin",
    "permissions": ["read", "write", "delete"],
    "iat": datetime.utcnow(),
    "exp": datetime.utcnow() + timedelta(minutes=30)
}

token = jwt.encode(payload, SECRET, algorithm=ALGO)
print(f"Token: {token[:50]}...")

decoded = jwt.decode(token, SECRET, algorithms=[ALGO])
print(f"Decoded: {decoded}")
```

### 3. Basic Auth with Flask

```python
from flask import Flask, jsonify, request
from functools import wraps

app = Flask(__name__)

users = {
    "alice": {"password": "pass123", "role": "user"},
    "bob": {"password": "pass456", "role": "user"},
    "admin": {"password": "admin123", "role": "admin"}
}

def authenticate(f):
    @wraps(f)
    def wrapper(*args, **kwargs):
        auth = request.authorization
        if not auth or not auth.username or not auth.password:
            return jsonify({"error": "Authentication required"}), 401
        user = users.get(auth.username)
        if not user or user["password"] != auth.password:
            return jsonify({"error": "Invalid credentials"}), 401
        request.user = {"username": auth.username, "role": user["role"]}
        return f(*args, **kwargs)
    return wrapper

@app.route("/api/profile")
@authenticate
def profile():
    return jsonify({"profile": request.user})

@app.route("/api/admin")
@authenticate
def admin():
    if request.user["role"] != "admin":
        return jsonify({"error": "Admin access required"}), 403
    return jsonify({"message": "Admin panel"})
```

### 4. Session-Based Auth

```python
from flask import Flask, jsonify, request, session
import uuid

app = Flask(__name__)
app.secret_key = "secret-key-123"

sessions_db = {}
users_db = {"alice": {"password": "pass123", "name": "Alice"}}

@app.route("/login", methods=["POST"])
def login():
    data = request.get_json()
    username = data.get("username")
    password = data.get("password")
    user = users_db.get(username)
    if not user or user["password"] != password:
        return jsonify({"error": "Invalid credentials"}), 401
    session_id = str(uuid.uuid4())
    sessions_db[session_id] = {"username": username, "name": user["name"]}
    session["session_id"] = session_id
    return jsonify({"message": "Logged in", "session_id": session_id})

@app.route("/profile")
def profile():
    session_id = session.get("session_id")
    if not session_id or session_id not in sessions_db:
        return jsonify({"error": "Not authenticated"}), 401
    return jsonify({"user": sessions_db[session_id]})

@app.route("/logout")
def logout():
    session_id = session.pop("session_id", None)
    if session_id:
        sessions_db.pop(session_id, None)
    return jsonify({"message": "Logged out"})
```

### 5. Token Refresh

```python
import jwt
from datetime import datetime, timedelta
from flask import Flask, jsonify, request

app = Flask(__name__)
SECRET = "your-secret"
REFRESH_SECRET = "refresh-secret"

def create_access_token(user_id):
    return jwt.encode({"sub": user_id, "type": "access", "exp": datetime.utcnow() + timedelta(minutes=15)}, SECRET, algorithm="HS256")

def create_refresh_token(user_id):
    return jwt.encode({"sub": user_id, "type": "refresh", "exp": datetime.utcnow() + timedelta(days=7)}, REFRESH_SECRET, algorithm="HS256")

@app.route("/login", methods=["POST"])
def login():
    data = request.get_json()
    if data.get("username") == "alice" and data.get("password") == "pass":
        return jsonify({
            "access_token": create_access_token("user_1"),
            "refresh_token": create_refresh_token("user_1"),
            "token_type": "bearer"
        })
    return jsonify({"error": "Invalid"}), 401

@app.route("/refresh", methods=["POST"])
def refresh():
    refresh_token = request.json.get("refresh_token")
    try:
        payload = jwt.decode(refresh_token, REFRESH_SECRET, algorithms=["HS256"])
        if payload.get("type") != "refresh":
            return jsonify({"error": "Invalid token type"}), 401
        return jsonify({"access_token": create_access_token(payload["sub"])})
    except jwt.ExpiredSignatureError:
        return jsonify({"error": "Refresh token expired"}), 401
    except jwt.InvalidTokenError:
        return jsonify({"error": "Invalid refresh token"}), 401
```

## Intermediate Examples

### 1. OAuth2 with FastAPI

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel
from jose import JWTError, jwt
from passlib.context import CryptContext
from datetime import datetime, timedelta
from typing import Optional

SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

app = FastAPI()
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

fake_users_db = {
    "alice": {
        "username": "alice", "full_name": "Alice Smith", "email": "alice@example.com",
        "hashed_password": pwd_context.hash("secret"), "disabled": False
    }
}

class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    username: Optional[str] = None

class User(BaseModel):
    username: str
    email: str
    full_name: str
    disabled: bool

class UserInDB(User):
    hashed_password: str

def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)

def get_user(db, username: str):
    if username in db:
        return UserInDB(**db[username])
    return None

def authenticate_user(fake_db, username: str, password: str):
    user = get_user(fake_db, username)
    if not user or not verify_password(password, user.hashed_password):
        return False
    return user

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=15))
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

async def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Could not validate credentials", headers={"WWW-Authenticate": "Bearer"})
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username = payload.get("sub")
        if username is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception
    user = get_user(fake_users_db, username)
    if user is None:
        raise credentials_exception
    return user

async def get_current_active_user(current_user: User = Depends(get_current_user)):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user

@app.post("/token", response_model=Token)
async def login_for_access_token(form_data: OAuth2PasswordRequestForm = Depends()):
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Incorrect username or password", headers={"WWW-Authenticate": "Bearer"})
    access_token = create_access_token(data={"sub": user.username}, expires_delta=timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES))
    return {"access_token": access_token, "token_type": "bearer"}

@app.get("/users/me", response_model=User)
async def read_users_me(current_user: User = Depends(get_current_active_user)):
    return current_user
```

### 2. OAuth2 Authorization Code Flow

```python
import requests
from flask import Flask, jsonify, request, redirect, session
import secrets

app = Flask(__name__)
app.secret_key = "secret-key"

CLIENT_ID = "your-client-id"
CLIENT_SECRET = "your-client-secret"
REDIRECT_URI = "http://localhost:5000/callback"
AUTHORIZE_URL = "https://provider.com/oauth/authorize"
TOKEN_URL = "https://provider.com/oauth/token"
USER_INFO_URL = "https://provider.com/userinfo"

@app.route("/login")
def login():
    state = secrets.token_urlsafe(16)
    session["oauth_state"] = state
    params = f"client_id={CLIENT_ID}&response_type=code&redirect_uri={REDIRECT_URI}&state={state}&scope=openid%20profile%20email"
    return redirect(f"{AUTHORIZE_URL}?{params}")

@app.route("/callback")
def callback():
    error = request.args.get("error")
    if error:
        return jsonify({"error": error}), 400
    code = request.args.get("code")
    state = request.args.get("state")
    if state != session.get("oauth_state"):
        return jsonify({"error": "State mismatch"}), 400
    token_data = {"grant_type": "authorization_code", "code": code, "redirect_uri": REDIRECT_URI, "client_id": CLIENT_ID, "client_secret": CLIENT_SECRET}
    token_response = requests.post(TOKEN_URL, data=token_data)
    if token_response.status_code != 200:
        return jsonify({"error": "Failed to get token"}), 400
    tokens = token_response.json()
    user_response = requests.get(USER_INFO_URL, headers={"Authorization": f"Bearer {tokens['access_token']}"})
    return jsonify({"message": "Login successful", "user": user_response.json()})
```

### 3. Password Hashing with Argon2

```python
from argon2 import PasswordHasher
from argon2.exceptions import VerifyMismatchError

ph = PasswordHasher(time_cost=3, memory_cost=65536, parallelism=4, hash_len=32, salt_len=16)

passwords = ["CorrectHorseBatteryStaple", "password123", "LetMeIn!"]

hashed_passwords = {}
for pwd in passwords:
    hashed = ph.hash(pwd)
    hashed_passwords[pwd] = hashed
    print(f"Hashed: {hashed[:60]}...")

for pwd in passwords:
    stored = hashed_passwords[pwd]
    try:
        ph.verify(stored, pwd)
        print(f"Verified: {pwd[:10]}... -> OK")
        if ph.check_needs_rehash(stored):
            print("  Password hash needs rehash!")
            new_hash = ph.hash(pwd)
            hashed_passwords[pwd] = new_hash
    except VerifyMismatchError:
        print(f"Verified: {pwd[:10]}... -> FAILED")
```

### 4. Multi-Factor Authentication Concepts

```python
import pyotp
import qrcode
from io import BytesIO
import base64

class MFAHandler:
    def __init__(self, issuer="MyApp"):
        self.issuer = issuer

    def generate_secret(self):
        return pyotp.random_base32()

    def get_provisioning_uri(self, username, secret):
        return pyotp.totp.TOTP(secret).provisioning_uri(name=username, issuer_name=self.issuer)

    def generate_qr_b64(self, username, secret):
        uri = self.get_provisioning_uri(username, secret)
        qr = qrcode.make(uri)
        buffer = BytesIO()
        qr.save(buffer, format="PNG")
        return base64.b64encode(buffer.getvalue()).decode()

    def verify_totp(self, secret, token):
        return pyotp.TOTP(secret).verify(token, valid_window=1)

    def generate_backup_codes(self, count=8):
        import secrets
        return [secrets.token_hex(4) for _ in range(count)]

mfa = MFAHandler()
secret = mfa.generate_secret()
username = "alice@example.com"
print(f"Secret: {secret}")
print(f"Backup codes: {mfa.generate_backup_codes()}")

totp = pyotp.TOTP(secret)
current_token = totp.now()
print(f"Current TOTP: {current_token}")
print(f"Verify: {mfa.verify_totp(secret, current_token)}")
```

### 5. Role-Based Access Control (RBAC)

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import jwt, JWTError
from pydantic import BaseModel
from enum import Enum
from typing import List

app = FastAPI()
security = HTTPBearer()
SECRET_KEY = "secret-key"

class Role(str, Enum):
    ADMIN = "admin"
    MODERATOR = "moderator"
    USER = "user"

class Permission(str, Enum):
    READ = "read"
    WRITE = "write"
    DELETE = "delete"
    MANAGE_USERS = "manage_users"

ROLE_PERMISSIONS = {
    Role.ADMIN: [Permission.READ, Permission.WRITE, Permission.DELETE, Permission.MANAGE_USERS],
    Role.MODERATOR: [Permission.READ, Permission.WRITE, Permission.DELETE],
    Role.USER: [Permission.READ]
}

def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)):
    token = credentials.credentials
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        return payload
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")

def require_permission(required_permission: Permission):
    def permission_checker(current_user: dict = Depends(get_current_user)):
        user_role = Role(current_user.get("role", "user"))
        user_permissions = ROLE_PERMISSIONS.get(user_role, [])
        if required_permission not in user_permissions:
            raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="Insufficient permissions")
        return current_user
    return permission_checker

@app.get("/api/public")
def public():
    return {"message": "Public endpoint"}

@app.get("/api/read")
def read_endpoint(user: dict = Depends(require_permission(Permission.READ))):
    return {"message": f"Read access granted to {user.get('sub')}"}

@app.post("/api/write")
def write_endpoint(user: dict = Depends(require_permission(Permission.WRITE))):
    return {"message": f"Write access granted to {user.get('sub')}"}
```

### 6. Social Auth with Authlib

```python
from authlib.integrations.flask_client import OAuth
from flask import Flask, jsonify, session, url_for

app = Flask(__name__)
app.secret_key = "secret"

oauth = OAuth(app)

oauth.register(
    name="google",
    client_id="your-google-client-id",
    client_secret="your-google-client-secret",
    server_metadata_url="https://accounts.google.com/.well-known/openid-configuration",
    client_kwargs={"scope": "openid email profile"}
)

oauth.register(
    name="github",
    client_id="your-github-client-id",
    client_secret="your-github-client-secret",
    access_token_url="https://github.com/login/oauth/access_token",
    authorize_url="https://github.com/login/oauth/authorize",
    api_base_url="https://api.github.com/",
    client_kwargs={"scope": "user:email"}
)

@app.route("/login/google")
def login_google():
    return oauth.google.authorize_redirect(url_for("authorize_google", _external=True))

@app.route("/authorize/google")
def authorize_google():
    token = oauth.google.authorize_access_token()
    user_info = oauth.google.parse_id_token(token)
    session["user"] = {"provider": "google", "email": user_info.get("email"), "name": user_info.get("name")}
    return jsonify(session["user"])

@app.route("/login/github")
def login_github():
    return oauth.github.authorize_redirect(url_for("authorize_github", _external=True))

@app.route("/authorize/github")
def authorize_github():
    token = oauth.github.authorize_access_token()
    resp = oauth.github.get("user", token=token)
    user_info = resp.json()
    session["user"] = {"provider": "github", "login": user_info.get("login"), "name": user_info.get("name")}
    return jsonify(session["user"])

@app.route("/profile")
def profile():
    user = session.get("user")
    if not user:
        return jsonify({"error": "Not logged in"}), 401
    return jsonify(user)

@app.route("/logout")
def logout():
    session.pop("user", None)
    return jsonify({"message": "Logged out"})
```

### 7. API Key Authentication

```python
from fastapi import FastAPI, HTTPException, Depends, Security
from fastapi.security import APIKeyHeader, APIKeyQuery
from typing import Optional

app = FastAPI()

api_keys_db = {
    "key_user1": {"name": "Alice", "role": "read"},
    "key_user2": {"name": "Bob", "role": "write"},
    "key_admin": {"name": "Admin", "role": "admin"}
}

api_key_header = APIKeyHeader(name="X-API-Key", auto_error=False)
api_key_query = APIKeyQuery(name="api_key", auto_error=False)

async def get_api_key(header_key: Optional[str] = Security(api_key_header), query_key: Optional[str] = Security(api_key_query)):
    api_key = header_key or query_key
    if not api_key:
        raise HTTPException(status_code=401, detail="API key is missing")
    if api_key not in api_keys_db:
        raise HTTPException(status_code=403, detail="Invalid API key")
    return api_keys_db[api_key]

def require_role(required_role: str):
    async def role_checker(api_key: dict = Depends(get_api_key)):
        if api_key["role"] == "admin":
            return api_key
        if api_key["role"] == required_role:
            return api_key
        if required_role == "read" and api_key["role"] in ("read", "write"):
            return api_key
        raise HTTPException(status_code=403, detail=f"{required_role} access required")
    return role_checker

@app.get("/api/read")
async def read_data(api_key: dict = Depends(require_role("read"))):
    return {"message": f"Read access granted to {api_key['name']}"}

@app.post("/api/write")
async def write_data(api_key: dict = Depends(require_role("write"))):
    return {"message": f"Write access granted to {api_key['name']}"}
```

### 8. Session vs Token Auth Comparison

```python
from flask import Flask, jsonify, request, session
from functools import wraps
import jwt
import uuid
from datetime import datetime, timedelta

app = Flask(__name__)
app.secret_key = "session-secret"
JWT_SECRET = "jwt-secret"

sessions_db = {}
users_db = {"alice": "pass123"}

def session_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        session_id = session.get("session_id")
        if not session_id or session_id not in sessions_db:
            return jsonify({"error": "Not authenticated"}), 401
        request.user = sessions_db[session_id]
        return f(*args, **kwargs)
    return decorated

@app.route("/session/login", methods=["POST"])
def session_login():
    data = request.get_json()
    user = data.get("username")
    pwd = data.get("password")
    if user in users_db and users_db[user] == pwd:
        session_id = str(uuid.uuid4())
        sessions_db[session_id] = {"username": user}
        session["session_id"] = session_id
        return jsonify({"message": "Session login successful"})
    return jsonify({"error": "Invalid"}), 401

@app.route("/session/profile")
@session_required
def session_profile():
    return jsonify({"auth_type": "session", "user": request.user})

@app.route("/session/logout")
def session_logout():
    session_id = session.pop("session_id", None)
    if session_id:
        sessions_db.pop(session_id, None)
    return jsonify({"message": "Logged out"})

def token_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        token = request.headers.get("Authorization", "").replace("Bearer ", "")
        if not token:
            return jsonify({"error": "No token"}), 401
        try:
            payload = jwt.decode(token, JWT_SECRET, algorithms=["HS256"])
            request.user = payload
        except jwt.ExpiredSignatureError:
            return jsonify({"error": "Token expired"}), 401
        except jwt.InvalidTokenError:
            return jsonify({"error": "Invalid token"}), 401
        return f(*args, **kwargs)
    return decorated

@app.route("/token/login", methods=["POST"])
def token_login():
    data = request.get_json()
    if data.get("username") == "alice" and data.get("password") == "pass123":
        token = jwt.encode({"sub": "alice", "exp": datetime.utcnow() + timedelta(hours=1)}, JWT_SECRET, algorithm="HS256")
        return jsonify({"token": token, "type": "bearer"})
    return jsonify({"error": "Invalid"}), 401

@app.route("/token/profile")
@token_required
def token_profile():
    return jsonify({"auth_type": "token", "user": request.user})
```

### 9. Secure Cookie Management

```python
from flask import Flask, jsonify, make_response, request
import jwt
from datetime import datetime, timedelta

app = Flask(__name__)
app.config["SECRET_KEY"] = "your-secret-key"

@app.route("/login/secure")
def login_secure():
    token = jwt.encode({"sub": "user_123", "exp": datetime.utcnow() + timedelta(hours=1)}, app.config["SECRET_KEY"], algorithm="HS256")
    response = make_response(jsonify({"message": "Logged in"}))
    response.set_cookie("session_token", token, httponly=True, secure=True, samesite="Lax", max_age=3600)
    return response

@app.route("/profile/secure")
def profile_secure():
    token = request.cookies.get("session_token")
    if not token:
        return jsonify({"error": "Not authenticated"}), 401
    try:
        data = jwt.decode(token, app.config["SECRET_KEY"], algorithms=["HS256"])
        return jsonify({"user": data})
    except jwt.InvalidTokenError:
        return jsonify({"error": "Invalid token"}), 401

@app.route("/logout/secure")
def logout_secure():
    response = make_response(jsonify({"message": "Logged out"}))
    response.set_cookie("session_token", "", expires=0, httponly=True, secure=True)
    return response
```

### 10. Full Authentication System

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel, EmailStr
from jose import JWTError, jwt
from passlib.context import CryptContext
from datetime import datetime, timedelta
from typing import Optional
import enum

SECRET_KEY = "super-secret-key-change-in-production"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE = 30
REFRESH_TOKEN_EXPIRE = 7

app = FastAPI(title="Auth System")
pwd_context = CryptContext(schemes=["bcrypt", "argon2"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/token")

class UserRole(str, enum.Enum):
    ADMIN = "admin"
    USER = "user"
    MODERATOR = "moderator"

class UserCreate(BaseModel):
    username: str
    email: EmailStr
    password: str
    full_name: str

class UserResponse(BaseModel):
    username: str
    email: str
    full_name: str
    role: UserRole
    disabled: bool

class UserInDB(UserResponse):
    hashed_password: str

class Token(BaseModel):
    access_token: str
    refresh_token: str
    token_type: str


users_db = {}
refresh_tokens_db = {}

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def get_user(username: str) -> Optional[UserInDB]:
    if username in users_db:
        return UserInDB(**users_db[username])
    return None

def authenticate_user(username: str, password: str):
    user = get_user(username)
    if not user or not verify_password(password, user.hashed_password):
        return False
    return user

def create_token(data: dict, expires_delta: timedelta):
    to_encode = data.copy()
    to_encode["exp"] = datetime.utcnow() + expires_delta
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def create_tokens(username: str):
    access = create_token({"sub": username, "type": "access"}, timedelta(minutes=ACCESS_TOKEN_EXPIRE))
    refresh = create_token({"sub": username, "type": "refresh"}, timedelta(days=REFRESH_TOKEN_EXPIRE))
    return access, refresh

async def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Could not validate credentials", headers={"WWW-Authenticate": "Bearer"})
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username = payload.get("sub")
        if username is None or payload.get("type") != "access":
            raise credentials_exception
    except JWTError:
        raise credentials_exception
    user = get_user(username)
    if user is None:
        raise credentials_exception
    return user

async def get_current_active_user(current_user: UserInDB = Depends(get_current_user)):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user

@app.post("/auth/register", response_model=UserResponse, status_code=201)
def register(user_data: UserCreate):
    if get_user(user_data.username):
        raise HTTPException(status_code=409, detail="Username already exists")
    hashed = hash_password(user_data.password)
    user_dict = user_data.model_dump()
    user_dict.pop("password")
    user_dict["hashed_password"] = hashed
    user_dict["role"] = UserRole.USER
    user_dict["disabled"] = False
    users_db[user_data.username] = user_dict
    return UserResponse(**user_dict)

@app.post("/auth/token", response_model=Token)
def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = authenticate_user(form_data.username, form_data.password)
    if not user:
        raise HTTPException(status_code=401, detail="Incorrect username or password")
    access_token, refresh_token = create_tokens(user.username)
    refresh_tokens_db[refresh_token] = user.username
    return Token(access_token=access_token, refresh_token=refresh_token, token_type="bearer")

@app.post("/auth/refresh", response_model=Token)
def refresh_token(refresh_token: str):
    if refresh_token not in refresh_tokens_db:
        raise HTTPException(status_code=401, detail="Invalid refresh token")
    try:
        payload = jwt.decode(refresh_token, SECRET_KEY, algorithms=[ALGORITHM])
        if payload.get("type") != "refresh":
            raise HTTPException(status_code=401, detail="Invalid token type")
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid refresh token")
    username = payload.get("sub")
    user = get_user(username)
    if not user:
        raise HTTPException(status_code=401, detail="User not found")
    del refresh_tokens_db[refresh_token]
    new_access, new_refresh = create_tokens(username)
    refresh_tokens_db[new_refresh] = username
    return Token(access_token=new_access, refresh_token=new_refresh, token_type="bearer")

@app.get("/auth/me", response_model=UserResponse)
def get_profile(current_user: UserInDB = Depends(get_current_active_user)):
    return UserResponse(**current_user.model_dump())

@app.post("/auth/logout")
def logout(refresh_token: str, current_user: UserInDB = Depends(get_current_active_user)):
    refresh_tokens_db.pop(refresh_token, None)
    return {"message": "Logged out"}
```

## Advanced Examples

### 1. Passwordless Authentication

```python
from fastapi import FastAPI, HTTPException, BackgroundTasks
from pydantic import BaseModel, EmailStr
import secrets
import hashlib
import time

app = FastAPI()
magic_links_db = {}

class RequestMagicLink(BaseModel):
    email: EmailStr

class VerifyMagicLink(BaseModel):
    token: str

@app.post("/auth/magic-link")
async def request_magic_link(data: RequestMagicLink, background_tasks: BackgroundTasks):
    token = secrets.token_urlsafe(32)
    token_hash = hashlib.sha256(token.encode()).hexdigest()
    magic_links_db[token_hash] = {"email": data.email, "expires_at": time.time() + 900, "used": False}
    async def send_email(email: str, token: str):
        link = f"http://localhost:8000/auth/verify?token={token}"
        print(f"Magic link for {email}: {link}")
    background_tasks.add_task(send_email, data.email, token)
    return {"message": "Magic link sent", "email": data.email}

@app.post("/auth/verify")
async def verify_magic_link(data: VerifyMagicLink):
    token_hash = hashlib.sha256(data.token.encode()).hexdigest()
    stored = magic_links_db.get(token_hash)
    if not stored:
        raise HTTPException(status_code=400, detail="Invalid token")
    if stored["used"]:
        raise HTTPException(status_code=400, detail="Token already used")
    if time.time() > stored["expires_at"]:
        raise HTTPException(status_code=400, detail="Token expired")
    stored["used"] = True
    from jose import jwt
    from datetime import datetime, timedelta
    access_token = jwt.encode({"sub": stored["email"], "type": "access", "exp": datetime.utcnow() + timedelta(hours=1)}, "secret", algorithm="HS256")
    return {"access_token": access_token, "token_type": "bearer"}
```

### 2. OAuth2 with Social Providers

```python
from fastapi import FastAPI, HTTPException
from httpx import AsyncClient
from pydantic import BaseModel
from jose import jwt
from datetime import datetime, timedelta

app = FastAPI()

class SocialLoginRequest(BaseModel):
    provider: str
    access_token: str

async def verify_google_token(token: str):
    async with AsyncClient() as client:
        resp = await client.get("https://www.googleapis.com/oauth2/v3/userinfo", headers={"Authorization": f"Bearer {token}"})
        if resp.status_code != 200:
            raise HTTPException(status_code=401, detail="Invalid Google token")
        data = resp.json()
        return {"provider": "google", "sub": data["sub"], "email": data.get("email"), "name": data.get("name")}

async def verify_github_token(token: str):
    async with AsyncClient() as client:
        resp = await client.get("https://api.github.com/user", headers={"Authorization": f"Bearer {token}"})
        if resp.status_code != 200:
            raise HTTPException(status_code=401, detail="Invalid GitHub token")
        data = resp.json()
        return {"provider": "github", "sub": str(data["id"]), "email": data.get("email"), "name": data.get("name") or data["login"]}

@app.post("/auth/social-login")
async def social_login(request: SocialLoginRequest):
    if request.provider == "google":
        user_info = await verify_google_token(request.access_token)
    elif request.provider == "github":
        user_info = await verify_github_token(request.access_token)
    else:
        raise HTTPException(status_code=400, detail=f"Unsupported provider: {request.provider}")
    token = jwt.encode({"sub": user_info["sub"], "provider": request.provider, "email": user_info.get("email"), "exp": datetime.utcnow() + timedelta(hours=24)}, "secret", algorithm="HS256")
    return {"access_token": token, "token_type": "bearer", "user": user_info}
```

### 3. JWT with Asymmetric Keys (RS256)

```python
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.backends import default_backend
from jose import jwt
from datetime import datetime, timedelta
import json

class AsymmetricJWTManager:
    def __init__(self):
        self.private_key = rsa.generate_private_key(public_exponent=65537, key_size=2048, backend=default_backend())
        self.public_key = self.private_key.public_key()
        self.private_pem = self.private_key.private_bytes(encoding=serialization.Encoding.PEM, format=serialization.PrivateFormat.PKCS8, encryption_algorithm=serialization.NoEncryption())
        self.public_pem = self.public_key.public_bytes(encoding=serialization.Encoding.PEM, format=serialization.PublicFormat.SubjectPublicKeyInfo)

    def create_token(self, subject: str, role: str = "user") -> str:
        payload = {"sub": subject, "role": role, "iat": datetime.utcnow(), "exp": datetime.utcnow() + timedelta(hours=1)}
        return jwt.encode(payload, self.private_pem, algorithm="RS256")

    def verify_token(self, token: str) -> dict:
        try:
            return jwt.decode(token, self.public_pem, algorithms=["RS256"])
        except Exception:
            return None

    def get_jwks(self) -> dict:
        public_numbers = self.public_key.public_numbers()
        import base64
        def int_to_base64url(num):
            num_bytes = num.to_bytes((num.bit_length() + 7) // 8, byteorder='big')
            return base64.urlsafe_b64encode(num_bytes).rstrip(b'=').decode()
        return {"keys": [{"kty": "RSA", "use": "sig", "alg": "RS256", "n": int_to_base64url(public_numbers.n), "e": int_to_base64url(public_numbers.e)}]}

jwt_manager = AsymmetricJWTManager()
token = jwt_manager.create_token("user_123", role="admin")
print(f"RS256 Token: {token[:60]}...")
decoded = jwt_manager.verify_token(token)
print(f"Decoded: {decoded}")
print(f"JWKS: {json.dumps(jwt_manager.get_jwks(), indent=2)}")
```

### 4. Secure Password Reset Flow

```python
from fastapi import FastAPI, HTTPException, BackgroundTasks
from pydantic import BaseModel, EmailStr
import secrets
import hashlib
import time

app = FastAPI()
password_reset_tokens = {}
users_db = {"alice@example.com": {"password_hash": "hashed_placeholder", "name": "Alice"}}

class ForgotPasswordRequest(BaseModel):
    email: EmailStr

class ResetPasswordRequest(BaseModel):
    token: str
    new_password: str

@app.post("/auth/forgot-password")
async def forgot_password(data: ForgotPasswordRequest, background_tasks: BackgroundTasks):
    if data.email not in users_db:
        return {"message": "If the email exists, a reset link has been sent"}
    token = secrets.token_urlsafe(32)
    token_hash = hashlib.sha256(token.encode()).hexdigest()
    password_reset_tokens[token_hash] = {"email": data.email, "expires_at": time.time() + 3600, "used": False}
    async def send_reset_email(email: str, token: str):
        print(f"Password reset for {email}: https://example.com/reset-password?token={token}")
    background_tasks.add_task(send_reset_email, data.email, token)
    return {"message": "If the email exists, a reset link has been sent"}

@app.post("/auth/reset-password")
async def reset_password(data: ResetPasswordRequest):
    token_hash = hashlib.sha256(data.token.encode()).hexdigest()
    stored = password_reset_tokens.get(token_hash)
    if not stored:
        raise HTTPException(status_code=400, detail="Invalid token")
    if stored["used"]:
        raise HTTPException(status_code=400, detail="Token already used")
    if time.time() > stored["expires_at"]:
        raise HTTPException(status_code=400, detail="Token expired")
    stored["used"] = True
    from passlib.context import CryptContext
    pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
    users_db[stored["email"]]["password_hash"] = pwd_context.hash(data.new_password)
    return {"message": "Password has been reset successfully"}
```

### 5. Full MFA Implementation

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import pyotp
import qrcode
import base64
from io import BytesIO
import secrets
from datetime import datetime, timedelta

app = FastAPI()
users_db = {}
mfa_states = {}

class MFASetupResponse(BaseModel):
    secret: str
    qrcode: str
    backup_codes: list

class MFAVerifyRequest(BaseModel):
    username: str
    token: str

class MFALoginRequest(BaseModel):
    username: str
    password: str
    mfa_token: str

def generate_backup_codes(count=8):
    return [secrets.token_hex(4) for _ in range(count)]

@app.post("/mfa/setup")
async def setup_mfa(username: str):
    secret = pyotp.random_base32()
    uri = pyotp.totp.TOTP(secret).provisioning_uri(name=username, issuer_name="MyApp")
    qr = qrcode.make(uri)
    buffer = BytesIO()
    qr.save(buffer, format="PNG")
    qr_b64 = base64.b64encode(buffer.getvalue()).decode()
    backup_codes = generate_backup_codes()
    mfa_states[username] = {"secret": secret, "backup_codes": {code: False for code in backup_codes}, "verified": False}
    return MFASetupResponse(secret=secret, qrcode=qr_b64, backup_codes=backup_codes)

@app.post("/mfa/verify")
async def verify_mfa(data: MFAVerifyRequest):
    mfa_state = mfa_states.get(data.username)
    if not mfa_state:
        raise HTTPException(status_code=400, detail="MFA not set up")
    totp = pyotp.TOTP(mfa_state["secret"])
    if totp.verify(data.token, valid_window=1):
        mfa_state["verified"] = True
        return {"message": "MFA verified successfully"}
    for code, used in mfa_state["backup_codes"].items():
        if data.token == code and not used:
            mfa_state["backup_codes"][code] = True
            mfa_state["verified"] = True
            return {"message": "Backup code accepted"}
    raise HTTPException(status_code=400, detail="Invalid token")

@app.post("/mfa/login")
async def login_with_mfa(data: MFALoginRequest):
    if data.username not in users_db:
        raise HTTPException(status_code=401, detail="Invalid credentials")
    mfa_state = mfa_states.get(data.username)
    if not mfa_state:
        raise HTTPException(status_code=400, detail="MFA not set up")
    totp = pyotp.TOTP(mfa_state["secret"])
    valid = totp.verify(data.mfa_token, valid_window=1)
    if not valid:
        for code, used in mfa_state["backup_codes"].items():
            if data.mfa_token == code and not used:
                mfa_state["backup_codes"][code] = True
                valid = True
                break
    if not valid:
        raise HTTPException(status_code=401, detail="Invalid MFA token")
    from jose import jwt
    access_token = jwt.encode({"sub": data.username, "exp": datetime.utcnow() + timedelta(hours=1)}, "secret", algorithm="HS256")
    return {"access_token": access_token, "token_type": "bearer"}
```

## Real-World Use Cases

- **Web Application Login**: User authentication with email/password, social login, SSO
- **API Security**: JWT bearer tokens for stateless API authentication
- **Mobile App Auth**: OAuth2 with refresh tokens for persistent mobile sessions
- **Microservices Auth**: Service-to-service authentication with mutual TLS or JWTs
- **Single Sign-On (SSO)**: SAML/OIDC integration for enterprise portals
- **Two-Factor Authentication**: TOTP-based MFA for sensitive operations
- **Passwordless Login**: Magic links or one-time codes for frictionless auth
- **API Key Management**: Developer portals with API key generation and rotation
- **OAuth2 Provider**: Custom OAuth2 server for third-party application access
- **Identity Federation**: Cross-domain authentication with OpenID Connect

## Common Mistakes

- **Storing passwords in plain text** — never hash with MD5/SHA1
- **Not using salt for password hashing** — bcrypt/argon2 include salt automatically
- **Short JWT expiration** — tokens that never expire are a security risk
- **Storing JWT in localStorage** — vulnerable to XSS attacks
- **Not validating JWT signature** — accepting tampered tokens
- **Exposing internal IDs in JWT** — leaking user enumeration information
- **Not using HTTPS** — tokens and passwords sent in plain text
- **Weak secret keys** — using "secret" or easily guessable strings
- **Session fixation** — not regenerating session IDs after login
- **Not implementing rate limiting** — allowing brute force attacks on login

## Best Practices

- Always use strong, adaptive password hashing (argon2 > bcrypt > pbkdf2)
- Use HTTPS for all authentication-related traffic
- Implement short-lived access tokens (15-60 min) with refresh tokens
- Store JWT in HttpOnly cookies for web apps, not localStorage
- Always validate JWT signature, expiration, and issuer
- Use separate secrets for access and refresh tokens
- Implement account lockout after failed login attempts
- Use CSRF tokens for session-based authentication
- Log all authentication events for audit trails
- Implement rate limiting on login, registration, and password reset endpoints
- Use MFA for sensitive operations and admin accounts
- Rotate secrets and keys regularly

## Interview Questions

**Q1: Difference between authentication and authorization?**
A: Authentication verifies identity ("who you are"), authorization determines access ("what you can do").

**Q2: How does JWT work?**
A: JWT has three parts: header, payload, signature. The server signs with a secret. Clients present the token, servers verify the signature.

**Q3: What is the OAuth2 authorization code flow?**
A: Client redirects user to auth server, user authenticates, returns code. Client exchanges code for tokens using client secret.

**Q4: Why bcrypt over SHA-256 for passwords?**
A: bcrypt is adaptive, includes salt, and is intentionally slow. SHA-256 is fast, making brute-force practical.

**Q5: Session vs token authentication?**
A: Session stores state server-side, requires lookup. Token (JWT) is stateless — user info is embedded. Token scales better horizontally.

## Coding Challenges

**Challenge 1: JWT Auth System**
Build a complete auth system with registration, login, token refresh, and protected routes using FastAPI and PyJWT.

**Challenge 2: OAuth2 Client**
Create an OAuth2 client that authenticates via GitHub, stores the access token, and fetches user repositories.

**Challenge 3: Password Manager**
Implement secure password storage with bcrypt hashing, strength validation, and forgot password flow.

**Challenge 4: MFA-Protected API**
Add TOTP-based MFA to an existing API with setup, verification, backup codes.

**Challenge 5: Role-Based Access Control**
Design an RBAC system with users, roles, and permissions. Implement middleware that checks permissions.

## Summary

Authentication is critical for any application handling user data. Python provides robust libraries — PyJWT for JWTs, bcrypt/argon2 for password hashing, Authlib for OAuth2, pyotp for MFA. Understanding session vs token auth, JWT handling, secure password storage, OAuth2 flows, and RBAC enables building secure and scalable systems. Modern authentication balances security with user experience through refresh tokens, social login, and MFA.

## Related Topics

- [71. APIs](./71_apis.md)
- [75. REST API Design](./75_rest_api_design.md)
- [73. FastAPI](./73_fastapi.md)
- [72. Flask](./72_flask.md)
- [70. HTTP Requests](./70_http_requests.md)
- [74. Web Scraping](./74_web_scraping.md)
