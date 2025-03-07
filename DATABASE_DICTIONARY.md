# **ARTINTEL LM — Comprehensive Database Dictionary**

## **1. Introduction**

This document presents a **comprehensive data model** for the Artintel LM platform, a web application designed to simplify the discovery, selection, fine-tuning, deployment, and monitoring of AI language models (both LLMs and SLMs). The platform serves both **individual customers** and **enterprise organizations** requiring security, data control, and seamless integrations.

### **Primary Goals of this Data Model**
- Provide a **robust authentication** foundation (users, profiles, sessions).
- Enable **enterprise-ready features** like organizations, team management, and RBAC (role-based access control).
- Support **subscription management** (tiers, plans, billing) for monetization.
- Handle **advanced logging and auditing** for compliance, security, and analytics (HIPAA, GDPR, etc.).
- Remain **scalable** and **extensible** — apply partitioning, indexing, archiving strategies for performance and cost optimization.

> **Note**: This data model is in an **early planning phase**. Further refinements and reiterations will be applied as the application evolves.

---

## **2. Table of Contents**

1. [Users Table](#users-table)  
2. [User Profiles Table](#user-profiles-table)  
3. [Organizations Table](#organizations-table)  
4. [Team Members Table](#team-members-table)  
5. [RBAC (Roles, Permissions, Role Permissions, User Roles)](#rbac)  
   - [Roles Table](#roles-table)  
   - [Permissions Table](#permissions-table)  
   - [Role Permissions Table](#role-permissions-table)  
   - [User Roles Table](#user-roles-table)  
6. [Subscription Plans Table](#subscription-plans-table)  
7. [Subscriptions Table](#subscriptions-table)  
8. [Billing History Table](#billing-history-table)  
9. [User Sessions Table](#user-sessions-table)  
10. [API Usage Table](#api-usage-table)  
11. [Authentication Logs Table](#authentication-logs-table)  
12. [Security Audit Table](#security-audit-table)  
13. [Notification Preferences Table](#notification-preferences-table)  
14. [API Keys Table](#api-keys-table)  
15. [Model Access Permissions Table](#model-access-permissions-table)  
16. [Billing Info Table](#billing-info-table)  
17. [Implementation Considerations](#implementation-considerations)  
18. [Relationships & Diagram](#relationships--diagram)

---

## **3. Users Table**

### **Overview**
Stores core user identities and access control fields. Forms the **foundation of authentication** and user-specific data.

### **Business Importance**
- **Authentication & Access**: Verifies identity, credentials.
- **Subscription & Tier Management**: Tracks user’s tier (free, advanced, etc.).
- **Compliance**: Enforces email verification, superuser privileges, etc.

### **Fields**

| Field Name        | Data Type    | Constraints                  | Description                                                                                           | Business Importance                                                                                 |
|-------------------|-------------|------------------------------|-------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------|
| **id**            | VARCHAR(36) | **PK**                       | UUID for each user (avoids sequential IDs).                                                           | Ensures globally unique user IDs, limiting data exposure.                                             |
| **email**         | VARCHAR(255)| **Unique, Index, Not Null**  | User's email address.                                                                                 | Primary credential for login, password resets, notifications.                                         |
| **hashed_password**|VARCHAR(255)| **Not Null**                 | Bcrypt-hashed password.                                                                               | Secures authentication while preventing plaintext password storage.                                   |
| **is_active**     | BOOLEAN     | **Default True**             | Whether account is active.                                                                            | Allows soft deactivation of users without losing their data.                                          |
| **is_verified**   | BOOLEAN     | **Default False**            | Whether email is verified.                                                                            | Ensures email ownership, reduces spam accounts, improves trust.                                       |
| **is_superuser**  | BOOLEAN     | **Default False**            | Whether user has admin rights.                                                                        | Grants elevated privileges; essential for platform management.                                        |
| **user_type**     | VARCHAR(50) | **Default 'regular'**        | Category (regular/enterprise).                                                                        | Differentiates feature sets or billing models based on account type.                                  |
| **subscription_tier**|VARCHAR(50)|**Default 'free'**           | Subscription level (free, advanced, etc.).                                                            | Controls feature/billing entitlements for the user.                                                   |
| **full_name**     | VARCHAR(255)| **Nullable**                 | User’s full name.                                                                                     | Improves personalization, communications.                                                              |
| **organization**  | VARCHAR(255)| **Nullable**                 | User’s organization name.                                                                             | Quick link to enterprise or team usage, used for smaller user org logic.                              |
| **created_at**    | DATETIME    | **Default NOW()**            | When account was created.                                                                             | Analytics, auditing, compliance.                                                                       |
| **updated_at**    | DATETIME    | **On Update NOW()**          | Last update timestamp.                                                                                | Tracks user record changes over time.                                                                  |

### **Technical Considerations**
1. **UUID Storage**: Stored as `VARCHAR(36)` for cross-DB compatibility.  
2. **Password Security**: Bcrypt with cost factor 12 for balanced performance and security.  
3. **Indexing**: Email is indexed (most frequent WHERE query for login).  
4. **Soft Deletion**: `is_active` to disable user without removing data.

### **Example Usage Scenarios**
- **Login**: Validate `email` + `hashed_password`.  
- **Subscription Upgrades**: Check `subscription_tier` to determine new features.  
- **Admin Portals**: `is_superuser` gates admin routes.  
- **User Deactivation**: Flip `is_active = false` if account is suspected compromised.

---

## **4. User Profiles Table**

### **Overview**
Holds **non-auth** details (avatar, display name, preference JSON, etc.) that can evolve independently from core credentials.

### **Business Importance**
- **Enhanced Personalization**: Bio, avatar, display name.  
- **Organization-Specific**: Domain or internal role references.  
- **Separation from Security**: Minimizes clutter in the `users` table.

### **Fields**

| Field Name           | Data Type    | Constraints                  | Description                                                                | Business Importance                                                             |
|----------------------|-------------|------------------------------|----------------------------------------------------------------------------|---------------------------------------------------------------------------------|
| **id**               | VARCHAR(36) | **PK**                       | UUID for the profile record.                                               | Separates profile from core user data; flexible expansions.                     |
| **user_id**          | VARCHAR(36) | **FK, Unique**              | One-to-one link to `users.id`.                                             | Ensures each user has at most one profile.                                      |
| **display_name**     | VARCHAR(100)|                              | A user-chosen handle or alias.                                             | Hides real names publicly if needed.                                            |
| **avatar_url**       | VARCHAR(255)| **Nullable**                 | Link to profile pic.                                                        | Boosts user engagement & identity.                                              |
| **organization_name**| VARCHAR(255)| **Nullable**                 | The user’s stated org name.                                                | Helps team/enterprise flows if user belongs to smaller org usage.               |
| **organization_domain**|VARCHAR(255)|**Nullable**                 | Domain for enterprise checks.                                              | Potential auto-validation for enterprise members.                               |
| **role**             | VARCHAR(50) | **Default 'developer'**      | Self-declared role (manager, dev, etc.).                                    | Allows finer control if not fully covered by RBAC or for quick display.         |
| **bio**              | TEXT        | **Nullable**                 | Freeform user description.                                                 | Encourages community & networking.                                              |
| **security_questions**|TEXT        | **Nullable**                 | JSON array of Q&A for password recovery.                                    | Secondary authentication factor.                                                |
| **preferences**      | TEXT        | **Nullable**                 | JSON-encoded user settings.                                                | Allows storing theme, notification prefs, feature toggles, etc.                 |
| **created_at**       | DATETIME    | **Default NOW()**            | Profile creation date.                                                     | Auditing & analytics.                                                           |
| **updated_at**       | DATETIME    | **On Update NOW()**          | Last time this record changed.                                             | Tracks how recently user details changed.                                       |

### **Technical Considerations**
1. **JSON Storage**: For preferences, security questions, easy to store flexible settings.  
2. **Cascading Deletion**: `ON DELETE CASCADE` from `user_id` ensures profile is removed if user is deleted.  
3. **Performance**: Index on `user_id` for quick lookups.  
4. **Flexibility**: The `role` field here is a user-facing label, separate from official RBAC roles.

### **Example Usage Scenarios**
- **User Customization**: Query `preferences` to build personalized dashboards.  
- **Enterprise Branding**: `organization_domain` helps auto-check domain-based enterprise membership.  
- **Profile Editing**: Users set `display_name`, `bio`, etc. in an account settings page.  
- **Account Recovery**: Security questions can be used as a second factor.

---

## **5. Organizations Table**

### **Overview**
Used to handle **enterprise-level accounts** (larger companies, multi-team). Distinct from user’s personal data.

### **Business Importance**
- **Billing & Admin**: Ties to subscription plans, consolidated billing.  
- **Team Collaboration**: All members grouped under an org for resource sharing.  
- **Enterprise Onboarding**: Domain-based membership or invites.

### **Fields**

| Field Name      | Data Type    | Constraints         | Description                                                         | Business Importance                                                        |
|-----------------|-------------|---------------------|---------------------------------------------------------------------|----------------------------------------------------------------------------|
| **id**          | VARCHAR(36) | **PK**              | UUID for the organization.                                          | Distinct identification for each org entity.                               |
| **name**        | VARCHAR(255)| **Not Null**        | The organization’s displayed name.                                  | Branding, display in dashboards, billing, etc.                              |
| **domain**      | VARCHAR(255)| **Unique, Index**   | Primary email domain for potential auto-verification.               | Simplifies enterprise user sign-up, security checks.                        |
| **plan_id**     | VARCHAR(36) | **FK**              | Links to a subscription plan for the org.                           | Ties enterprise to a plan (some have advanced usage or seats).             |
| **billing_email**|VARCHAR(255)| **Not Null**        | Email for financial communications.                                 | Distinguishes finance contact from main admin.                              |
| **logo_url**    | VARCHAR(255)| **Nullable**        | Link to org’s logo.                                                 | UI branding for enterprise members.                                         |
| **website**     | VARCHAR(255)| **Nullable**        | Org website URL.                                                    | Additional context, marketing references.                                   |
| **address**     | TEXT        | **Nullable**        | Physical address.                                                   | Tax, compliance, shipping (if needed).                                      |
| **phone**       | VARCHAR(50) | **Nullable**        | Contact phone number.                                               | Another support or billing contact method.                                  |
| **tax_id**      | VARCHAR(100)| **Nullable**        | VAT or tax identifier.                                              | Regulatory compliance, billing.                                             |
| **created_at**  | DATETIME    | **Default NOW()**   | Org creation date.                                                  | Lifecycle tracking for the org’s subscription.                              |
| **updated_at**  | DATETIME    | **On Update NOW()** | Last update.                                                        | Tracks changes to org details.                                              |
| **is_active**   | BOOLEAN     | **Default True**    | Whether the org is active.                                          | Temporarily or permanently suspend an org’s usage if needed.                |
| **metadata**    | TEXT        | **Nullable**        | JSON for extra attributes.                                          | Flexible extension (department structure, compliance notes).                |

### **Technical Considerations**
1. **Domain**: “Unique, Index” for quick domain-based queries.  
2. **Foreign Key**: `plan_id` references some form of subscription plan.  
3. **Soft Deactivation**: `is_active` to suspend or restore entire org usage quickly.

### **Example Usage Scenarios**
- **Enterprise Onboarding**: Check `domain` to auto-assign employees.  
- **Billing & Plan Management**: `plan_id` references advanced or custom enterprise plan.  
- **Organization Branding**: Show `logo_url`, `name` in dashboards for all members.

---

## **6. Team Members Table**

### **Overview**
Maps **individual users** to **organizations**, specifying roles, departmental info, and administrative rights within that org.

### **Business Importance**
- **Hierarchical Permission**: Distinguishes org admins, managers, staff.  
- **Cross-Org Usage**: A single user can belong to multiple orgs.  
- **Collaboration**: Tracks membership invites, statuses (pending, accepted).

### **Fields**

| Field Name       | Data Type    | Constraints         | Description                                                         | Business Importance                                                   |
|------------------|-------------|---------------------|---------------------------------------------------------------------|-----------------------------------------------------------------------|
| **id**           | VARCHAR(36) | **PK**              | Unique membership record ID.                                        | Distinguishes each user-org relationship.                             |
| **organization_id**|VARCHAR(36)| **FK**              | Links to `organizations.id`.                                        | Ties membership to a single org.                                      |
| **user_id**      | VARCHAR(36) | **FK**              | Links to `users.id`.                                                | Identifies which user is in the org.                                  |
| **role**         | VARCHAR(50) | **Not Null**        | Role within that specific org.                                      | E.g. “Org admin”, “Finance manager”.                                  |
| **title**        | VARCHAR(255)| **Nullable**        | Job title or position.                                              | Improves clarity for org directory.                                   |
| **department**   | VARCHAR(255)| **Nullable**        | Department or sub-team.                                             | Facilitates internal grouping and usage reporting.                    |
| **invite_status**| VARCHAR(50) | **Default 'accepted'**| Tracks membership invitation state.                                 | Could be “pending,” “revoked,” or “accepted.”                         |
| **invite_email** | VARCHAR(255)| **Nullable**        | Email used for invitation.                                          | Allows sending invites to non-registered addresses.                   |
| **invited_by**   | VARCHAR(36) | **FK, Nullable**    | The user who initiated the invite.                                  | Accountability for who is bringing new members in.                    |
| **is_admin**     | BOOLEAN     | **Default False**   | If user is an org admin.                                            | Grants advanced privileges within the org.                            |
| **created_at**   | DATETIME    | **Default NOW()**   | When membership began.                                              | Auditing, lifecycle tracking.                                         |
| **updated_at**   | DATETIME    | **On Update NOW()** | Last membership update time.                                        | Noting role changes, re-invites, etc.                                 |
| **last_active**  | DATETIME    | **Nullable**        | Last org-level activity from user.                                  | Possibly used for auto-removing inactive members.                     |
| **permissions**  | TEXT        | **Nullable**        | JSON-based extra permissions.                                       | Could store special custom org-level perms beyond role defaults.      |

### **Technical Considerations**
1. **Many-to-Many**: Single user can belong to multiple orgs, and an org can have multiple users.  
2. **Invitation Flow**: `invite_status`, `invite_email`, `invited_by` help manage org membership lifecycles.  
3. **Index**: On `(organization_id, user_id)` to quickly retrieve membership or check duplicates.

### **Example Usage Scenarios**
- **Organizational Admin Tools**: List all members by `organization_id`.  
- **Enterprise Onboarding**: If `invite_status = pending`, user sees an invite link.  
- **Departmental Permissions**: `department` can group members, or `permissions` JSON for fine-tuning.

---

## **7. RBAC**

To support robust **Role-Based Access Control**, we have four tables: **Roles**, **Permissions**, **Role Permissions**, and **User Roles**. This structure ensures a fine-grained security model.

### **7.1 Roles Table**

#### **Overview**
Declares named roles (e.g., "Admin", "Viewer") available system-wide or by organization.

#### **Business Importance**
- **Security**: High-level grouping of privileges.  
- **Extensibility**: Custom or system roles to handle new use cases.

#### **Fields**

| Field Name    | Data Type    | Constraints                | Description                                    | Business Importance                                      |
|---------------|-------------|----------------------------|------------------------------------------------|----------------------------------------------------------|
| **id**        | VARCHAR(36) | **PK**                     | UUID for the role.                              | Distinguishes each defined role.                         |
| **name**      | VARCHAR(100)| **Not Null, Unique**       | Role name, e.g. "Admin".                        | Direct label for assigning to users.                     |
| **description**| TEXT        | **Nullable**               | Explanation of role purpose.                     | Aids admin clarity.                                      |
| **is_system** | BOOLEAN     | **Default True**           | Whether role is system-defined or custom.       | Distinguishes standard roles from user-defined.          |
| **created_at**| DATETIME    | **Default NOW()**          | Creation date.                                  | Tracks addition of new roles.                            |
| **updated_at**| DATETIME    | **On Update NOW()**        | Last updated.                                   | Tracks role definition changes.                          |

#### **Technical Considerations**
- Typically small table, minimal performance overhead.  
- Might store an `organization_id` if roles are org-specific, or keep them global.

#### **Example Usage Scenarios**
- **Built-In System Roles**: "SuperAdmin", "Support", "Viewer."  
- **Custom**: Org-defined roles with specialized needs.

---

### **7.2 Permissions Table**

#### **Overview**
Lists individual **actions** (e.g., "create:models", "delete:teams") that can be granted to roles.

#### **Business Importance**
- **Fine-Grained Control**: Breaking system capabilities into discrete permissions.  
- **Security & Compliance**: Easy to see which actions exist, who can do them.

#### **Fields**

| Field Name      | Data Type    | Constraints               | Description                                 | Business Importance                                 |
|-----------------|-------------|---------------------------|---------------------------------------------|-----------------------------------------------------|
| **id**          | VARCHAR(36) | **PK**                    | UUID for the permission.                    | Distinguishes each unique permission.               |
| **name**        | VARCHAR(100)| **Not Null, Unique**      | Permission identifier, e.g. "read:billing".  | The fundamental label used in code checks.          |
| **description** | TEXT        | **Nullable**              | Explains the permission’s domain.            | Aids devs and admins in understanding usage.        |
| **resource_type**|VARCHAR(50) | **Not Null**              | Category of resource (e.g., "billing").      | Helps group permissions by resource.                |
| **created_at**  | DATETIME    | **Default NOW()**         | When the permission was created.            | Lifecycle tracking.                                 |

#### **Technical Considerations**
- Could store them in code, but DB approach allows admin expansions.  
- Possibly a “scope” concept for grouping.  
- Minimal indexing needed; primarily used during role-permission lookup.

#### **Example Usage Scenarios**
- **In-App Checks**: "User must have `permission_id` matching 'create:models' to proceed."  
- **Admin UI**: Show full permission list to customize roles.

---

### **7.3 Role Permissions Table**

#### **Overview**
Bridges **Roles** and **Permissions** in a **many-to-many** relationship.

#### **Business Importance**
- Central place to define which role can do which actions.  
- Easy to add or revoke a permission from a role without changing code.

#### **Fields**

| Field Name       | Data Type    | Constraints          | Description                                                     | Business Importance                                              |
|------------------|-------------|----------------------|-----------------------------------------------------------------|------------------------------------------------------------------|
| **id**           | VARCHAR(36) | **PK**               | Unique ID for the mapping.                                      | Identifies the role-permission link.                            |
| **role_id**      | VARCHAR(36) | **FK**               | Reference to `roles.id`.                                        | Ties the mapping to a specific role.                            |
| **permission_id**| VARCHAR(36) | **FK**               | Reference to `permissions.id`.                                  | Ties the mapping to a specific permission.                      |
| **created_at**   | DATETIME    | **Default NOW()**    | When mapping was created.                                       | Tracks changes for auditing.                                    |

#### **Technical Considerations**
- Index `(role_id, permission_id)` for quick retrieval.  
- Add `UNIQUE (role_id, permission_id)` to prevent duplicates.

#### **Example Usage Scenarios**
- **Adding a permission**: Insert a row for `(role_id, permission_id)`.  
- **Revoking**: Delete that row.

---

### **7.4 User Roles Table**

#### **Overview**
Assigns one or more **roles** to **users**, potentially within a specific organization context.

#### **Business Importance**
- **Dynamic Access**: One user can have multiple roles in different orgs or system-wide.  
- **Team-based**: Organization-based roles can differ from global roles.

#### **Fields**

| Field Name        | Data Type    | Constraints           | Description                                             | Business Importance                                                         |
|-------------------|-------------|-----------------------|---------------------------------------------------------|-----------------------------------------------------------------------------|
| **id**            | VARCHAR(36) | **PK**                | Unique assignment ID.                                   | Distinguishes each user-role assignment.                                   |
| **user_id**       | VARCHAR(36) | **FK**                | Points to `users.id`.                                   | Identifies the user receiving the role.                                    |
| **role_id**       | VARCHAR(36) | **FK**                | Points to `roles.id`.                                   | Identifies which role is assigned.                                         |
| **organization_id**|VARCHAR(36) | **FK, Nullable**      | If role is specific to an org.                          | Allows the same user to hold a different role per org.                     |
| **assigned_by**   | VARCHAR(36) | **FK, Nullable**      | Who assigned this role.                                 | Accountability for admin actions.                                          |
| **created_at**    | DATETIME    | **Default NOW()**     | When role was assigned.                                 | Auditing; shows how user’s privileges changed over time.                   |
| **expires_at**    | DATETIME    | **Nullable**          | Optional auto-expiry date.                              | Temporary roles for project-based or short-term tasks.                     |

#### **Technical Considerations**
- Potential `UNIQUE (user_id, role_id, organization_id)` to avoid duplicates.  
- Possibly add partial indexing if roles are frequently checked by org.

#### **Example Usage Scenarios**
- **Global Admin**: `organization_id` is null, role is system-wide "Admin."  
- **Org Manager**: A user can be "Manager" in `organization_id=ABC` but "Viewer" in another org.

---

## **8. Subscription Plans Table**

*(Sometimes referred to as `plans` in short.)*

### **Overview**
Defines **tiers** (free, advanced, enterprise) or custom plan data, including pricing, feature sets, usage constraints.

### **Business Importance**
- **Monetization**: Central place to store monthly/yearly costs, usage quotas.  
- **Feature Entitlements**: Helps the system know which features are active on each plan.

### **Fields**

| Field Name      | Data Type    | Constraints            | Description                                                          | Business Importance                                                 |
|-----------------|-------------|------------------------|----------------------------------------------------------------------|---------------------------------------------------------------------|
| **id**          | VARCHAR(36) | **PK**                 | UUID for each plan.                                                 | Uniquely identifies a plan (e.g. "free" or "enterprise").           |
| **name**        | VARCHAR(100)| **Not Null**           | Display name (e.g. "Advanced").                                      | Shown in marketing/billing pages.                                   |
| **code**        | VARCHAR(50) | **Unique, Not Null**   | Internal plan code (e.g. "ADV").                                     | Programmatically references the plan.                               |
| **description** | TEXT        | **Nullable**           | Further details for user reference.                                  | Clear plan explanations in UI.                                      |
| **price_monthly**|DECIMAL(10,2)|**Not Null**           | Monthly price in currency.                                          | For month-based subscriptions.                                      |
| **price_yearly**|DECIMAL(10,2)|**Not Null**           | Yearly price in currency.                                           | For annual subscriptions (discounted?).                             |
| **currency**    | VARCHAR(3)  | **Default 'USD'**      | Currency code.                                                       | Multi-currency support.                                             |
| **billing_interval**|VARCHAR(20)|**Default 'month'**   | Standard cycle (month, year, etc.).                                 | Distinguishes billing frequency.                                    |
| **trial_days**  | INTEGER     | **Default 0**          | Free trial period length.                                           | Encourages try-before-you-buy.                                      |
| **api_calls_limit**|INTEGER   | **Nullable**           | Maximum API calls in period.                                        | Usage quotas.                                                       |
| **storage_limit_mb**|INTEGER  | **Nullable**           | Allotted storage in MB.                                             | For training data, model storage, etc.                              |
| **max_models**  | INTEGER     | **Nullable**           | Limit on custom/fine-tuned models.                                  | Differentiates advanced usage.                                      |
| **max_users**   | INTEGER     | **Nullable**           | Team size limit (for organizations).                                | Enterprise or advanced might allow bigger teams.                    |
| **features**    | TEXT        | **Not Null**           | JSON array describing plan-specific perks.                           | E.g. "fine_tuning": true, "priority_support": false.                |
| **is_active**   | BOOLEAN     | **Default True**        | Whether this plan is currently offered.                              | Hide or retire old plans gracefully.                                |
| **sort_order**  | INTEGER     | **Default 0**          | Display ordering in plan lists.                                     | Helps marketing order (Free -> Advanced -> Ultimate).               |
| **created_at**  | DATETIME    | **Default NOW()**       | When plan was created.                                              | Historical tracking.                                                |
| **updated_at**  | DATETIME    | **On Update NOW()**     | Last plan update.                                                   | Show “last changed date” or version.                                |

### **Technical Considerations**
- Minimal usage table, but do index `code` if used frequently by the system.  
- JSON-based features to handle flexible additions.

### **Example Usage Scenarios**
- **Showing Plan Options**: Fetch all `is_active = true`, order by `sort_order`.  
- **Plan Upgrades**: Compare new plan’s `api_calls_limit` or `max_models`.

---

## **9. Subscriptions Table**

### **Overview**
Tracks **active** or **past** subscriptions linking users or orgs to a chosen plan, plus status, start dates, etc.

### **Business Importance**
- **Billing & Enforcement**: Ties usage to an actual plan.  
- **Lifecycle**: Allows trial periods, cancellation flows, etc.  
- **Granular**: A user can have multiple subscriptions if allowed, or an org can share one.

### **Fields**

| Field Name       | Data Type    | Constraints          | Description                                                  | Business Importance                                      |
|------------------|-------------|----------------------|--------------------------------------------------------------|----------------------------------------------------------|
| **id**           | VARCHAR(36) | **PK**               | Unique subscription ID.                                     | Distinguishes each subscription contract.               |
| **user_id**      | VARCHAR(36) | **FK, Nullable**     | If subscription is individual.                               | Ties to user (if personal plan).                         |
| **organization_id**|VARCHAR(36)| **FK, Nullable**     | If subscription is org-level.                                | Ties to org.                                            |
| **plan_id**      | VARCHAR(36) | **FK**               | Which plan is subscribed to.                                 | E.g. "free," "advanced," etc.                            |
| **status**       | VARCHAR(20) | **Not Null**         | "active", "canceled", "past_due", etc.                       | Key state for billing logic.                             |
| **billing_info_id**|VARCHAR(36)| **FK**               | Payment method used.                                        | Links to user or org payment details.                   |
| **start_date**   | DATETIME    | **Not Null**         | When subscription began.                                    | Used for recurring charges.                              |
| **current_period_start**|DATETIME|**Not Null**         | Start of current billing cycle.                              | For usage resets or invoice creation.                    |
| **current_period_end**|DATETIME|**Not Null**          | End of current billing cycle.                                | If date passes and not canceled, auto-renew or grace.    |
| **cancel_at_period_end**|BOOLEAN|**Default False**     | If true, subscription ends at period end.                    | Standard subscription cancellation approach.             |
| **canceled_at**  | DATETIME    | **Nullable**         | Exact time subscription was canceled.                        | Auditing; used for partial refunds or proration.         |
| **trial_start**  | DATETIME    | **Nullable**         | Free trial start.                                           | Some plans have a trial.                                 |
| **trial_end**    | DATETIME    | **Nullable**         | Free trial end.                                             | If user cancels before this date, no charge.             |
| **quantity**     | INTEGER     | **Default 1**        | Seats or license count.                                     | For multi-user seat-based subscription.                  |
| **metadata**     | TEXT        | **Nullable**         | JSON with extra info.                                       | Could store custom data like usage notes, discount codes.|
| **created_at**   | DATETIME    | **Default NOW()**     | Insert timestamp.                                           | For referencing creation date.                           |
| **updated_at**   | DATETIME    | **On Update NOW()**   | Last updated.                                               | Tracks modifications to subscription.                    |

### **Technical Considerations**
- Possibly index `(user_id, status)` or `(organization_id, status)`.  
- *Potential expansions:* Automatic proration, partial refunds, seat changes.

### **Example Usage Scenarios**
- **Billing**: At `current_period_end`, system charges the user/org for next cycle.  
- **Cancel**: If `cancel_at_period_end = true`, the sub transitions to canceled at that date.  
- **Trials**: If `trial_end` is in the future, user might have limited features or no charge yet.

---

## **10. Billing History Table**

### **Overview**
Each row represents a **billing transaction** (payment, refund, partial credit, etc.).

### **Business Importance**
- **Financial Record**: Summaries of charges, compliance records.  
- **Customer Disputes**: Provide reference to `invoice_id` or `receipt_url`.  
- **Analytics**: Understand revenue, churn, user spend patterns.

### **Fields**

| Field Name           | Data Type    | Constraints          | Description                                                  | Business Importance                                         |
|----------------------|-------------|----------------------|--------------------------------------------------------------|-------------------------------------------------------------|
| **id**               | VARCHAR(36) | **PK**               | Unique transaction ID.                                      | Identifies each billing event.                              |
| **subscription_id**  | VARCHAR(36) | **FK**               | Link to `subscriptions.id`.                                 | Ties this charge to a specific subscription.               |
| **user_id**          | VARCHAR(36) | **FK, Nullable**     | For personal billing.                                       | If individual user.                                         |
| **organization_id**  | VARCHAR(36) | **FK, Nullable**     | For org-level billing.                                      | If enterprise invoice.                                      |
| **amount**           | DECIMAL(10,2)|**Not Null**         | Monetary amount charged.                                    | Key for financial calculations.                             |
| **currency**         | VARCHAR(3)  | **Default 'USD'**    | Currency code.                                              | Multi-currency support.                                     |
| **status**           | VARCHAR(20) | **Not Null**         | "succeeded", "failed", "refunded", etc.                     | Tells if payment was completed or reversed.                |
| **description**      | TEXT        | **Nullable**         | Payment or item details.                                    | Clarifies line items or reason for the charge.             |
| **invoice_id**       | VARCHAR(100)| **Nullable**         | ID from external payment processor.                         | Tracks third-party references (Stripe, PayPal, etc.).       |
| **receipt_url**      | VARCHAR(255)| **Nullable**         | Link to the actual invoice/receipt.                         | Customer can review charges.                                |
| **payment_method_id**| VARCHAR(36) | **FK**               | Reference to `billing_info.id`.                             | Payment method used.                                        |
| **payment_method_details**|TEXT    | **Nullable**         | JSON with method specifics.                                 | E.g. "Card brand: Visa, last4: 1234."                       |
| **billing_reason**   | VARCHAR(50) | **Nullable**         | "recurring", "one_time", "upgrade", etc.                    | Summarizes purpose of the transaction.                      |
| **period_start**     | DATETIME    | **Nullable**         | Start of service window.                                    | For usage-based or subscription-based charges.             |
| **period_end**       | DATETIME    | **Nullable**         | End of service window.                                      | Clarifies usage timeframe.                                  |
| **created_at**       | DATETIME    | **Default NOW()**     | Transaction time.                                           | For chronological queries.                                  |
| **metadata**         | TEXT        | **Nullable**         | JSON for custom data.                                       | Additional cross-reference or notes.                        |

### **Technical Considerations**
- Large table if system handles many micro-transactions. Partition or archive older records.  
- Index on `(subscription_id, created_at)` common for revenue reports.

### **Example Usage Scenarios**
- **Invoice Generation**: Summaries show up in user’s account or admin’s ledger.  
- **Refund**: Another row with negative or refunded amount.  
- **Year-End Summaries**: Query all successful charges for the year.

---

## **11. User Sessions Table**

### **Overview**
Manages **logged-in sessions**. When users log in, a record is created; can show device, IP, last activity.

### **Business Importance**
- **Security**: Force logouts, limit concurrency, track suspicious access.  
- **Enhanced UX**: Let users see active sessions, voluntarily end any.

### **Fields**

| Field Name    | Data Type    | Constraints         | Description                                            | Business Importance                                          |
|---------------|-------------|---------------------|--------------------------------------------------------|--------------------------------------------------------------|
| **id**        | VARCHAR(36) | **PK**              | UUID for the session.                                  | Unique session tracking.                                    |
| **user_id**   | VARCHAR(36) | **FK**              | Reference to `users.id`.                               | Identifies which user holds the session.                     |
| **token_hash**| VARCHAR(255)| **Not Null**        | Hash of session token.                                 | Store hashed tokens only for security.                       |
| **ip_address**| VARCHAR(45) |                     | IP address of the client.                              | Security/tracking.                                           |
| **user_agent**| VARCHAR(255)|                     | Browser or client info.                                | Distinguish device or environment.                           |
| **device_name**|VARCHAR(100)| **Nullable**        | Friendly name user sets for their device.              | Helps user identify sessions (e.g., "My iPhone X").           |
| **is_active** | BOOLEAN     | **Default True**    | If session is valid.                                   | Allows forced logout.                                        |
| **created_at**| DATETIME    | **Default NOW()**   | Time session began.                                    | For session age / idle checks.                               |
| **expires_at**| DATETIME    | **Not Null**        | Hard session expiry.                                   | Enforces time-limited sessions.                              |
| **last_activity**|DATETIME  |                     | Latest action time.                                    | Use for idle session timeouts.                               |
| **location_data**|TEXT      | **Nullable**        | JSON with geolocation details.                         | Possibly city/country from IP lookup.                        |

### **Technical Considerations**
- **Index**: `(user_id, is_active)` if frequently checking user’s active sessions.  
- May require high insertion and deletion rates if ephemeral tokens.

### **Example Usage Scenarios**
- **User Portal**: Show “You have 2 active sessions.”  
- **Admin**: Could forcibly set `is_active = false` to end suspicious sessions.  
- **Token Rotation**: `token_hash` updates if session is extended.

---

## **12. API Usage Table**

### **Overview**
Captures each **API call** made by a user or system key. Useful for billing, rate limiting, or usage analytics.

### **Business Importance**
- **Billing**: Charge by usage.  
- **Security & Anomaly Detection**: Spike detection in suspicious calls.  
- **Performance Monitoring**: track `response_time_ms`, status codes.

### **Fields**

| Field Name        | Data Type    | Constraints         | Description                                                 | Business Importance                                            |
|-------------------|-------------|---------------------|-------------------------------------------------------------|----------------------------------------------------------------|
| **id**            | VARCHAR(36) | **PK**              | UUID for usage record.                                     | Each call’s usage log.                                         |
| **user_id**       | VARCHAR(36) | **FK**              | Which user invoked the call.                               | For accountability, usage metrics.                             |
| **endpoint**      | VARCHAR(255)| **Not Null**        | e.g. "/v1/models/finetune".                                | Identifies what the user is accessing.                         |
| **status_code**   | SMALLINT    | **Not Null**        | HTTP status returned.                                      | Shows success/failure patterns.                                |
| **response_time_ms**|INTEGER    |                     | Duration in ms.                                            | Key metric for performance optimization.                       |
| **request_size_bytes**|INTEGER  |                     | Incoming payload size.                                     | Potential billing or analysis.                                 |
| **response_size_bytes**|INTEGER |                     | Outgoing payload size.                                     | Potential bandwidth cost tracking.                             |
| **timestamp**     | DATETIME    | **Default NOW()**   | When call happened.                                        | Time-based analytics & rate limiting.                          |
| **ip_address**    | VARCHAR(45) |                     | Source of request.                                         | Could correlate suspicious usage from known IP.                |
| **user_agent**    | VARCHAR(255)|                     | Browser or client signature.                               | Distinguish bots or scripts from normal usage.                 |

### **Technical Considerations**
1. **Partition by Month**: Large volume logs, improves query performance.  
2. **Index** `(timestamp)` or `(user_id, timestamp)` for usage queries, graphs.

### **Example Usage Scenarios**
- **Billing**: If plan has monthly call limits, sum user’s calls from `timestamp` in current period.  
- **Fraud Detection**: Identify unusual activity from new IP or repeated error codes.

---

## **13. Authentication Logs Table**

### **Overview**
Records attempts to **log in** or re-auth, capturing success/failure, IP, device, etc.

### **Business Importance**
- **Security**: Helps detect brute force, stolen credentials usage.  
- **Compliance**: Many standards require logging of auth events.  
- **Audit & Forensics**: Investigate unauthorized access or suspicious patterns.

### **Fields**

| Field Name      | Data Type    | Constraints            | Description                                                | Business Importance                                                    |
|-----------------|-------------|-------------------------|------------------------------------------------------------|------------------------------------------------------------------------|
| **id**          | VARCHAR(36) | **PK**                  | Unique event ID.                                           | Distinguishes each auth event.                                        |
| **user_id**     | VARCHAR(36) | **FK, Nullable**        | If known, link to `users.id`.                              | If unknown user (wrong email?), remains null.                          |
| **email**       | VARCHAR(255)|                         | Email used in attempt.                                     | Tracks even for failed attempts.                                      |
| **success**     | BOOLEAN     | **Not Null**            | If attempt was successful.                                 | Security indicator.                                                   |
| **failure_reason**|VARCHAR(100)|**Nullable**            | E.g. "wrong_password", "account_inactive".                 | Aids debugging and suspicious pattern analysis.                        |
| **ip_address**  | VARCHAR(45) |                         | IP for geolocation or blocking.                            | Could do IP-based rate limiting.                                      |
| **user_agent**  | VARCHAR(255)|                         | Browser or device used.                                    | Helps detect automated or unusual clients.                            |
| **timestamp**   | DATETIME    | **Default NOW()**       | When the event occurred.                                   | Key for time-based analytics.                                         |
| **device_id**   | VARCHAR(100)| **Nullable**            | Unique device fingerprint.                                 | Trusted device features or repeated attempts.                         |
| **location_data**|TEXT        | **Nullable**            | JSON with geodata (city, country).                        | Further forensic or compliance use.                                  |

### **Technical Considerations**
- Potentially large table, do partition or archive.  
- Index `(timestamp, success)` for daily or monthly summaries.

### **Example Usage Scenarios**
- **Lockouts**: After X consecutive fails for the same email, block new attempts.  
- **Security Alerts**: Email the user if a successful login from unfamiliar location.  
- **Regulatory**: Show all login events to an auditor.

---

## **14. Security Audit Table**

### **Overview**
Captures **security-relevant** actions like password changes, roles updates, or advanced config modifications.

### **Business Importance**
- **Compliance**: Many standards require a record of security-critical changes.  
- **Investigations**: Show who changed what, when, from which IP.

### **Fields**

| Field Name       | Data Type    | Constraints          | Description                                                | Business Importance                                                       |
|------------------|-------------|----------------------|------------------------------------------------------------|---------------------------------------------------------------------------|
| **id**           | VARCHAR(36) | **PK**               | Unique security event ID.                                  | Differentiates each action.                                              |
| **user_id**      | VARCHAR(36) | **FK**               | Who performed the action.                                  | Accountability for security changes.                                      |
| **action_type**  | VARCHAR(50) | **Not Null**         | E.g. "role_change", "password_reset".                      | Quick categorization.                                                     |
| **target_user_id**|VARCHAR(36) | **FK, Nullable**     | If action affected another user.                           | E.g. Admin changed someone else’s password.                               |
| **description**  | TEXT        | **Not Null**         | Explanation of the action.                                 | Detailed context for compliance.                                          |
| **old_value**    | TEXT        | **Nullable**         | Value before change.                                       | Helps revert or see differences.                                          |
| **new_value**    | TEXT        | **Nullable**         | Value after change.                                        | Documentation of final state.                                             |
| **ip_address**   | VARCHAR(45) |                      | Source IP.                                                 | Security location context.                                                |
| **timestamp**    | DATETIME    | **Default NOW()**     | When this action occurred.                                 | Timelines & chain-of-custody.                                             |
| **metadata**     | TEXT        | **Nullable**         | JSON for extra info.                                       | Possibly store approval references or further details.                    |

### **Technical Considerations**
- Potential large volume if system logs many config changes.  
- Index `(action_type, timestamp)` for quick admin reviews.

### **Example Usage Scenarios**
- **Password Reset**: `old_value` might have partial hashed password? Typically store partial info only.  
- **Change of Subscription Tier**: Admin sets user from “free” to “enterprise.”  
- **Deletion of Data**: Record the request or process for compliance.

---

## **15. Notification Preferences Table**

### **Overview**
Houses user-level settings for how/when they want to be notified about different platform events.

### **Business Importance**
- **User Satisfaction**: Avoid spamming; respect preferences.  
- **Regulatory**: Some regions require easy unsubscribes from certain notifications.  
- **Advanced Feature**: Different channels (email, SMS, push).

### **Fields**

| Field Name         | Data Type    | Constraints         | Description                                                    | Business Importance                                                   |
|--------------------|-------------|---------------------|----------------------------------------------------------------|------------------------------------------------------------------------|
| **id**             | VARCHAR(36) | **PK**              | UUID for the preference record.                                | Tracks each user’s distinct notification config.                      |
| **user_id**        | VARCHAR(36) | **FK**              | Points to `users.id`.                                          | Identifies which user these prefs belong to.                          |
| **notification_type**|VARCHAR(50)| **Not Null**        | Category e.g. "billing", "security", "general".                | Each type might have different channels/frequency.                    |
| **channel**        | VARCHAR(20) | **Not Null**        | "email", "push", "sms", etc.                                   | The method of delivering notices.                                      |
| **enabled**        | BOOLEAN     | **Default True**    | Whether user wants this type of notice.                        | Quickly enable/disable notifications.                                 |
| **frequency**      | VARCHAR(20) | **Default 'immediate'** | "immediate", "daily_digest", etc.                           | Different schedules for notifications.                                |
| **quiet_hours_start**|TIME       | **Nullable**        | Start of do-not-disturb.                                       | User may not want 2am messages.                                       |
| **quiet_hours_end**|TIME        | **Nullable**        | End of do-not-disturb.                                         | Respects user’s daily schedule.                                       |
| **created_at**     | DATETIME    | **Default NOW()**   | When preference was created.                                   | Helps revert or see when user changed settings.                       |
| **updated_at**     | DATETIME    | **On Update NOW()** | Last updated.                                                 | Keep track of changes over time.                                      |

### **Technical Considerations**
- Potentially many rows per user, if multiple channels or event types.  
- Index `(user_id, notification_type)` for quick lookups.

### **Example Usage Scenarios**
- **Immediate Security Alerts**: "enabled = true," "frequency = immediate," "channel = email."  
- **Digest**: Some users prefer daily or weekly emails rather than immediate.

---

## **16. API Keys Table**

### **Overview**
Supports machine-to-machine or programmatic access with **token-based** credentials.

### **Business Importance**
- **Integrations**: 3rd party scripts or internal microservices can call the platform.  
- **Security**: Access is restricted by key scopes, can be revoked easily.

### **Fields**

| Field Name        | Data Type    | Constraints              | Description                                                 | Business Importance                                                   |
|-------------------|-------------|--------------------------|-------------------------------------------------------------|-----------------------------------------------------------------------|
| **id**            | VARCHAR(36) | **PK**                   | UUID for the API key record.                               | Unique identification per key.                                        |
| **user_id**       | VARCHAR(36) | **FK, Nullable**         | If personal key.                                           | Ties usage or billing to a specific user.                             |
| **organization_id**|VARCHAR(36) | **FK, Nullable**         | If org-level key.                                          | Shared by entire organization.                                       |
| **name**          | VARCHAR(255)| **Not Null**             | Descriptive name, e.g. "DevOps pipeline key."               | Helps user remember which key is which.                               |
| **key_prefix**    | VARCHAR(10) | **Unique**               | First few chars of the real key.                           | Identify a key without exposing the full secret.                      |
| **key_hash**      | VARCHAR(255)| **Not Null**             | Hashed version of the actual key.                          | Don’t store secrets in plaintext.                                     |
| **scopes**        | TEXT        | **Not Null**             | JSON listing permissions or usage scope.                   | E.g. "read:models", "fine_tune:all".                                  |
| **expires_at**    | DATETIME    | **Nullable**             | Key expiry date.                                           | Encourage key rotation, improved security.                            |
| **last_used_at**  | DATETIME    | **Nullable**             | When key was last used.                                    | Identify stale keys for revocation.                                   |
| **created_at**    | DATETIME    | **Default NOW()**        | Key creation time.                                         | Auditing.                                                             |
| **created_by**    | VARCHAR(36) | **FK**                   | Who created the key.                                       | Accountability (an admin or the user?).                               |
| **is_active**     | BOOLEAN     | **Default True**         | If the key is still valid.                                 | Quick revocation mechanism.                                           |
| **ip_whitelist**  | TEXT        | **Nullable**             | JSON array of IPs allowed.                                 | Lock down usage to certain network addresses.                         |
| **rate_limit**    | INTEGER     | **Nullable**             | Additional usage limit just for this key.                  | Fine-tuned throttle beyond plan’s default.                            |

### **Technical Considerations**
- Keep key secret out of DB by storing only hashed.  
- Possibly index `key_prefix` for partial lookup.  
- Large number of keys for big orgs with multiple integrations.

### **Example Usage Scenarios**
- **CI/CD**: Pipeline using an API key for automated model updates.  
- **Revocation**: Admin sets `is_active = false` or sets `expires_at`.  
- **Usage**: Also track usage in `api_usage`, referencing `user_id` or `organization_id`.

---

## **17. Model Access Permissions Table**

### **Overview**
Specifies whether a user or org can read/fine-tune/deploy certain models. Helps handle advanced licensing or pay-per-model usage.

### **Business Importance**
- **Granular Access**: Some models might be restricted to certain paying tiers or compliance rules.  
- **Usage Caps**: Limit tokens or concurrency for certain users.

### **Fields**

| Field Name          | Data Type    | Constraints          | Description                                                    | Business Importance                                                           |
|---------------------|-------------|----------------------|----------------------------------------------------------------|-------------------------------------------------------------------------------|
| **id**              | VARCHAR(36) | **PK**               | UUID for permission record.                                   | Identifies each model-specific permission.                                   |
| **user_id**         | VARCHAR(36) | **FK, Nullable**     | If this permission is for a specific user.                     | Might be null if it's org-based.                                             |
| **organization_id** | VARCHAR(36) | **FK, Nullable**     | If this permission is for an entire org.                       | Allows org-wide licensing of a model.                                        |
| **model_id**        | VARCHAR(255)| **Not Null**         | Unique ID for the model (e.g., "falcon-7b").                   | Reference the model in the catalog.                                          |
| **permission_type** | VARCHAR(50) | **Not Null**         | e.g. "read", "fine-tune", "deploy".                            | Defines usage scope.                                                         |
| **max_tokens**      | INTEGER     | **Nullable**         | Token limit for usage.                                         | Might throttle how many tokens can be processed daily.                       |
| **priority_access** | BOOLEAN     | **Default False**     | If user/org can skip queues.                                   | Premium service for enterprise.                                             |
| **cost_multiplier** | DECIMAL(10,4)|**Default 1.0**       | Extra cost factor for usage.                                   | e.g. 1.2 for high-end model usage billing.                                  |
| **granted_at**      | DATETIME    | **Default NOW()**     | When permission was granted.                                   | Lifecycle tracking.                                                          |
| **granted_by**      | VARCHAR(36) | **FK, Nullable**     | The user who granted permission.                               | Accountability.                                                              |
| **expires_at**      | DATETIME    | **Nullable**         | If this permission eventually ends.                            | Temp access or limited trial.                                                |
| **is_active**       | BOOLEAN     | **Default True**      | If permission is currently valid.                              | Quick suspension.                                                            |
| **usage_conditions**| TEXT        | **Nullable**         | JSON with special rules.                                       | e.g. "Must not use for competitor projects."                                 |

### **Technical Considerations**
- Potential partial overlap with subscription but more fine-grained (per-model).  
- Index `(model_id, user_id)` or `(model_id, organization_id)` for quick lookups.

### **Example Usage Scenarios**
- **Exclusive Model**: Some org pays extra for a new unreleased model with cost multiplier.  
- **Trial Model**: Expires after 14 days if `expires_at` is set.  
- **Limited Token**: Use `max_tokens` to forcibly cap usage beyond plan defaults.

---

## **18. Billing Info Table**

### **Overview**
Stores payment data (card, bank) for users or orgs. Distinguishes personal from corporate billing.

### **Business Importance**
- **Recurring Subscriptions**: Auto-charges monthly or annually.  
- **Compliance**: Holds address, tax IDs for correct invoicing.  
- **Flexibility**: Users/Orgs can have multiple payment methods.

### **Fields**

| Field Name               | Data Type    | Constraints            | Description                                                       | Business Importance                                                 |
|--------------------------|-------------|------------------------|-------------------------------------------------------------------|---------------------------------------------------------------------|
| **id**                   | VARCHAR(36) | **PK**                 | Unique billing record ID.                                         | Distinguishes each payment method entry.                            |
| **user_id**              | VARCHAR(36) | **FK, Nullable**       | Ties to user if personal.                                         | Could be null if org-based.                                         |
| **organization_id**      | VARCHAR(36) | **FK, Nullable**       | Ties to org if corporate payment.                                 | Distinguishes personal from enterprise.                             |
| **payment_provider**     | VARCHAR(50) | **Not Null**           | E.g. "Stripe", "PayPal".                                          | Platform can support multiple gateways.                             |
| **payment_method_id**    | VARCHAR(255)| **Nullable**           | ID in payment provider system.                                    | For referencing a card or token externally.                         |
| **payment_method_type**  | VARCHAR(50) | **Nullable**           | "credit_card", "bank_transfer", etc.                              | Clarifies how charges are done.                                     |
| **last_four**            | VARCHAR(4)  | **Nullable**           | For credit cards.                                                 | Minimally identify card to user.                                    |
| **expiry_month**         | SMALLINT    | **Nullable**           | Card expiration month.                                            | Warnings for soon expiring.                                         |
| **expiry_year**          | SMALLINT    | **Nullable**           | Card expiration year.                                             | Warnings for soon expiring.                                         |
| **cardholder_name**      | VARCHAR(255)| **Nullable**           | Name on the card.                                                 | Payment verification.                                               |
| **billing_address_line1**|VARCHAR(255)| **Nullable**           | Street address.                                                   | Potential tax/country logic.                                        |
| **billing_address_line2**|VARCHAR(255)| **Nullable**           | Additional address info.                                          | Extra line for suite, apt #.                                        |
| **billing_city**         | VARCHAR(255)| **Nullable**           | City.                                                             | Support location-based compliance.                                  |
| **billing_state**        | VARCHAR(255)| **Nullable**           | State or province.                                                | Tax regions vary.                                                   |
| **billing_postal_code**  | VARCHAR(20) | **Nullable**           | ZIP/postal code.                                                  | Often required by payment processors.                               |
| **billing_country**      | VARCHAR(2)  | **Nullable**           | ISO country code.                                                 | Currency and tax rates differ by country.                           |
| **tax_id**               | VARCHAR(100)| **Nullable**           | VAT or business tax ID.                                           | For B2B or enterprise compliance.                                   |
| **is_default**           | BOOLEAN     | **Default True**        | If this is the main method.                                       | Users/Orgs can store multiple methods but one primary.              |
| **created_at**           | DATETIME    | **Default NOW()**       | When record was created.                                          | History tracking.                                                   |
| **updated_at**           | DATETIME    | **On Update NOW()**     | Last updated.                                                     | Noting card or address changes.                                     |

### **Technical Considerations**
- Possibly store data encrypted at rest.  
- Typically store only partial card info (like last 4) for compliance (PCI-DSS).

### **Example Usage Scenarios**
- **Recurring Billing**: `is_default = true` means auto-charge monthly using that method.  
- **Org Payment**: `organization_id` is set, so entire team’s usage is billed.  
- **Address Update**: E.g. if user moves country, changes tax rates.

---

## **19. Implementation Considerations**

1. **Core Tables First**: Implement `users` & `user_profiles` for robust authentication and profile expansions.  
2. **Incremental Approach**: Add advanced features (RBAC, subscription, billing, logs) as the platform grows, ensuring each piece is tested thoroughly.  
3. **Partition by Date**: Large-volume logs (e.g., `api_usage`, `auth_logs`, `security_audit`) can be partitioned monthly or quarterly to maintain performance.  
4. **Regular Archiving**: Old log data can be archived to cheaper storage to keep queries fast while meeting compliance retention.  
5. **Encryption**: For sensitive columns (`hashed_password`, security questions, payment info), implement column-level or field-level encryption, possibly integrated with secrets manager (e.g., HashiCorp Vault).  
6. **Indexes**: Beyond primary keys, add indexes for common queries (`email`, `(organization_id, user_id)`, `(timestamp, success)`). For JSON fields (preferences, features), consider MySQL JSON indexes.  
7. **Foreign Key Constraints**: Use `ON DELETE CASCADE` or `SET NULL` where appropriate. Keep relational integrity consistent.  
8. **Monitoring**: Implement DB monitoring to watch for table growth, row locking, query performance. Helps scale or optimize proactively.  
9. **Core Separation**: Distinguishing `users` from `user_profiles` improves **authentication clarity** while enabling enterprise expansions and flexible user data.  
10. **Extended Logging & Compliance**: `api_usage`, `auth_logs`, `security_audit`, etc. are **vital** for HIPAA/GDPR compliance, security forensics, and anomaly detection.  
11. **Enterprise Team Structures**: The `organizations` + `team_members` approach supports hierarchical usage, allows a single user to belong to multiple orgs.  
12. **RBAC**: Fine-grained control with **roles**, **permissions**, **role_permissions**, **user_roles** allows complex security and grouping, especially for enterprise scenarios.  
13. **Subscription & Billing**: `subscription_plans`, `subscriptions`, `billing_history` handle the monetization. Additional usage-based costs for HPC usage (training, inference) can integrate with `api_usage` or custom cost multipliers.  
14. **Performance & Key Columns**: Using `VARCHAR(36)` for UUID in MySQL is typical, but keep an eye on indexing overhead. If heavy searching on JSON columns, explore MySQL’s JSON indexes or partial indexes.  
15. **Additional Logging Tables**: For advanced security or analytics, logs can become huge. Partition or regularly archive to preserve performance.  
16. **Compliance & Security**: Column-level encryption for `payment_method_id`, `security_questions`, or partial credit card data is strongly recommended. Manage secrets with a tool like **HashiCorp Vault** to meet PCI-DSS or HIPAA needs.

---

## **20. Relationships & Diagram**

Below is a **high-level** depiction of how these tables interact:

Potentially a star schema diagram or most likely an ERD (entity relationship diagram):


### **Relationship Highlights**
1. **Users - User_Profiles**: 1-to-1 extension of user info.  
2. **Users - User_Roles** - Roles**: Many-to-many via user_roles.  
3. **Organizations - Team_Members** - Users**: Many-to-many with extra fields for org role.  
4. **Subscription_Plans - Subscriptions**: 1-to-many, a plan can have many active subs.  
5. **Subscriptions - Billing_History**: 1-to-many, each sub can have multiple transactions.  
6. **user_id** or **organization_id** used in many places (API_Keys, Model_Access_Permissions, etc.) to handle personal vs. enterprise usage.

---

# **Conclusion**

This **structured data model** for the Artintel LM platform addresses:
- **Core authentication** (Users & Profiles).
- **Enterprise readiness** (Organizations, Team Members).
- **RBAC** for fine-grained security (Roles, Permissions, etc.).
- **Monetization** with subscription management & billing history.
- **Advanced logging** (API usage, auth logs, security audit).
- **Scalability & compliance** via partitioning, archiving, encryption, indexing, and monitoring strategies.

As the platform **scales**, iterative refinements will integrate advanced HPC cost tracking, further logging expansions, and new compliance or enterprise demands. The separation of concerns, robust foreign key relationships, and flexible JSON columns ensure the system can adapt quickly to evolving AI model usage and business rules.

> **Next Steps**:  
> 1. **Implemented** core tables (`users`, `user_profiles`) for immediate authentication and identity.  
> 2. **Gradually** add the additional tables (organizations, RBAC, subscriptions, logs) as needed.  
> 3. **Partition & Archive** large log tables (api_usage, auth_logs) to maintain performance.  
> 4. **Encrypt** sensitive columns, especially for production compliance.  
> 5. **Monitor** DB usage to optimize queries and indexes proactively.

This document will undergo regular updates as new requirements arise or usage patterns reveal performance/feature gaps. By following the best practices outlined, Artintel LM can maintain a secure, scalable, and feature-rich data backbone for AI model orchestration.
