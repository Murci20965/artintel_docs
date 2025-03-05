# ARTINTEL LM Platform: Database and API Documentation

This document provides a comprehensive overview of the database structure, API endpoints, and data flow within the ARTINTEL LM authentication service. It is designed to be both technically precise and accessible to non-technical stakeholders.

## Table of Contents

1. [Database Overview](#database-overview)
2. [Database Models](#database-models)
   - [Users Table](#users-table)
   - [User Profiles Table](#user-profiles-table)
3. [API Endpoints and Database Interactions](#api-endpoints-and-database-interactions)
   - [Authentication Endpoints](#authentication-endpoints)
   - [User Management Endpoints](#user-management-endpoints)
4. [Frontend-Backend Communication](#frontend-backend-communication)
   - [Authentication Flow](#authentication-flow)
   - [Data Fetching and Submission](#data-fetching-and-submission)
5. [Recommended Database Enhancements](#recommended-database-enhancements)

## Database Overview

The ARTINTEL LM platform uses a MySQL database to store user data, authentication information, and user preferences. The database is designed with the following principles:

- **Data Integrity**: Relationships between tables are enforced using foreign keys
- **Security**: Passwords are never stored in plain text
- **Scalability**: Tables are structured to accommodate growth
- **UUID Compatibility**: MySQL-compatible UUID handling for primary keys

## Database Models

### Users Table

The `users` table stores core user authentication and authorization information. It is the primary table for user management.

| Field | Data Type | Description | Importance |
|-------|-----------|-------------|------------|
| `id` | VARCHAR(36) | Unique identifier for each user, stored as a UUID string | Primary key, used to identify users across the application |
| `email` | VARCHAR(255) | User's email address | Used for login, verification, and communication; must be unique |
| `hashed_password` | VARCHAR(255) | Bcrypt-hashed password | Securely stores password; never stored or transmitted in plain text |
| `is_active` | BOOLEAN | Whether the user account is active | Controls if user can log in; used for temporary or permanent deactivation |
| `is_verified` | BOOLEAN | Whether the email has been verified | Controls access to features requiring verified status |
| `is_superuser` | BOOLEAN | Whether the user has admin privileges | Controls access to admin-only features and endpoints |
| `user_type` | VARCHAR(50) | Type of user (regular, enterprise) | Determines feature access and billing tier |
| `subscription_tier` | VARCHAR(50) | Subscription level (free, premium, etc.) | Controls feature access and usage limits |
| `full_name` | VARCHAR(255) | User's full name | Used for personalization and identification |
| `organization` | VARCHAR(255) | User's organization name | Used for enterprise accounts and analytics |
| `created_at` | DATETIME | When the user account was created | Used for analytics and account age tracking |
| `updated_at` | DATETIME | When the user account was last updated | Used for tracking changes and triggering events |

#### Technical Implementation Notes:
- The `id` field uses `VARCHAR(36)` instead of MySQL's native UUID type for better compatibility
- The `hashed_password` field stores passwords processed with bcrypt (cost factor 12)
- The `updated_at` field is automatically updated using MySQL's `ON UPDATE CURRENT_TIMESTAMP`

### User Profiles Table

The `user_profiles` table stores additional user information beyond what's needed for authentication. It has a one-to-one relationship with the users table.

| Field | Data Type | Description | Importance |
|-------|-----------|-------------|------------|
| `id` | VARCHAR(36) | Unique identifier for the profile | Primary key, different from user_id for security |
| `user_id` | VARCHAR(36) | Reference to the user's ID | Foreign key linking to users table |
| `display_name` | VARCHAR(100) | User's preferred display name | Used in UI elements instead of full_name |
| `avatar_url` | VARCHAR(255) | URL to user's profile picture | Visual identification in UI |
| `organization_name` | VARCHAR(255) | Detailed organization name | More specific than the users.organization field |
| `organization_domain` | VARCHAR(255) | Domain name of the organization | Used for email verification and SSO |
| `role` | VARCHAR(50) | User's role (developer, analyst, etc.) | Controls feature access and UI customization |
| `bio` | TEXT | User's biography or description | Displayed in profile views |
| `security_questions` | TEXT | JSON-encoded security questions/answers | Used for account recovery as a secondary method |
| `preferences` | TEXT | JSON-encoded user preferences | Stores UI preferences, notification settings, etc. |
| `created_at` | DATETIME | When the profile was created | Used for analytics |
| `updated_at` | DATETIME | When the profile was last updated | Used for tracking changes |

#### Technical Implementation Notes:
- The `preferences` field stores a JSON object with user settings like `{"theme": "dark", "email_notifications": true}`
- The `security_questions` field stores a JSON array of questions and hashed answers
- This table is created automatically when a user registers with profile information

## API Endpoints and Database Interactions

This section documents how each API endpoint interacts with the database, including specific read and write operations.

### Authentication Endpoints

#### `POST /auth/login` - OAuth2 Login

**Database Operations:**
- **Read**: Queries the `users` table to retrieve the user by email
  ```sql
  SELECT * FROM users WHERE email = ? LIMIT 1
  ```
- **Verify**: Compares provided password with the stored hashed_password
- **No Write Operations**: This endpoint does not modify the database

**Data Flow:**
1. Frontend sends email/password in form-urlencoded format
2. Backend queries database to retrieve user
3. If user exists, password is verified using bcrypt
4. If verified, JWT token is generated (not stored in database)
5. Response includes token and user data

#### `POST /auth/login/json` - JSON Login

Identical database operations to OAuth2 login, but accepts JSON input instead of form data.

#### `POST /auth/register` - User Registration

**Database Operations:**
- **Read**: Checks if email already exists
  ```sql
  SELECT id FROM users WHERE email = ? LIMIT 1
  ```
- **Write**: Creates new user record
  ```sql
  INSERT INTO users (id, email, hashed_password, is_active, is_verified, ...)
  VALUES (?, ?, ?, ?, ?, ...)
  ```
- **Optional Write**: May send verification email (not stored in database)

**Data Flow:**
1. Frontend sends registration data in JSON format
2. Backend validates data and checks for existing users
3. If validation passes, password is hashed using bcrypt
4. New user record is created with UUID v4 ID
5. Verification email is sent if email verification is enabled
6. Response includes JWT token and user data

#### `POST /auth/register/with-profile` - Registration with Profile

**Database Operations:**
- Same user creation as regular registration
- **Additional Write**: Creates user profile record
  ```sql
  INSERT INTO user_profiles (id, user_id, display_name, avatar_url, ...)
  VALUES (?, ?, ?, ?, ...)
  ```

**Data Flow:**
1. Frontend sends registration and profile data in JSON format
2. Backend validates data and checks for existing users
3. Transaction begins (to ensure both user and profile are created or neither)
4. User record is created
5. Profile record is created with link to user_id
6. Transaction commits
7. Response includes JWT token and complete user data with profile

#### `GET /auth/verify-email/{token}` - Email Verification

**Database Operations:**
- **Read**: Decodes token to get user ID (not a database operation)
- **Read**: Retrieves user to verify existence
  ```sql
  SELECT * FROM users WHERE id = ? LIMIT 1
  ```
- **Write**: Updates user verification status
  ```sql
  UPDATE users SET is_verified = true WHERE id = ?
  ```

**Data Flow:**
1. User clicks verification link in email with token
2. Backend decodes JWT token to extract user ID
3. User record is updated to mark as verified
4. Response indicates success or failure

#### `POST /auth/refresh-token` - Token Refresh

**Database Operations:**
- **Read**: Retrieves user to verify existence
  ```sql
  SELECT * FROM users WHERE id = ? LIMIT 1
  ```
- **No Write Operations**: This endpoint does not modify the database

### User Management Endpoints

#### `GET /users/me` - Get Current User

**Database Operations:**
- **Read**: Retrieves user data
  ```sql
  SELECT id, email, is_active, is_verified, user_type, subscription_tier, full_name, ...
  FROM users WHERE id = ? LIMIT 1
  ```

**Data Flow:**
1. Frontend includes JWT token in Authorization header
2. Backend decodes token to get user ID
3. User data is retrieved from database
4. Response returns user information

#### `PUT /users/me` - Update Current User

**Database Operations:**
- **Read**: Retrieves current user data
  ```sql
  SELECT * FROM users WHERE id = ? LIMIT 1
  ```
- **Write**: Updates user information
  ```sql
  UPDATE users SET email = ?, full_name = ?, ... WHERE id = ?
  ```

**Data Flow:**
1. Frontend sends updated user data with JWT token
2. Backend validates data and retrieves current user
3. User record is updated with new information
4. Response returns updated user data

#### `GET /users/me/profile` - Get User Profile

**Database Operations:**
- **Read**: Retrieves user profile
  ```sql
  SELECT * FROM user_profiles WHERE user_id = ? LIMIT 1
  ```

**Data Flow:**
1. Frontend includes JWT token in Authorization header
2. Backend decodes token to get user ID
3. Profile data is retrieved from database
4. Response returns profile information or 404 if no profile exists

#### `PUT /users/me/profile` - Update User Profile

**Database Operations:**
- **Read**: Checks if profile exists
  ```sql
  SELECT id FROM user_profiles WHERE user_id = ? LIMIT 1
  ```
- **Write**: Updates existing profile or creates new one
  ```sql
  UPDATE user_profiles SET display_name = ?, avatar_url = ?, ... WHERE user_id = ?
  ```
  or
  ```sql
  INSERT INTO user_profiles (id, user_id, display_name, ...) VALUES (?, ?, ?, ...)
  ```

**Data Flow:**
1. Frontend sends updated profile data with JWT token
2. Backend validates data and checks if profile exists
3. Profile is updated or created as needed
4. Response returns updated profile data

## Frontend-Backend Communication

The frontend communicates with the backend API through HTTP requests. This section explains the standard patterns and flows.

### Authentication Flow

1. **Login Process**:
   - Frontend collects credentials (email/password)
   - POST request to `/auth/login/json` with credentials
   - Backend validates and returns JWT token
   - Frontend stores token in localStorage or secure cookie
   - Token is included in all subsequent requests

2. **Registration Process**:
   - Frontend collects user information
   - POST request to `/auth/register` or `/auth/register/with-profile`
   - Backend creates user and returns JWT token
   - Frontend stores token and redirects to dashboard or profile completion

3. **Token Management**:
   - Frontend checks token expiration before requests
   - If expired, POST request to `/auth/refresh-token`
   - Updates stored token with new one
   - If refresh fails, redirects to login

### Data Fetching and Submission

1. **Request Headers**:
   - All authenticated requests include:
     ```
     Authorization: Bearer <jwt_token>
     Content-Type: application/json
     ```

2. **Error Handling**:
   - HTTP 401: Token invalid/expired → redirect to login
   - HTTP 403: Permission denied → show permission error
   - HTTP 404: Resource not found → show not found message
   - HTTP 422: Validation error → display field-specific errors
   - HTTP 500: Server error → display generic error message

3. **Typical Data Flow**:
   - Frontend initiates request with appropriate HTTP method
   - Backend authenticates and processes request
   - Backend returns data or error with appropriate HTTP status
   - Frontend updates UI based on response

## Recommended Database Enhancements

Based on the current database design, the following enhancements could improve functionality, organization, and scalability:

### 1. Separate Authentication Table

**Recommendation**: Create a dedicated `user_auth` table to separate authentication concerns from user information.

| Table | Fields | Benefits |
|-------|--------|----------|
| `user_auth` | `user_id`, `hashed_password`, `email`, `is_active`, `is_verified`, `last_login`, `failed_login_attempts` | Improved security, better separation of concerns |

### 2. User Activity Tracking

**Recommendation**: Create a `user_activity` table to track important user actions.

| Table | Fields | Benefits |
|-------|--------|----------|
| `user_activity` | `id`, `user_id`, `activity_type`, `timestamp`, `ip_address`, `user_agent`, `details` | Audit trail, security monitoring, user behavior analytics |

### 3. Organizations Table

**Recommendation**: Create a dedicated `organizations` table for enterprise accounts.

| Table | Fields | Benefits |
|-------|--------|----------|
| `organizations` | `id`, `name`, `domain`, `industry`, `size`, `billing_info`, `plan_id`, `admin_user_id` | Better organization management, multi-user accounts |

### 4. Permissions System

**Recommendation**: Implement a role-based access control system with these tables:

| Table | Fields | Benefits |
|-------|--------|----------|
| `roles` | `id`, `name`, `description` | Define standard roles |
| `permissions` | `id`, `name`, `description`, `resource_type` | Define granular permissions |
| `role_permissions` | `role_id`, `permission_id` | Connect roles to permissions |
| `user_roles` | `user_id`, `role_id`, `organization_id` | Assign roles to users (with context) |

### 5. Subscription Management

**Recommendation**: Create tables to manage subscription plans and billing:

| Table | Fields | Benefits |
|-------|--------|----------|
| `plans` | `id`, `name`, `description`, `monthly_price`, `annual_price`, `features` | Plan definitions |
| `subscriptions` | `id`, `user_id`, `organization_id`, `plan_id`, `start_date`, `end_date`, `status`, `payment_method` | Subscription tracking |
| `billing_history` | `id`, `user_id`, `organization_id`, `amount`, `currency`, `status`, `timestamp`, `invoice_id` | Payment history |

### 6. API Key Management

**Recommendation**: Create a table for API key management:

| Table | Fields | Benefits |
|-------|--------|----------|
| `api_keys` | `id`, `user_id`, `organization_id`, `name`, `key_hash`, `last_used`, `created_at`, `revoked_at`, `permissions` | Secure API access for programmatic use |

These recommended enhancements would provide better structure, improved security, and more robust functionality while keeping individual tables focused on specific concerns. 