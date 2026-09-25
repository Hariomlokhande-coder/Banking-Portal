# Banking Portal API

A comprehensive banking portal API built with Spring Boot that provides secure user authentication, account management, and transaction processing capabilities.

## 📋 Table of Contents

- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Prerequisites](#-prerequisites)
- [Installation](#-installation)
- [Configuration](#-configuration)
- [Running the Application](#-running-the-application)
- [Docker Setup](#-docker-setup)
- [API Endpoints](#-api-endpoints)
- [Project Structure](#-project-structure)
- [Security Features](#-security-features)
- [Testing](#-testing)
- [Documentation](#-documentation)
- [Contributing](#-contributing)

## ✨ Features

- **User Management**
  - User registration with account creation
  - User profile updates
  - Secure password reset with OTP verification

- **Authentication & Authorization**
  - Password-based login
  - OTP-based login
  - JWT token-based authentication
  - Login notifications with geolocation tracking

- **Account Operations**
  - PIN creation and management
  - Cash deposit and withdrawal
  - Fund transfer between accounts
  - Account balance inquiry

- **Transaction Management**
  - Transaction history retrieval
  - Bank statement generation and email delivery
  - Transaction tracking with timestamps

- **Security**
  - BCrypt password encryption
  - JWT token management
  - PIN validation
  - CORS configuration
  - Rate limiting and idempotency checks

- **Additional Features**
  - Email notifications
  - Caching with Redis and Caffeine
  - Swagger/OpenAPI documentation
  - Comprehensive error handling

## 🛠 Tech Stack

- **Framework**: Spring Boot 3.3.1
- **Language**: Java 17
- **Database**: MySQL 8.0
- **Cache**: Redis 7, Caffeine
- **Security**: Spring Security, JWT (jjwt 0.11.5)
- **Build Tool**: Maven
- **Documentation**: SpringDoc OpenAPI (Swagger)
- **Email**: Spring Mail (SMTP)
- **Other Libraries**:
  - Lombok
  - MapStruct
  - JavaFaker
  - libphonenumber
  - Jackson

## 📦 Prerequisites

Before you begin, ensure you have the following installed:

- **Java Development Kit (JDK)**: Version 17 or higher
- **Maven**: Version 3.6+ (or use the included `mvnw` wrapper)
- **MySQL**: Version 8.0+
- **Redis**: Version 7+ (optional, for caching)
- **Git**: For cloning the repository

## 🚀 Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd BankingPortal-API
   ```

2. **Create MySQL database**
   ```sql
   CREATE DATABASE bankingapp;
   ```

3. **Configure application properties**
   ```bash
   cp src/main/resources/application.properties.sample src/main/resources/application.properties
   ```

4. **Update configuration** (see [Configuration](#-configuration) section)

## ⚙️ Configuration

Edit `src/main/resources/application.properties` with your settings:

### Database Configuration
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/bankingapp
spring.datasource.username=your_username
spring.datasource.password=your_password
```

### JWT Configuration
```properties
jwt.secret=your-secret-key-change-this-in-production
jwt.expiration=86400000  # 24 hours in milliseconds
jwt.header=Authorization
jwt.prefix=Bearer
```

### Email Configuration
```properties
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=your-email@gmail.com
spring.mail.password=your-app-password
```

### Geolocation API
```properties
geo.api.url=https://api.findip.net/
geo.api.key=your-api-key
```

### Server Port
```properties
server.port=8180
```

## 🏃 Running the Application

### Using Maven Wrapper (Recommended)
```bash
# Build the project
./mvnw clean install

# Run the application
./mvnw spring-boot:run
```

### Using Maven (if installed)
```bash
mvn clean install
mvn spring-boot:run
```

The application will start on `http://localhost:8180`

## 📡 API Endpoints

### User Endpoints
- `POST /api/users/register` - Register a new user
- `POST /api/users/login` - Login with password
- `POST /api/users/generate-otp` - Generate OTP for login
- `POST /api/users/verify-otp` - Verify OTP and login
- `POST /api/users/update` - Update user profile
- `GET /api/users/logout` - Logout (invalidate token)

### Authentication Endpoints
- `POST /api/auth/password-reset/send-otp` - Request OTP for password reset
- `POST /api/auth/password-reset/verify-otp` - Verify OTP and get reset token
- `POST /api/auth/password-reset` - Reset password with token

### Account Endpoints (Protected - Requires JWT)
- `GET /api/account/pin/check` - Check if PIN is created
- `POST /api/account/pin/create` - Create account PIN
- `POST /api/account/pin/update` - Update account PIN
- `POST /api/account/deposit` - Deposit cash
- `POST /api/account/withdraw` - Withdraw cash
- `POST /api/account/fund-transfer` - Transfer funds to another account
- `GET /api/account/transactions` - Get transaction history
- `GET /api/account/send-statement` - Send bank statement via email

### Dashboard Endpoints (Protected - Requires JWT)
- `GET /api/dashboard/user` - Get user details
- `GET /api/dashboard/account` - Get account details

### API Documentation
- `GET /swagger-ui.html` - Swagger UI interface
- `GET /v3/api-docs` - OpenAPI JSON specification

## 📁 Project Structure

```
BankingPortal-API/
├── src/
│   ├── main/
│   │   ├── java/com/webapp/bankingportal/
│   │   │   ├── BankingportalApplication.java    # Main application class
│   │   │   ├── config/                          # Configuration classes
│   │   │   │   ├── CacheConfig.java
│   │   │   │   ├── CorsConfig.java
│   │   │   │   ├── RedisConfig.java
│   │   │   │   ├── SwaggerConfig.java
│   │   │   │   └── WebSecurityConfig.java
│   │   │   ├── controller/                      # REST controllers
│   │   │   │   ├── AccountController.java
│   │   │   │   ├── AuthController.java
│   │   │   │   ├── DashboardController.java
│   │   │   │   ├── GlobalExceptionHandler.java
│   │   │   │   └── UserController.java
│   │   │   ├── dto/                             # Data Transfer Objects
│   │   │   ├── entity/                          # JPA entities
│   │   │   ├── exception/                       # Custom exceptions
│   │   │   ├── mapper/                          # MapStruct mappers
│   │   │   ├── repository/                      # JPA repositories
│   │   │   ├── security/                        # Security components
│   │   │   ├── service/                         # Business logic
│   │   │   ├── type/                            # Enums
│   │   │   └── util/                            # Utility classes
│   │   └── resources/
│   │       ├── application.properties.sample    # Configuration template
│   │       └── logback-spring.xml               # Logging configuration
│   └── test/                                     # Test files
├── docker/
│   └── docker-compose.yml                       # Docker services
├── Dockerfile                                   # Docker image configuration
├── pom.xml                                      # Maven dependencies
├── mvnw                                         # Maven wrapper (Unix)
├── mvnw.cmd                                     # Maven wrapper (Windows)
├── SECURITY.md                                  # Security policy
└── WORKFLOW_DOCUMENTATION.md                    # Detailed workflow docs
```

## 🔒 Security Features

- **JWT Authentication**: Secure token-based authentication
- **Password Encryption**: BCrypt hashing for passwords and PINs
- **OTP Verification**: Time-limited OTP for sensitive operations
- **Token Management**: Database-stored tokens with expiration
- **CORS Configuration**: Configured for secure cross-origin requests
- **Input Validation**: Comprehensive validation using Bean Validation
- **Idempotency**: Prevents duplicate transactions
- **Login Alerts**: Email notifications with geolocation

For security vulnerability reporting, see [SECURITY.md](SECURITY.md)

## 🧪 Testing

Run tests using Maven:
```bash
./mvnw test
```

The project includes unit tests and integration tests for:
- Account operations
- Authentication flows
- Token management
- Cache functionality
- Dashboard services

## 📚 Documentation

- **API Documentation**: Access Swagger UI at `http://localhost:8180/swagger-ui.html`
- **Workflow Documentation**: See [WORKFLOW_DOCUMENTATION.md](WORKFLOW_DOCUMENTATION.md) for detailed workflow explanations
- **Postman Collection**: Import `Banking Portal.postman_collection.json` into Postman for API testing

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License (if applicable) - see the LICENSE file for details.

## 👤 Author

**Hariom Lokhande**

For questions or support, please contact: hariomlokhande3456@gmail.com or send Pull request and any suggestion directly here.

## 🎯 Key Features in Detail

### Transaction Types
- Cash Deposit
- Cash Withdrawal
- Fund Transfer

### Email Notifications
- Welcome email on registration
- Login alerts with geolocation
- OTP delivery
- Bank statement emails

### Caching Strategy
- **Redis**: OTP storage, idempotency keys
- **Caffeine**: Application-level caching

---

**Note**: Make sure to update all sensitive configuration values (database passwords, JWT secrets, email credentials) before deploying to production.
