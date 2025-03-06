# ARTINTEL LM Database Dictionary

## Introduction

This document provides a comprehensive reference for the database structure of the ARTINTEL LM platform. It details each table, field descriptions, data types, relationships, and the importance of each element in the database design.

## Table of Contents

1. [Users Table](#users-table)
2. [User Profiles Table](#user-profiles-table)
3. [Recommended Additional Tables](#recommended-additional-tables)
   - [API Usage Tracking](#api-usage-tracking)
   - [Authentication Logs](#authentication-logs)
   - [Security Audit Records](#security-audit-records)
   - [User Sessions](#user-sessions)
   - [Notification Preferences](#notification-preferences)
   - [Organizations](#organizations)
   - [Team Members](#team-members)
   - [API Keys](#api-keys)
   - [Model Access Permissions](#model-access-permissions)
   - [Subscription Plans](#subscription-plans)
   - [Billing Information](#billing-information)

---

## Users Table

### Overview

The `users` table is the cornerstone of the authentication system, storing essential user identity and access control information. It manages core user accounts and authentication state.

### Business Importance

This table is critical for:
- User identity verification and authentication
- Access control and permission management
- Account status tracking
- Subscription and tier management
- Compliance with data protection regulations

### Fields

| Field Name | Data Type | Constraints | Description | Business Importance |
|------------|-----------|-------------|-------------|---------------------|
| `id` | VARCHAR(36) | Primary Key | UUID string identifying the user | Provides a secure, globally unique identifier for each user account that won't expose sequence information |
| `email` | VARCHAR(255) | Unique, Index, Not Null | User's email address | Critical for account verification, password recovery, and notifications; serves as primary login credential |
| `hashed_password` | VARCHAR(255) | Not Null | Bcrypt-hashed password | Securely stores authentication credentials without exposing plaintext passwords; crucial for security compliance |
| `is_active` | BOOLEAN | Default True | Whether account is active | Controls access to the system; allows for account suspension without deletion |
| `is_verified` | BOOLEAN | Default False | Whether email has been verified | Ensures users have access to their registered email; reduces spam accounts and improves security |
| `is_superuser` | BOOLEAN | Default False | Whether user has admin rights | Controls access to administrative functions; critical for platform management and security |
| `user_type` | VARCHAR(50) | Default 'regular' | Type of user account (regular/enterprise) | Determines feature sets and billing strategies; supports business model segmentation |
| `subscription_tier` | VARCHAR(50) | Default 'free' | Subscription level | Controls feature access based on payment tier; central to monetization strategy |
| `full_name` | VARCHAR(255) | Nullable | User's full name | Improves user experience with personalization and identification in communications |
| `organization` | VARCHAR(255) | Nullable | User's organization | Supports enterprise features; enables team management and organization-level reporting |
| `created_at` | DATETIME | Default CURRENT_TIMESTAMP | Account creation timestamp | Supports analytics, account age tracking, and audit requirements |
| `updated_at` | DATETIME | On Update CURRENT_TIMESTAMP | Last update timestamp | Enables change tracking, helps identify stale accounts, supports audit trails |

### Technical Considerations

1. **UUID Implementation**: MySQL native UUID support was avoided in favor of VARCHAR(36) for better compatibility across database systems and simpler migration paths.

2. **Password Security**: Passwords are hashed using bcrypt with a cost factor of 12, providing strong security while maintaining reasonable performance.

3. **Indexing Strategy**: The email field is indexed to optimize login queries, which are the most frequent operations against this table.

4. **Soft Deletion**: The `is_active` field allows for account deactivation without data loss, supporting both user privacy requests and data retention requirements.

### Example Usage Scenarios

- **User Authentication**: When a user logs in, the system verifies their credentials against the `email` and `hashed_password` fields
- **Access Control**: Administrative interfaces check the `is_superuser` field before granting access to sensitive operations
- **Subscription Management**: Feature access is controlled by checking the `subscription_tier` field
- **Account Recovery**: Password reset workflows verify the user identity via the `email` field

---

## User Profiles Table

### Overview

The `user_profiles` table extends the core user information with additional, non-authentication related details. It stores personal preferences, profile information, and settings that enhance the user experience.

### Business Importance

This table supports:
- Enhanced personalization
- User experience customization
- Organization and team hierarchy management
- Feature preference storage
- Profile discovery and networking features

### Fields

| Field Name | Data Type | Constraints | Description | Business Importance |
|------------|-----------|-------------|-------------|---------------------|
| `id` | VARCHAR(36) | Primary Key | UUID string identifying the profile | Ensures each profile has a unique identifier separate from user id, supporting complex relationships |
| `user_id` | VARCHAR(36) | Foreign Key, Unique | Reference to users.id | Creates a one-to-one relationship with the user record; ensures referential integrity |
| `display_name` | VARCHAR(100) | | User's preferred display name | Supports personalization without exposing full name; improves user experience |
| `avatar_url` | VARCHAR(255) | Nullable | URL to user's profile picture | Enhances user interface with visual identification; increases platform engagement |
| `organization_name` | VARCHAR(255) | Nullable | Name of user's organization | Supports enterprise features and organizational grouping; enables team-based workflows |
| `organization_domain` | VARCHAR(255) | Nullable | Domain of user's organization | Facilitates auto-verification of enterprise accounts; supports security policies |
| `role` | VARCHAR(50) | Default 'developer' | User's role (developer, manager, etc.) | Enables role-based access control within organizations; supports team workflows |
| `bio` | TEXT | Nullable | User's biography or description | Enhances networking features; supports community building and expertise sharing |
| `security_questions` | TEXT | Nullable | JSON-encoded security questions/answers | Provides additional account recovery options; enhances account security |
| `preferences` | TEXT | Nullable | JSON-encoded user preferences | Stores customization settings; improves user experience through personalization |
| `created_at` | DATETIME | Default CURRENT_TIMESTAMP | Profile creation timestamp | Supports analytics and tracks profile completeness over time |
| `updated_at` | DATETIME | On Update CURRENT_TIMESTAMP | Last update timestamp | Helps identify recently changed profiles; supports activity tracking |

### Technical Considerations

1. **JSON Storage**: The `preferences` and `security_questions` fields use TEXT with JSON encoding, providing flexible schema evolution without requiring table alterations for new preference types.

2. **Cascading Deletion**: The foreign key to the users table uses `ON DELETE CASCADE` to ensure that when a user is deleted, their profile is automatically removed.

3. **Performance Optimization**: The `user_id` field is indexed to optimize the common operation of retrieving a profile for a given user.

4. **Flexible Role System**: The `role` field uses a string enum pattern instead of an integer, making the database more human-readable and allowing for easy addition of new roles.

### Example Usage Scenarios

- **User Settings**: The system fetches and stores theme preferences from the `preferences` JSON field
- **Team Management**: Organization admins can view members by querying profiles with matching `organization_name`
- **Profile Completion**: New users are guided to complete their profiles by filling optional fields like `bio` and `avatar_url`
- **Account Recovery**: Additional verification is provided through stored `security_questions` when users lose access

---

## Recommended Additional Tables

The current schema includes the essential tables for user authentication and profiles. Based on the application's requirements, the following additional tables are recommended to enhance functionality and security.

### API Usage Tracking

#### Overview

The `api_usage` table would track API calls made by each user, supporting rate limiting, billing, and usage analytics.

#### Recommended Fields

| Field Name | Data Type | Constraints | Description | Business Importance |
|------------|-----------|-------------|-------------|---------------------|
| `id` | VARCHAR(36) | Primary Key | UUID for the usage record | Provides unique identification for each API call record |
| `user_id` | VARCHAR(36) | Foreign Key | Reference to users.id | Links usage to specific users for accountability and billing |
| `endpoint` | VARCHAR(255) | Not Null | API endpoint called | Identifies which features are being used; supports feature analytics |
| `status_code` | SMALLINT | Not Null | HTTP status returned | Helps monitor API health and user experience issues |
| `response_time_ms` | INTEGER | | Response time in milliseconds | Supports performance monitoring and SLA compliance |
| `request_size_bytes` | INTEGER | | Size of the request payload | Helps identify abnormal usage patterns and optimize pricing models |
| `response_size_bytes` | INTEGER | | Size of the response payload | Supports bandwidth tracking and cost allocation |
| `timestamp` | DATETIME | Default CURRENT_TIMESTAMP | When the call occurred | Essential for time-based analytics and rate limiting |
| `ip_address` | VARCHAR(45) | | Caller's IP address | Supports security monitoring and geographic usage analytics |
| `user_agent` | VARCHAR(255) | | Client user agent | Helps identify platform usage patterns and client application versions |

#### Benefits

- Enables accurate billing based on actual API usage
- Supports rate limiting for different subscription tiers
- Provides data for optimizing API performance
- Helps identify suspicious activity patterns
- Generates insights on feature usage and adoption

### Authentication Logs

#### Overview

The `auth_logs` table would store detailed records of authentication attempts, supporting security monitoring and compliance requirements.

#### Recommended Fields

| Field Name | Data Type | Constraints | Description | Business Importance |
|------------|-----------|-------------|-------------|---------------------|
| `id` | VARCHAR(36) | Primary Key | UUID for the log entry | Unique identifier for each authentication event |
| `user_id` | VARCHAR(36) | Foreign Key, Nullable | Reference to users.id | Links authentication attempts to users when available |
| `email` | VARCHAR(255) | | Email used in the attempt | Records the credential used, even for failed attempts |
| `success` | BOOLEAN | Not Null | Whether authentication succeeded | Critical for security monitoring and breach detection |
| `failure_reason` | VARCHAR(100) | Nullable | Why authentication failed | Helps diagnose issues and identify attack patterns |
| `ip_address` | VARCHAR(45) | | Source IP address | Supports geographic analysis and abnormal access detection |
| `user_agent` | VARCHAR(255) | | Browser/client information | Helps identify suspicious clients and platform usage |
| `timestamp` | DATETIME | Default CURRENT_TIMESTAMP | When the attempt occurred | Enables time-based security analysis and reporting |
| `device_id` | VARCHAR(100) | Nullable | Unique device identifier | Supports device tracking and trusted device features |
| `location_data` | TEXT | Nullable | JSON with geolocation info | Enhances security analysis with geographic context |

#### Benefits

- Supports compliance with security standards requiring authentication logging
- Enables detection of brute force and credential stuffing attacks
- Provides audit trail for security investigations
- Supports abnormal access detection and alerting
- Enables analysis of login patterns and device usage

### Security Audit Records

#### Overview

The `security_audit` table would track security-relevant actions like password changes, role modifications, and settings updates.

#### Recommended Fields

| Field Name | Data Type | Constraints | Description | Business Importance |
|------------|-----------|-------------|-------------|---------------------|
| `id` | VARCHAR(36) | Primary Key | UUID for audit record | Unique identifier for each security event |
| `user_id` | VARCHAR(36) | Foreign Key | User who performed the action | Establishes accountability for security changes |
| `action_type` | VARCHAR(50) | Not Null | Type of security action | Categorizes events for analysis and reporting |
| `target_user_id` | VARCHAR(36) | Foreign Key, Nullable | User affected by the action | Identifies the subject of changes made by admins |
| `description` | TEXT | Not Null | Detailed description of action | Provides context for security review and investigation |
| `old_value` | TEXT | Nullable | Previous state if applicable | Enables reverting changes and understanding modifications |
| `new_value` | TEXT | Nullable | New state if applicable | Documents the result of the security action |
| `ip_address` | VARCHAR(45) | | Source IP address | Provides location context for security events |
| `timestamp` | DATETIME | Default CURRENT_TIMESTAMP | When the action occurred | Critical for timeline reconstruction during investigations |
| `metadata` | TEXT | Nullable | JSON with additional context | Stores extra information relevant to specific action types |

#### Benefits

- Creates immutable audit trail for security-relevant changes
- Supports compliance with regulations requiring change tracking
- Enables reconstruction of security events during investigations
- Provides accountability for administrative actions
- Helps detect unauthorized changes to security settings

### User Sessions

#### Overview

The `user_sessions` table would manage active user sessions, supporting features like session listing, forced logout, and concurrent session limits.

#### Recommended Fields

| Field Name | Data Type | Constraints | Description | Business Importance |
|------------|-----------|-------------|-------------|---------------------|
| `id` | VARCHAR(36) | Primary Key | UUID for the session | Unique identifier for each user session |
| `user_id` | VARCHAR(36) | Foreign Key | Reference to users.id | Links sessions to specific users |
| `token_hash` | VARCHAR(255) | Not Null | Hash of the session token | Supports token validation without storing actual tokens |
| `ip_address` | VARCHAR(45) | | Client IP address | Supports session validation and security monitoring |
| `user_agent` | VARCHAR(255) | | Client browser/app info | Helps identify the device and platform used |
| `device_name` | VARCHAR(100) | Nullable | User-friendly device name | Improves user experience when managing sessions |
| `is_active` | BOOLEAN | Default True | Whether session is still valid | Supports manual session termination and cleanup |
| `created_at` | DATETIME | Default CURRENT_TIMESTAMP | Session start time | Used for session timeout calculations and analytics |
| `expires_at` | DATETIME | Not Null | When session will expire | Supports enforcement of session lifetime policies |
| `last_activity` | DATETIME | | Last activity timestamp | Enables idle session timeout and usage analytics |
| `location_data` | TEXT | Nullable | JSON with location info | Enhances security with geographic context |

#### Benefits

- Enables users to view and manage their active sessions
- Supports forced logout of compromised sessions
- Allows limiting concurrent sessions by subscription tier
- Provides data for analyzing user activity patterns
- Enhances security by tracking session contexts

### Notification Preferences

#### Overview

The `notification_preferences` table would store detailed user preferences for different types of notifications across various channels.

#### Recommended Fields

| Field Name | Data Type | Constraints | Description | Business Importance |
|------------|-----------|-------------|-------------|---------------------|
| `id` | VARCHAR(36) | Primary Key | UUID for the preference | Unique identifier for each notification preference |
| `user_id` | VARCHAR(36) | Foreign Key | Reference to users.id | Links preferences to specific users |
| `notification_type` | VARCHAR(50) | Not Null | Category of notification | Categorizes different types of system notifications |
| `channel` | VARCHAR(20) | Not Null | Delivery channel (email, push, etc.) | Supports multi-channel notification strategies |
| `enabled` | BOOLEAN | Default True | Whether notifications are enabled | Gives users control over communication preferences |
| `frequency` | VARCHAR(20) | Default 'immediate' | Delivery timing preference | Supports batching and scheduled notification delivery |
| `quiet_hours_start` | TIME | Nullable | Start of do-not-disturb period | Enhances user experience by respecting quiet periods |
| `quiet_hours_end` | TIME | Nullable | End of do-not-disturb period | Completes the quiet hours preference window |
| `created_at` | DATETIME | Default CURRENT_TIMESTAMP | When preference was created | Supports tracking preference changes over time |
| `updated_at` | DATETIME | On Update CURRENT_TIMESTAMP | When preference was updated | Identifies recent changes to notification settings |

#### Benefits

- Provides granular control over communication preferences
- Reduces notification fatigue through user-controlled settings
- Supports compliance with communication regulations
- Improves user experience by respecting notification preferences
- Enables sophisticated notification delivery strategies

### Organizations

#### Overview

The `organizations` table would manage enterprise-level accounts, supporting team-based features, consolidated billing, and centralized administration.

#### Recommended Fields

| Field Name | Data Type | Constraints | Description | Business Importance |
|------------|-----------|-------------|-------------|---------------------|
| `id` | VARCHAR(36) | Primary Key | UUID for the organization | Unique identifier for each organization entity |
| `name` | VARCHAR(255) | Not Null | Organization's displayed name | Used for branding, UI display, and identification |
| `domain` | VARCHAR(255) | Unique, Index | Primary email domain | Enables auto-verification of employees and email domain validation |
| `plan_id` | VARCHAR(36) | Foreign Key | Reference to subscription plans | Links the organization to its enterprise subscription level |
| `billing_email` | VARCHAR(255) | Not Null | Email for billing communications | Separates financial communications from technical communications |
| `logo_url` | VARCHAR(255) | Nullable | URL to organization's logo | Supports branding customization in the platform interface |
| `website` | VARCHAR(255) | Nullable | Organization's website URL | Provides additional validation and contact information |
| `address` | TEXT | Nullable | Physical address | Required for compliance and billing documentation |
| `phone` | VARCHAR(50) | Nullable | Contact phone number | Provides additional contact method for account issues |
| `tax_id` | VARCHAR(100) | Nullable | VAT or tax identification | Essential for legal compliance in financial transactions |
| `created_at` | DATETIME | Default CURRENT_TIMESTAMP | When organization was created | Tracks account lifecycle and supports compliance |
| `updated_at` | DATETIME | On Update CURRENT_TIMESTAMP | When organization was updated | Identifies recent changes to organizational info |
| `is_active` | BOOLEAN | Default True | Whether organization is active | Supports temporary or permanent suspension of enterprise accounts |
| `metadata` | TEXT | Nullable | JSON-encoded additional data | Provides flexibility for storing specialized organization attributes |

#### Benefits

- Enables enterprise-level account management and billing
- Supports team-based access control and resource sharing
- Simplifies onboarding for employees through domain verification
- Facilitates organizational usage reporting and analytics
- Provides foundation for team collaboration features

### Team Members

#### Overview

The `team_members` table would manage relationships between users and organizations, defining roles and permissions within organizations.

#### Recommended Fields

| Field Name | Data Type | Constraints | Description | Business Importance |
|------------|-----------|-------------|-------------|---------------------|
| `id` | VARCHAR(36) | Primary Key | UUID for the team membership | Unique identifier for each organizational relationship |
| `organization_id` | VARCHAR(36) | Foreign Key | Reference to organizations.id | Links the membership to a specific organization |
| `user_id` | VARCHAR(36) | Foreign Key | Reference to users.id | Identifies the user who is a member of the organization |
| `role` | VARCHAR(50) | Not Null | Role within the organization | Determines permissions and access levels within the org |
| `title` | VARCHAR(255) | Nullable | Job title or position | Provides context in team directories and communications |
| `department` | VARCHAR(255) | Nullable | Department or team | Supports organizational structure and reporting |
| `invite_status` | VARCHAR(50) | Default 'accepted' | Invitation status | Tracks pending invitations and membership status |
| `invite_email` | VARCHAR(255) | Nullable | Email used for invitation | Allows inviting users not yet registered in the system |
| `invited_by` | VARCHAR(36) | Foreign Key, Nullable | User who sent invitation | Establishes accountability for team composition |
| `is_admin` | BOOLEAN | Default False | Whether user is org admin | Controls administrative privileges within the organization |
| `created_at` | DATETIME | Default CURRENT_TIMESTAMP | When membership began | Tracks membership history and tenure |
| `updated_at` | DATETIME | On Update CURRENT_TIMESTAMP | When membership was updated | Identifies recent role or status changes |
| `last_active` | DATETIME | Nullable | Last activity in org context | Helps identify inactive team members |
| `permissions` | TEXT | Nullable | JSON-encoded permissions | Provides granular access control within the organization |

#### Benefits

- Supports hierarchical permission structures within organizations
- Enables team-based resource allocation and access control
- Facilitates departmental usage tracking and cost allocation
- Provides audit trail for organizational role changes
- Supports complex enterprise team structures and workflows

### API Keys

#### Overview

The `api_keys` table would manage programmatic access credentials, supporting secure machine-to-machine authentication for integrations.

#### Recommended Fields

| Field Name | Data Type | Constraints | Description | Business Importance |
|------------|-----------|-------------|-------------|---------------------|
| `id` | VARCHAR(36) | Primary Key | UUID for the API key | Unique identifier for each API key |
| `user_id` | VARCHAR(36) | Foreign Key, Nullable | Reference to users.id | Links personal API keys to specific users |
| `organization_id` | VARCHAR(36) | Foreign Key, Nullable | Reference to organizations.id | Links organizational API keys to specific organizations |
| `name` | VARCHAR(255) | Not Null | User-defined name for the key | Helps users identify keys in their dashboard |
| `key_prefix` | VARCHAR(10) | Unique | First few chars of key for identification | Allows users to identify which key was used without revealing full key |
| `key_hash` | VARCHAR(255) | Not Null | Hashed value of the API key | Securely stores the key without revealing the actual value |
| `scopes` | TEXT | Not Null | JSON-encoded permission scopes | Defines exactly what actions the key can perform |
| `expires_at` | DATETIME | Nullable | Expiration timestamp | Supports temporary access and security best practices |
| `last_used_at` | DATETIME | Nullable | When key was last used | Helps identify unused keys that should be rotated |
| `created_at` | DATETIME | Default CURRENT_TIMESTAMP | When key was created | Supports key rotation policies and audit requirements |
| `created_by` | VARCHAR(36) | Foreign Key | User who created the key | Establishes accountability for key creation |
| `is_active` | BOOLEAN | Default True | Whether key is active | Supports immediate revocation without deletion |
| `ip_whitelist` | TEXT | Nullable | JSON list of allowed IPs | Enhances security by restricting access to specific networks |
| `rate_limit` | INTEGER | Nullable | Custom rate limit for this key | Supports fine-grained usage control per integration |

#### Benefits

- Enables secure programmatic access to the platform
- Supports integration with external systems and workflows
- Provides granular control over API access permissions
- Enables audit trail for machine-to-machine interactions
- Supports secure key rotation and access management

### Model Access Permissions

#### Overview

The `model_permissions` table would define which users or organizations have access to specific AI models or model types, supporting access control and feature availability.

#### Recommended Fields

| Field Name | Data Type | Constraints | Description | Business Importance |
|------------|-----------|-------------|-------------|---------------------|
| `id` | VARCHAR(36) | Primary Key | UUID for the permission | Unique identifier for each permission record |
| `user_id` | VARCHAR(36) | Foreign Key, Nullable | Reference to users.id | Links permission to a specific user (if user-specific) |
| `organization_id` | VARCHAR(36) | Foreign Key, Nullable | Reference to organizations.id | Links permission to a specific organization (if org-wide) |
| `model_id` | VARCHAR(255) | Not Null | Identifier for the AI model | Specifies which model the permission applies to |
| `permission_type` | VARCHAR(50) | Not Null | Type of access (read, fine-tune, etc.) | Defines what operations are allowed on the model |
| `max_tokens` | INTEGER | Nullable | Token limit for this model | Allows setting usage limits at the model level |
| `priority_access` | BOOLEAN | Default False | Whether user has priority access | Supports premium features like higher request priority |
| `cost_multiplier` | DECIMAL(10,4) | Default 1.0 | Billing multiplier for this model | Supports custom pricing for specific users/models |
| `granted_at` | DATETIME | Default CURRENT_TIMESTAMP | When permission was granted | Tracks history of access permissions |
| `granted_by` | VARCHAR(36) | Foreign Key, Nullable | User who granted permission | Establishes accountability for permission changes |
| `expires_at` | DATETIME | Nullable | When permission expires | Supports temporary access grants |
| `is_active` | BOOLEAN | Default True | Whether permission is active | Allows temporary suspension without deletion |
| `usage_conditions` | TEXT | Nullable | JSON with special conditions | Supports complex usage rules beyond basic permissions |

#### Benefits

- Enables fine-grained access control to AI models
- Supports differentiated pricing and access tiers
- Facilitates compliance with licensing and usage agreements
- Provides foundation for usage accounting and billing
- Supports model-specific feature restrictions

### Subscription Plans

#### Overview

The `subscription_plans` table would define the available subscription tiers, their features, and pricing, supporting the platform's monetization strategy.

#### Recommended Fields

| Field Name | Data Type | Constraints | Description | Business Importance |
|------------|-----------|-------------|-------------|---------------------|
| `id` | VARCHAR(36) | Primary Key | UUID for the plan | Unique identifier for each subscription plan |
| `name` | VARCHAR(100) | Not Null | Plan name for display | Used in UI and customer communications |
| `code` | VARCHAR(50) | Unique, Not Null | Internal plan identifier | Used for programmatic identification of plans |
| `description` | TEXT | Nullable | Details about the plan | Provides customer-facing plan information |
| `price_monthly` | DECIMAL(10,2) | Not Null | Monthly price in base currency | Supports monthly billing option |
| `price_yearly` | DECIMAL(10,2) | Not Null | Yearly price in base currency | Supports annual billing with potential discount |
| `currency` | VARCHAR(3) | Default 'USD' | Currency code | Supports multi-currency billing |
| `billing_interval` | VARCHAR(20) | Default 'month' | Billing frequency | Defines standard billing cycle |
| `trial_days` | INTEGER | Default 0 | Free trial period | Supports try-before-you-buy strategy |
| `api_calls_limit` | INTEGER | Nullable | Max API calls per period | Defines usage quotas for this tier |
| `storage_limit_mb` | INTEGER | Nullable | Storage allocation in MB | Defines storage quotas for this tier |
| `max_models` | INTEGER | Nullable | Number of custom models allowed | Limits model creation based on tier |
| `max_users` | INTEGER | Nullable | User limit for org plans | Limits team size for organizational plans |
| `features` | TEXT | Not Null | JSON array of included features | Defines what features are available in this tier |
| `is_active` | BOOLEAN | Default True | Whether plan is currently offered | Controls plan visibility and availability |
| `sort_order` | INTEGER | Default 0 | Display order in plan lists | Controls presentation of plans in UI |
| `created_at` | DATETIME | Default CURRENT_TIMESTAMP | When plan was created | Supports plan history and version tracking |
| `updated_at` | DATETIME | On Update CURRENT_TIMESTAMP | When plan was updated | Identifies recent plan changes |

#### Benefits

- Centralizes subscription plan definitions
- Supports tiered pricing strategy
- Facilitates comparison of plan features
- Enables automated enforcement of plan limitations
- Provides foundation for upgrade/downgrade workflows

### Billing Information

#### Overview

The `billing_info` table would store payment methods, billing addresses, and related information to support the subscription billing process.

#### Recommended Fields

| Field Name | Data Type | Constraints | Description | Business Importance |
|------------|-----------|-------------|-------------|---------------------|
| `id` | VARCHAR(36) | Primary Key | UUID for the billing record | Unique identifier for each billing record |
| `user_id` | VARCHAR(36) | Foreign Key, Nullable | Reference to users.id | Links billing to individual user (for personal plans) |
| `organization_id` | VARCHAR(36) | Foreign Key, Nullable | Reference to organizations.id | Links billing to organization (for team plans) |
| `payment_provider` | VARCHAR(50) | Not Null | Payment processor used | Identifies which payment system handles the account |
| `payment_method_id` | VARCHAR(255) | Nullable | ID in payment provider system | References stored payment method in payment processor |
| `payment_method_type` | VARCHAR(50) | Nullable | Type of payment method | Identifies payment method category (card, bank, etc.) |
| `last_four` | VARCHAR(4) | Nullable | Last four digits of card | Provides user-friendly card identification |
| `expiry_month` | SMALLINT | Nullable | Card expiration month | Used for expiry warnings and updates |
| `expiry_year` | SMALLINT | Nullable | Card expiration year | Used for expiry warnings and updates |
| `cardholder_name` | VARCHAR(255) | Nullable | Name on payment method | Required for payment processing and verification |
| `billing_address_line1` | VARCHAR(255) | Nullable | Street address | Required for payment processing and tax compliance |
| `billing_address_line2` | VARCHAR(255) | Nullable | Additional address info | Supports complete address information |
| `billing_city` | VARCHAR(255) | Nullable | City | Required for payment processing and tax compliance |
| `billing_state` | VARCHAR(255) | Nullable | State/province | Required for payment processing and tax compliance |
| `billing_postal_code` | VARCHAR(20) | Nullable | Postal/ZIP code | Required for payment processing and tax compliance |
| `billing_country` | VARCHAR(2) | Nullable | Country code | Required for payment processing and tax compliance |
| `tax_id` | VARCHAR(100) | Nullable | VAT or tax number | Supports tax exemptions and business billing |
| `is_default` | BOOLEAN | Default True | Whether this is the default method | Supports multiple payment methods per account |
| `created_at` | DATETIME | Default CURRENT_TIMESTAMP | When record was created | Tracks payment method history |
| `updated_at` | DATETIME | On Update CURRENT_TIMESTAMP | When record was updated | Identifies recent payment method changes |

#### Benefits

- Supports recurring subscription billing
- Facilitates payment method management
- Provides necessary data for tax compliance
- Enables regional pricing and tax handling
- Supports multiple payment methods per account

---

## Implementation Considerations

When implementing these database tables, consider the following best practices:

1. **Start with Core Tables**: Begin with the `users` and `user_profiles` tables, which provide the foundation for authentication and personalization.

2. **Implement Incrementally**: Add additional tables based on immediate needs rather than implementing everything at once. This allows for testing and validation of each component.

3. **Consider Partitioning**: For tables expected to grow large (like `api_usage` and `auth_logs`), consider implementing partitioning strategies by date to maintain performance.

4. **Regular Archiving**: Implement policies for archiving old data from logging tables to maintain performance while preserving historical data for compliance.

5. **Encryption**: For fields containing sensitive information, consider implementing column-level encryption, especially for production environments.

6. **Indexes**: Beyond primary keys, add indexes strategically for fields frequently used in WHERE clauses or joins to optimize query performance.

7. **Foreign Key Constraints**: Implement appropriate ON DELETE and ON UPDATE behaviors for foreign keys to maintain data integrity during changes.

8. **Monitoring**: Set up database monitoring to track growth patterns and performance metrics, allowing for proactive optimization.

