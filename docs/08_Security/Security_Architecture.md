# Security Architecture

## Executive Summary

OpenDataAnalytics.AI implements defense-in-depth security architecture with multiple layers of protection including authentication, authorization, encryption, audit logging, and continuous monitoring.

## Security Principles

1. **Zero Trust**: Verify every request
2. **Defense in Depth**: Multiple protection layers
3. **Least Privilege**: Minimal necessary access
4. **Encryption by Default**: Protect data in transit and at rest
5. **Audit Everything**: Complete visibility
6. **Secure by Default**: Secure out-of-the-box
7. **Fail Secure**: Error on side of security

## Security Layers

```
┌─────────────────────────────────────────────────────┐
│              User/Network Layer                      │
│        (TLS, DDoS Protection, WAF)                   │
└────────────────────┬────────────────────────────────┘
┌────────────────────▼────────────────────────────────┐
│           Authentication Layer                      │
│   (OAuth2, JWT, MFA, Session Management)            │
└────────────────────┬────────────────────────────────┘
┌────────────────────▼────────────────────────────────┐
│           Authorization Layer                       │
│        (RBAC, ABAC, Resource Permissions)           │
└────────────────────┬────────────────────────────────┘
┌────────────────────▼────────────────────────────────┐
│            Application Layer                        │
│   (Input Validation, SQL Injection Prevention)      │
└────────────────────┬────────────────────────────────┘
┌────────────────────▼────────────────────────────────┐
│             Data Layer                              │
│      (Encryption, Access Control, Audit)            │
└─────────────────────────────────────────────────────┘
```

## Authentication Architecture

### Authentication Methods

#### 1. Email/Password
```
User → Credentials → Hash & Compare → JWT Token
                         ↓
                    Password Reset Email
```

**Requirements**:
- Minimum 12 characters
- Mix of uppercase, lowercase, numbers, symbols
- No common passwords
- Salted hash (bcrypt)

#### 2. OAuth2 / OIDC
```
User → Redirect to OAuth Provider → Provider Authenticates
    ↓
User → Returns to OpenDataAnalytics.AI with Auth Code
    ↓
Backend → Exchange Code for Token
    ↓
Create User if New → Issue JWT
```

**Supported Providers**:
- Google
- GitHub
- Microsoft
- Custom OIDC

#### 3. Multi-Factor Authentication (MFA)

```
Password ✓
    ↓
Prompt for MFA → Time-based OTP (TOTP) / SMS / Email
    ↓
Verify Code → Issue JWT with MFA Flag
```

**Methods**:
- TOTP (Google Authenticator)
- SMS (optional)
- Email backup codes

### Token Management

#### JWT Token Structure

```json
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "user_123",
    "email": "user@example.com",
    "roles": ["analyst", "viewer"],
    "mfa_verified": true,
    "iat": 1640995200,
    "exp": 1641081600
  },
  "signature": "HMACSHA256(encoded_header.encoded_payload, secret)"
}
```

#### Token Lifecycle

```
Issue Token (exp: 1 hour)
    ↓
Use in Request Authorization Header
    ↓
If Expired → Use Refresh Token (exp: 30 days)
    ↓
Get New Access Token (exp: 1 hour)
    ↓
If Refresh Expired → Re-authenticate
```

#### Token Security

```python
# ✅ Secure token handling
class TokenManager:
    """Manage JWT tokens securely."""
    
    @staticmethod
    def create_token(user_id: str) -> str:
        """Create JWT token."""
        payload = {
            "sub": user_id,
            "iat": datetime.utcnow(),
            "exp": datetime.utcnow() + timedelta(hours=1),
            "jti": secrets.token_urlsafe()  # Unique token ID
        }
        return jwt.encode(payload, SECRET_KEY, algorithm="HS256")
    
    @staticmethod
    def verify_token(token: str) -> dict:
        """Verify JWT token."""
        try:
            # Check if token is revoked
            if is_token_revoked(token):
                raise InvalidTokenError("Token revoked")
            
            payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
            return payload
        except jwt.ExpiredSignatureError:
            raise InvalidTokenError("Token expired")
        except jwt.InvalidTokenError:
            raise InvalidTokenError("Invalid token")
    
    @staticmethod
    def revoke_token(token: str):
        """Revoke token (add to blacklist)."""
        add_to_revocation_list(token)
```

## Authorization Architecture

### Role-Based Access Control (RBAC)

