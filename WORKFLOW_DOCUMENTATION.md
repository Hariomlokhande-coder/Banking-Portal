# Banking Portal API - Complete Workflow Documentation

## 🚀 Application Startup Flow

### 1. **Application Initialization**
```
BankingportalApplication.java (Main Entry Point)
    ↓
@SpringBootApplication triggers:
    - Component scanning
    - Auto-configuration
    - Bean creation
    ↓
Configuration Classes Load:
    - WebSecurityConfig.java → Security setup
    - RedisConfig.java → Cache configuration
    - CacheConfig.java → Caffeine cache
    - CorsConfig.java → CORS settings
    - SwaggerConfig.java → API documentation
    ↓
Application starts on port 8180 (configurable)
```

### 2. **Configuration Files**
- **application.properties** (or .sample): Database, JWT, Email, Redis settings
- **pom.xml**: Maven dependencies and build configuration

---

## 📋 Key Workflows

### **WORKFLOW 1: User Registration**

```
POST /api/users/register
    ↓
UserController.registerUser()
    ↓
UserServiceImpl.registerUser()
    ├─→ ValidationUtil.validateNewUser() [Validates email, phone, etc.]
    ├─→ encodePassword() [BCrypt encryption]
    ├─→ saveUserWithAccount()
    │   ├─→ UserRepository.save() [Save User entity]
    │   └─→ AccountService.createAccount()
    │       ├─→ Generate unique 6-digit account number (UUID-based)
    │       └─→ AccountRepository.save() [Save Account entity]
    └─→ EmailService.sendEmail() [Welcome email]
    ↓
Returns: UserResponse (JSON with user details + account number)
```

**Files Involved:**
- `UserController.java` (line 35-46)
- `UserServiceImpl.java` (line 55-60)
- `ValidationUtil.java`
- `AccountServiceImpl.java` (line 41-47)
- `EmailServiceImpl.java`

---

### **WORKFLOW 2: User Login (Password-based)**

```
POST /api/users/login
    ↓
UserController.login()
    ↓
UserServiceImpl.login()
    ├─→ authenticateUser()
    │   ├─→ getUserByIdentifier() [Find user by email/account]
    │   └─→ AuthenticationManager.authenticate() [Spring Security]
    ├─→ sendLoginNotification() [Async email with geolocation]
    │   ├─→ GeolocationService.getGeolocation(IP)
    │   └─→ EmailService.sendEmail() [Login alert]
    ├─→ generateAndSaveToken()
    │   ├─→ TokenService.generateToken() [JWT creation]
    │   └─→ TokenService.saveToken() [Store in DB]
    └─→ Returns JWT token
    ↓
Response: "Token issued successfully: <JWT_TOKEN>"
```

**Files Involved:**
- `UserController.java` (line 48-52)
- `UserServiceImpl.java` (line 63-69)
- `TokenServiceImpl.java` (line 51-54, 129-145)
- `JwtAuthenticationFilter.java` (for subsequent requests)

---

### **WORKFLOW 3: User Login (OTP-based)**

```
Step 1: Generate OTP
POST /api/users/generate-otp
    ↓
UserController.generateOtp()
    ↓
UserServiceImpl.generateOtp()
    ├─→ getUserByIdentifier()
    ├─→ OtpService.generateOTP() [6-digit OTP, stored in Redis]
    └─→ sendOtpEmail() [Async email sending]
    ↓
Response: "OTP sent successfully to <email>"

Step 2: Verify OTP & Login
POST /api/users/verify-otp
    ↓
UserController.verifyOtpAndLogin()
    ↓
UserServiceImpl.verifyOtpAndLogin()
    ├─→ validateOtpRequest()
    ├─→ getUserByIdentifier()
    ├─→ validateOtp() [Check OTP from Redis]
    ├─→ generateAndSaveToken() [JWT creation]
    └─→ Returns JWT token
    ↓
Response: "Token issued successfully: <JWT_TOKEN>"
```

**Files Involved:**
- `UserController.java` (line 54-64)
- `UserServiceImpl.java` (line 72-86)
- `OtpServiceImpl.java`
- `TokenServiceImpl.java`

---

### **WORKFLOW 4: Protected API Request (With JWT)**

```
Any Protected Endpoint (e.g., /api/account/deposit)
    ↓
JwtAuthenticationFilter.doFilterInternal() [Intercepts request]
    ├─→ Extract "Authorization: Bearer <token>" header
    ├─→ TokenService.validateToken() [Check if token exists in DB]
    ├─→ TokenService.getUsernameFromToken() [Extract account number]
    ├─→ UserDetailsService.loadUserByUsername() [Load user details]
    └─→ Set Authentication in SecurityContext
    ↓
Controller Method Executes
    ├─→ LoggedinUser.getAccountNumber() [Get current user from SecurityContext]
    └─→ Business logic execution
    ↓
Response returned
```

**Files Involved:**
- `JwtAuthenticationFilter.java` (line 49-97)
- `TokenServiceImpl.java` (line 148-152)
- `LoggedinUser.java` (line 19-26)
- `WebSecurityConfig.java` (line 69-91)