#### Roles

| Role | Permissions |
|------|-------------|
| **Owner** | Full control, delete, transfer |
| **Editor** | Create, edit, delete own items |
| **Viewer** | Read-only access |
| **Commenter** | View, comment, no modifications |
| **Analyst** | Create analyses, dashboards |
| **Admin** | System administration |

#### Role Hierarchy

```
Admin
 ├─ Owner
 │  ├─ Editor
 │  │  ├─ Viewer
 │  │  └─ Commenter
 │  └─ Analyst
```

#### Permission Matrix

| Action | Owner | Editor | Analyst | Viewer | Commenter |
|--------|-------|--------|---------|--------|-----------|
| Create | ✅ | ✅ | ✅ | ❌ | ❌ |
| Read | ✅ | ✅ | ✅ | ✅ | ✅ |
| Update | ✅ | ✅ | ✅ | ❌ | ❌ |
| Delete | ✅ | ✅ | ❌ | ❌ | ❌ |
| Share | ✅ | ⚠️ | ❌ | ❌ | ❌ |
| Comment | ✅ | ✅ | ✅ | ✅ | ✅ |

### Attribute-Based Access Control (ABAC)

```python
# ABAC example
class AccessPolicy:
    """Evaluate access based on attributes."""
    
    @staticmethod
    def can_access(user: User, resource: Resource) -> bool:
        """
        Determine access based on attributes:
        - User role
        - Resource sensitivity
        - Time of day
        - Location
        - Device type
        """
        checks = [
            user.role in resource.allowed_roles,
            resource.sensitivity <= user.clearance,
            is_within_access_hours(user),
            is_trusted_location(user),
            is_compliant_device(user.device)
        ]
        return all(checks)
```

### Row-Level Security

```python
# Row-level security example
def get_projects_for_user(user_id: str) -> List[Project]:
    """Get only projects user has access to."""
    query = (
        Project.query
        .join(Collaborator, Project.id == Collaborator.project_id)
        .filter(Collaborator.user_id == user_id)
        .all()
    )
    return query

# Column-level masking
@dataclass
class DataColumn:
    name: str
    data_type: str
    sensitive: bool = False
    
    def get_masked_values(self, user: User) -> List:
        """Return masked values if user can't see column."""
        if not user.has_permission(f"view:{self.name}"):
            return ["***" for _ in self.data]
        return self.data
```

## Data Protection

### Encryption in Transit

```python
# TLS 1.2+ enforcement
# All HTTP → HTTPS redirect
# HSTS headers enabled

security_headers = {
    "Strict-Transport-Security": "max-age=31536000; includeSubDomains",
    "X-Content-Type-Options": "nosniff",
    "X-Frame-Options": "DENY",
    "X-XSS-Protection": "1; mode=block",
    "Content-Security-Policy": "default-src 'self'"
}
```

### Encryption at Rest

```python
# AES-256 encryption for sensitive data
from cryptography.fernet import Fernet

class DataEncryption:
    """Encrypt/decrypt sensitive data."""
    
    def __init__(self, key: str):
        self.cipher = Fernet(key)
    
    def encrypt(self, plaintext: str) -> str:
        """Encrypt data."""
        return self.cipher.encrypt(plaintext.encode()).decode()
    
    def decrypt(self, ciphertext: str) -> str:
        """Decrypt data."""
        return self.cipher.decrypt(ciphertext.encode()).decode()

# Usage
encryption = DataEncryption(ENCRYPTION_KEY)
encrypted_api_key = encryption.encrypt(user_api_key)
```

### Key Management

```python
# Secrets management
class SecretManager:
    """Manage encryption keys and secrets."""
    
    @staticmethod
    def get_secret(secret_name: str) -> str:
        """Get secret from secure storage."""
        # Use AWS Secrets Manager, HashiCorp Vault, or similar
        return load_from_vault(secret_name)
    
    @staticmethod
    def rotate_keys():
        """Rotate encryption keys regularly."""
        # Triggered monthly or after personnel changes
        pass
```

## API Security

### Input Validation

```python
# ✅ Validate all inputs
from pydantic import BaseModel, validator

class ProjectCreate(BaseModel):
    """Validate project creation."""
    name: str
    description: str
    
    @validator('name')
    def validate_name(cls, v):
        if len(v) < 3 or len(v) > 255:
            raise ValueError('Name must be 3-255 characters')
        return v
    
    @validator('description')
    def validate_description(cls, v):
        if len(v) > 1000:
            raise ValueError('Description too long')
        return v

# ✅ Sanitize outputs
def safe_json(data):
    """Return safe JSON without sensitive data."""
    return {
        "id": data["id"],
        "name": data["name"],
        # Exclude: password, api_key, secret
    }
```

### SQL Injection Prevention

```python
# ✅ Use parameterized queries
from sqlalchemy import text

# Good: Parameterized
query = text("SELECT * FROM users WHERE email = :email")
result = db.execute(query, {"email": email})

# ❌ Bad: String concatenation
query = f"SELECT * FROM users WHERE email = '{email}'"  # VULNERABLE
result = db.execute(query)

# ✅ Use ORM
user = User.query.filter_by(email=email).first()
```

### CSRF Protection

```python
# ✅ CSRF token validation
@app.post("/api/v1/projects")
def create_project(request: Request):
    """Create project with CSRF token."""
    token = request.headers.get("X-CSRF-Token")
    
    if not validate_csrf_token(token):
        raise HTTPException(status_code=403, detail="Invalid CSRF token")
    
    # Process request
```

### Rate Limiting

```python
# ✅ Rate limiting
from slowapi import Limiter

limiter = Limiter(key_func=get_remote_address)

@app.post("/api/v1/auth/token")
@limiter.limit("5/minute")  # 5 attempts per minute
def login(credentials: LoginRequest):
    """Login with rate limiting."""
    # Process login
```

## Audit and Logging

### Audit Trail

```python
# ✅ Log all actions
class AuditLog(Base):
    """Track all user actions."""
    id: str
    user_id: str
    action: str  # "create", "update", "delete"
    entity_type: str  # "project", "dataset"
    entity_id: str
    old_value: dict
    new_value: dict
    ip_address: str
    user_agent: str
    status: str  # "success", "failed"
    timestamp: datetime

# Usage
log_action(
    user_id="user_123",
    action="update",
    entity_type="dataset",
    entity_id="ds_456",
    old_value={"name": "old"},
    new_value={"name": "new"}
)
```

### Security Logging

```python
# ✅ Log security events
def log_security_event(event_type: str, details: dict):
    """Log security-relevant events."""
    events = {
        "authentication_failure": True,
        "authorization_failure": True,
        "suspicious_activity": True,
        "security_vulnerability": True,
        "compliance_violation": True,
        "data_access": True
    }
    
    if events.get(event_type):
        security_logger.warning(
            f"SECURITY: {event_type}",
            extra=details
        )
```

## Vulnerability Management

### Security Testing

1. **Static Analysis**: Scan code for vulnerabilities
2. **Dynamic Testing**: Test running application
3. **Penetration Testing**: Manual security testing
4. **Dependency Scanning**: Check third-party packages

### Incident Response

```python
class IncidentResponse:
    """Handle security incidents."""
    
    @staticmethod
    def on_potential_breach():
        """Respond to potential breach."""
        # 1. Isolate affected systems
        isolate_systems()
        
        # 2. Preserve evidence
        backup_logs()
        
        # 3. Investigate
        investigation = investigate_breach()
        
        # 4. Notify affected users
        notify_users(investigation.affected_users)
        
        # 5. Remediate
        remediate()
        
        # 6. Post-mortem
        conduct_postmortem()
```

## Compliance

### GDPR Compliance

```python
# ✅ GDPR features
class GDPRCompliance:
    @staticmethod
    def export_user_data(user_id: str) -> dict:
        """Export all user data (Right to Data Portability)."""
        return {
            "profile": get_user_profile(user_id),
            "projects": get_user_projects(user_id),
            "analyses": get_user_analyses(user_id),
            "activity": get_user_activity(user_id)
        }
    
    @staticmethod
    def delete_user_data(user_id: str):
        """Delete all user data (Right to be Forgotten)."""
        # Delete personal data
        # Anonymize activity logs
        # Revoke all tokens
        # Clear cache
```

### SOC 2 Compliance

- Availability
- Security
- Processing Integrity
- Confidentiality
- Privacy

---

## Related Documents
- [Encryption](Encryption.md)
- [RBAC](RBAC.md)
- [Data Privacy](Data_Privacy.md)
- [Threat Model](Threat_Model.md)
- [Compliance](Compliance.md)

**Last Updated**: December 2024