---

### **WORKFLOW 5: Cash Deposit**

```
POST /api/account/deposit
Headers: Authorization: Bearer <JWT_TOKEN>
Body: { "pin": "1234", "amount": 1000 }
    ↓
JwtAuthenticationFilter validates token
    ↓
AccountController.cashDeposit()
    ↓
AccountServiceImpl.cashDeposit() [@Transactional]
    ├─→ validatePin() [BCrypt match with stored PIN]
    ├─→ validateAmount() [Check: >0, multiple of 100, ≤100000]
    ├─→ Update Account.balance
    ├─→ Create Transaction record
    │   ├─→ TransactionType.CASH_DEPOSIT
    │   ├─→ Set sourceAccount
    │   └─→ TransactionRepository.save()
    └─→ AccountRepository.save()
    ↓
Response: "Cash deposit successful"
```

**Files Involved:**
- `AccountController.java` (line 60-69)
- `AccountServiceImpl.java` (line 161-177)
- `TransactionRepository.java`
- `AccountRepository.java`

---

### **WORKFLOW 6: Fund Transfer**

```
POST /api/account/fund-transfer
Headers: Authorization: Bearer <JWT_TOKEN>
Body: { "targetAccountNumber": "ABC123", "pin": "1234", "amount": 500 }
    ↓
JwtAuthenticationFilter validates token
    ↓
AccountController.fundTransfer()
    ↓
AccountServiceImpl.fundTransfer() [@Transactional]
    ├─→ validatePin(sourceAccount)
    ├─→ validateAmount()
    ├─→ Check: sourceAccount ≠ targetAccount
    ├─→ Validate targetAccount exists
    ├─→ Check sufficient balance
    ├─→ Update sourceAccount.balance (decrease)
    ├─→ Update targetAccount.balance (increase)
    └─→ Create Transaction record
        ├─→ TransactionType.CASH_TRANSFER
        ├─→ Set sourceAccount & targetAccount
        └─→ TransactionRepository.save()
    ↓
Response: "Cash transfer successful"
```

**Files Involved:**
- `AccountController.java` (line 82-92)
- `AccountServiceImpl.java` (line 205-240)
- `TransactionRepository.java`

---

### **WORKFLOW 7: Password Reset**

```
Step 1: Request OTP
POST /api/auth/password-reset/send-otp
Body: { "identifier": "user@email.com" }
    ↓
AuthController.sendOtpForPasswordReset()
    ↓
AuthServiceImpl.sendOtpForPasswordReset()
    ├─→ getUserByIdentifier()
    ├─→ OtpService.generateOTP()
    └─→ sendOtpEmail()
    ↓
Response: "OTP sent successfully"

Step 2: Verify OTP & Get Reset Token
POST /api/auth/password-reset/verify-otp
Body: { "identifier": "user@email.com", "otp": "123456" }
    ↓
AuthController.verifyOtpAndIssueResetToken()
    ↓
AuthServiceImpl.verifyOtpAndIssueResetToken()
    ├─→ validateOtp() [Check OTP from Redis]
    ├─→ generatePasswordResetToken()
    │   ├─→ Generate UUID token
    │   ├─→ Set expiry (24 hours)
    │   └─→ PasswordResetTokenRepository.save()
    └─→ Returns reset token
    ↓
Response: "Password reset token issued: <UUID_TOKEN>"

Step 3: Reset Password
POST /api/auth/password-reset
Body: { "identifier": "user@email.com", "resetToken": "<UUID>", "newPassword": "newpass" }
    ↓
AuthController.resetPassword()
    ↓
AuthServiceImpl.resetPassword() [@Transactional]
    ├─→ getUserByIdentifier()
    ├─→ verifyPasswordResetToken() [Check token validity]
    ├─→ UserService.resetPassword()
    │   ├─→ BCrypt encode new password
    │   └─→ UserRepository.save()
    └─→ deletePasswordResetToken()
    ↓
Response: "Password reset successful"
```

**Files Involved:**
- `AuthController.java` (line 20-34)
- `AuthServiceImpl.java` (line 65-108)
- `PasswordResetTokenRepository.java`
- `UserServiceImpl.java` (line 100-108)

---

### **WORKFLOW 8: PIN Management**

```
Create PIN:
POST /api/account/pin/create
Headers: Authorization: Bearer <JWT_TOKEN>
Body: { "password": "userpass", "pin": "1234" }
    ↓
AccountController.createPIN()
    ↓
AccountServiceImpl.createPin()
    ├─→ validatePassword() [BCrypt match]
    ├─→ Check PIN format (4 digits)
    ├─→ Check PIN doesn't exist
    ├─→ BCrypt encode PIN
    └─→ AccountRepository.save()
    ↓
Response: "PIN creation successful"

Update PIN:
POST /api/account/pin/update
Headers: Authorization: Bearer <JWT_TOKEN>
Body: { "oldPin": "1234", "password": "userpass", "newPin": "5678" }
    ↓
AccountController.updatePIN()
    ↓
AccountServiceImpl.updatePin()
    ├─→ validatePassword()
    ├─→ validatePin(oldPin)
    ├─→ Check new PIN format
    ├─→ BCrypt encode new PIN
    └─→ AccountRepository.save()
    ↓
Response: "PIN update successful"
```

**Files Involved:**
- `AccountController.java` (line 37-58)
- `AccountServiceImpl.java` (line 104-143)

---

## 🗄️ Database Schema Flow

```
User Entity
    ├─→ OneToOne → Account Entity
    │   ├─→ OneToMany → Transaction Entity (as sourceAccount)
    │   ├─→ OneToMany → Transaction Entity (as targetAccount)
    │   └─→ OneToMany → Token Entity (JWT tokens)
    └─→ OneToMany → PasswordResetToken Entity
```

**Entity Files:**
- `User.java` - User information
- `Account.java` - Account details, balance, PIN
- `Transaction.java` - Transaction history
- `Token.java` - Active JWT tokens
- `PasswordResetToken.java` - Password reset tokens
- `OtpInfo.java` - OTP information (stored in Redis)

---

## 🔐 Security Flow

```
Request → JwtAuthenticationFilter
    ├─→ Extract JWT from Authorization header
    ├─→ Validate token exists in database
    ├─→ Extract account number from token
    ├─→ Load UserDetails
    └─→ Set Authentication in SecurityContext
        ↓
Controller Method
    ├─→ LoggedinUser.getAccountNumber() [From SecurityContext]
    └─→ Execute business logic
```

**Security Files:**
- `WebSecurityConfig.java` - Security configuration
- `JwtAuthenticationFilter.java` - JWT validation filter
- `JwtAuthenticationEntryPoint.java` - Unauthorized handler
- `TokenServiceImpl.java` - JWT operations

---

## 📧 Email Service Flow

```
EmailService.sendEmail()
    ├─→ Configure SMTP (from application.properties)
    ├─→ Create MimeMessage
    ├─→ Set recipient, subject, body
    └─→ JavaMailSender.send() [Async]
```

**Email Templates:**
- Welcome email (registration)
- Login notification (with geolocation)
- OTP email
- Bank statement email

---

## 💾 Caching Strategy

```
Redis Cache:
    - OTP storage (with expiration)
    - Idempotency keys for transactions

Caffeine Cache:
    - Application-level caching
    - Configured in CacheConfig.java
```

**Cache Files:**
- `RedisConfig.java`
- `CacheConfig.java`
- `CacheServiceImpl.java`
- `OtpServiceImpl.java` (uses Redis)

---

## 🚦 Request Flow Summary

```
1. Client Request
    ↓
2. JwtAuthenticationFilter (if protected endpoint)
    ├─→ Validate JWT token
    └─→ Set SecurityContext
    ↓
3. Controller Layer
    ├─→ Extract request data
    ├─→ Call Service layer
    └─→ Return response
    ↓
4. Service Layer
    ├─→ Business logic
    ├─→ Validation
    ├─→ Repository calls
    └─→ External service calls (Email, Geolocation)
    ↓
5. Repository Layer
    └─→ Database operations (JPA/Hibernate)
    ↓
6. Response to Client
```

---

## 📁 Key File Locations

### Controllers
- `UserController.java` - User operations
- `AccountController.java` - Account operations
- `AuthController.java` - Authentication
- `DashboardController.java` - Dashboard data

### Services
- `UserServiceImpl.java` - User business logic
- `AccountServiceImpl.java` - Account operations
- `AuthServiceImpl.java` - Password reset
- `TokenServiceImpl.java` - JWT management
- `OtpServiceImpl.java` - OTP generation/validation
- `EmailServiceImpl.java` - Email sending
- `TransactionServiceImpl.java` - Transaction history

### Security
- `JwtAuthenticationFilter.java` - JWT filter
- `WebSecurityConfig.java` - Security configuration

### Entities
- `User.java`, `Account.java`, `Transaction.java`
- `Token.java`, `PasswordResetToken.java`, `OtpInfo.java`

### Configuration
- `application.properties` - Application settings
- `pom.xml` - Maven dependencies

---

## 🏃 How to Run

1. **Setup Database:**
   - Create MySQL database: `bankingapp`
   - Copy `application.properties.sample` to `application.properties`
   - Update database credentials

2. **Configure Services:**
   - Update JWT secret key
   - Configure email SMTP settings
   - Set Redis connection (if using)

3. **Build & Run:**
   ```bash
   mvn clean install
   mvn spring-boot:run
   ```

4. **Access:**
   - API: `http://localhost:8180`
   - Swagger UI: `http://localhost:8180/swagger-ui.html`

---

## 🔄 Transaction Flow Example

```
User Registration → Account Creation → Login → PIN Creation → Deposit → Transfer → View Transactions
```

Each step follows the workflows above, with proper authentication, validation, and database persistence.

