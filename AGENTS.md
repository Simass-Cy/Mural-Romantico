# AGENTS.md — Mural Romântico API

This file defines the working rules for AI coding agents contributing to this repository.

The project is a Spring Boot backend API for a romantic mural SaaS MVP. The MVP must be delivered quickly, but the architecture must be safe enough for real users, real Pix payments, private photos, and future recurrence support.

Agents must read this file before proposing or applying changes.

---

## 1. Product Context

This repository contains the backend API for a romantic mural web application.

The product allows a customer to create and purchase a private romantic digital mural for a couple. The mural includes memories, uploaded photos, letters, future letters, songs, movies, and a basic calendar.

The frontend already exists as HTML/CSS/JavaScript prototypes and will consume this API through REST endpoints.

### MVP Business Model

The MVP uses:

- one-time purchase of a mural;
- real Pix payment through Mercado Pago;
- automatic payment confirmation through webhook;
- access unlocked after payment approval;
- image upload through Amazon S3;
- architecture prepared for recurrence in the future;
- recurrence disabled in the MVP.

### Important Product Decision

Do not implement active recurring billing in the MVP.

The MVP must support the data model for future recurrence, but the current user-facing flow is:

```text
User creates account
User creates draft mural
User pays one-time Pix checkout
Mercado Pago confirms payment through webhook
Backend activates customer entitlement
Mural becomes available
User uploads photos and uses modules
```

---

## 2. Repository Structure

Project root:

```text
mural-romantico-api/
```

Main source paths:

```text
src/main/java/com/velvetmural/api
src/main/resources
src/main/resources/db/migration
src/test/java/com/velvetmural/api
docs/
```

The project uses:

- Java 17;
- Spring Boot;
- Maven;
- PostgreSQL;
- Flyway;
- Spring Security;
- JPA/Hibernate;
- Bean Validation;
- Amazon S3;
- Mercado Pago Pix.

---

## 3. Module and Package Organization

Use domain-oriented packages under:

```text
com.velvetmural.api
```

Recommended package structure:

```text
com.velvetmural.api
├── config
├── security
├── common
├── auth
├── user
├── account
├── mural
├── member
├── billing
├── payment
├── media
├── memory
├── letter
├── futureletter
├── song
├── movie
├── calendar
└── dashboard
```

### Package Responsibilities

#### `config`

General application configuration:

- CORS;
- Jackson;
- S3 client;
- Mercado Pago client/config;
- application properties;
- environment-based configuration.

#### `security`

Authentication and authorization:

- JWT;
- security filters;
- password hashing;
- current user provider;
- access checks;
- mural membership validation;
- entitlement validation.

#### `common`

Shared infrastructure:

- base exceptions;
- global exception handler;
- API error response model;
- pagination helpers;
- audit utilities;
- shared enums;
- validation helpers.

#### `auth`

Authentication flows:

- register;
- login;
- refresh/me endpoint if needed;
- password hashing integration.

#### `user`

User domain:

- user entity;
- user repository;
- user profile basics.

#### `account`

SaaS account domain:

- account entity;
- owner relationship;
- account status.

#### `mural`

Mural domain:

- draft mural creation;
- mural settings;
- slug;
- couple names;
- relationship start date;
- mural status.

#### `member`

Mural membership:

- owner;
- partner;
- viewer if needed later;
- invite token;
- invite acceptance.

#### `billing`

Commercial model:

- offers;
- checkout orders;
- customer entitlements;
- billing agreements for future recurrence.

#### `payment`

Payment provider integration:

- Mercado Pago Pix checkout;
- Pix payment transaction;
- webhook receiving;
- webhook idempotency;
- provider validation before activation.

#### `media`

S3 media upload:

- presigned upload URL;
- complete upload;
- media metadata;
- private object handling;
- media limits.

#### `memory`

Album of memories:

- memory CRUD;
- favorite/special flags;
- category/year filtering;
- optional linked photo.

#### `letter`

Regular letters:

- sent/received letters;
- read status;
- body;
- occasion;
- signature.

#### `futureletter`

Future letters:

- locked until open date;
- body must not be returned before open date;
- open/status logic.

#### `song`

Couple songs:

- title;
- artist;
- external link;
- memory text;
- favorite flag.

#### `movie`

Couple movies:

- title;
- genre;
- status;
- rating;
- note;
- favorite flag.

#### `calendar`

Basic romantic calendar:

- events;
- recurring yearly dates;
- event type;
- reminders later if needed.

#### `dashboard`

Home summary:

- couple data;
- days together;
- counts;
- latest memories;
- upcoming events;
- recent letters.

---

## 4. Build, Test, and Development Commands

Run commands from the project root, where `pom.xml` and `mvnw.cmd` exist.

### Windows PowerShell

```powershell
.\mvnw.cmd spring-boot:run
.\mvnw.cmd test
.\mvnw.cmd clean package
```

### Flyway

If Flyway is configured for the database:

```powershell
.\mvnw.cmd flyway:info
```

### Requirements

Use Java 17.

Do not assume Docker is available unless the user explicitly confirms it.

---

## 5. MVP Scope

### In Scope for the MVP

The MVP includes:

- authentication;
- user registration;
- login;
- account creation;
- draft mural creation;
- one-time mural purchase;
- Mercado Pago Pix checkout;
- automatic payment confirmation through webhook;
- customer entitlement activation;
- invite partner flow;
- dashboard/home summary;
- S3 photo upload;
- album of memories;
- regular letters;
- future letters;
- songs;
- movies;
- basic calendar;
- PostgreSQL persistence;
- Flyway migrations;
- security by mural membership;
- security by active entitlement.

### Out of Scope for the MVP

Do not implement unless explicitly approved:

- active recurring billing;
- Pix Automático;
- credit card;
- boleto;
- refunds;
- advanced admin panel;
- public marketplace;
- advanced analytics;
- email notifications;
- WhatsApp notifications;
- garden module;
- bouquet module;
- recipes module;
- destinations module;
- bookshelf module;
- wishlist module;
- dreams/objectives module;
- advanced media editing;
- CloudFront signed URLs;
- mobile app;
- microservices.

---

## 6. Billing and Access Model

The billing model must support one-time purchase now and recurrence later.

### Required Billing Tables

Use these concepts:

```text
offers
checkout_orders
payment_transactions
customer_entitlements
billing_agreements
payment_webhook_events
```

### `offers`

Represents what is sold.

The MVP offer is a one-time purchase:

```text
billing_type = ONE_TIME
recurrence_enabled = false
```

Future offers may be recurring:

```text
billing_type = RECURRING
recurrence_enabled = true
```

Do not expose recurring offers to users in the MVP.

### `checkout_orders`

Represents a customer purchase attempt.

Expected statuses:

```text
PENDING
PAID
EXPIRED
CANCELED
FAILED
```

### `payment_transactions`

Represents the provider payment.

For the MVP:

```text
provider = MERCADO_PAGO
payment_method = PIX
```

Expected statuses:

```text
PENDING
APPROVED
REJECTED
EXPIRED
CANCELED
FAILED
```

### `customer_entitlements`

This is the central access control abstraction.

The MVP unlocks access through:

```text
entitlement_type = LIFETIME_ACCESS
status = ACTIVE
ends_at = null
```

Access to mural modules must depend on an active customer entitlement, not on a subscription.

Correct access rule:

```text
authenticated user
+ user is mural member
+ mural has active customer_entitlement
= access allowed
```

### `billing_agreements`

This exists for future recurrence support.

In the MVP:

```text
recurrence_enabled = false
status = INACTIVE
```

Do not implement active recurring charges now.

---

## 7. Payment Rules — Mercado Pago Pix

The MVP must use real Pix payment through Mercado Pago.

### Required Flow

```text
User chooses offer
Backend creates checkout_order
Backend creates Mercado Pago Pix payment
Backend stores payment_transaction
Frontend shows QR Code / Pix Copia e Cola
User pays Pix
Mercado Pago sends webhook
Backend receives webhook
Backend stores webhook event
Backend validates/consults provider payment
Backend marks transaction APPROVED
Backend marks checkout_order PAID
Backend creates customer_entitlement ACTIVE
Mural becomes available
```

### Critical Security Rule

Never activate a mural by trusting only the webhook payload.

Webhook handling must:

1. receive the webhook;
2. store the raw event safely;
3. identify the provider payment id;
4. validate authenticity when supported;
5. query Mercado Pago or validate the payment with provider data;
6. confirm amount, status, provider payment id, and internal reference;
7. only then create or activate customer entitlement.

### Webhook Idempotency

Webhooks may be delivered more than once.

Implement webhook idempotency using `payment_webhook_events` or equivalent.

If the same event/payment is received twice:

- do not create duplicate entitlements;
- do not double-update orders incorrectly;
- return a safe success response if the event was already processed.

### Payment Logging

Do not log:

- full raw sensitive payloads in normal application logs;
- access tokens;
- customer documents;
- private user data.

It is acceptable to store the webhook payload in the database for audit if it does not expose secrets and is protected as internal data.

---

## 8. S3 Media Upload Rules

The MVP uses Amazon S3 for photo upload.

### Required Upload Strategy

Use S3 presigned URLs.

Do not send the entire file through the backend unless explicitly approved.

Flow:

```text
Frontend requests presigned URL
Backend validates user, mural, entitlement, file type, size, limits
Backend creates media_file with PENDING status
Backend returns presigned upload URL
Frontend uploads directly to S3
Frontend calls complete endpoint
Backend checks object if possible
Backend marks media_file AVAILABLE
Frontend displays the photo
```

### S3 Rules

- bucket must be private;
- do not expose AWS credentials to frontend;
- do not commit AWS credentials;
- object keys must not contain intimate user data;
- use generated UUID-based object keys;
- recommended object key pattern:

```text
murals/{muralId}/media/{uuid}.{extension}
```

### Media Table

`media_files` should store:

```text
id
mural_id
uploaded_by
original_name
storage_provider
bucket_name
object_key
content_type
size_bytes
status
created_at
uploaded_at
deleted_at
```

Expected statuses:

```text
PENDING
AVAILABLE
FAILED
DELETED
```

### MVP Media Limits

Enforce these limits unless the user changes them:

```text
50 photos per mural
5 MB per photo
250 MB total per mural
```

Accepted content types:

```text
image/jpeg
image/png
image/webp
```

Reject all other file types.

---

## 9. Multi-Tenant Security Rules

This is a SaaS. Data isolation is mandatory.

### Mandatory Rule

Never fetch tenant content only by its object id.

Wrong:

```text
findById(memoryId)
```

Correct:

```text
findByIdAndMuralId(memoryId, muralId)
```

Or equivalent query scoped by mural/account.

### Every Mural Content Entity Must Have `mural_id`

This applies to:

- memories;
- media files;
- letters;
- future letters;
- songs;
- movies;
- calendar events;
- dashboard-derived data;
- any future module content.

### Required Access Checks

Before reading or mutating mural content, validate:

```text
user is authenticated
user belongs to mural
target entity belongs to mural
mural has active entitlement when required
operation is allowed for user's mural role
```

### Roles

Minimum mural roles:

```text
OWNER
PARTNER
```

Optional later:

```text
VIEWER
```

OWNER can manage mural settings and invite partner.

PARTNER can use shared modules after accepting invite.

---

## 10. Future Letter Security Rule

Future letters have sensitive locking logic.

Before `open_date`, the API must not return the letter body.

The frontend hiding the body is not enough.

Correct behavior before open date:

```text
return metadata only
return locked status
return open_date
return days remaining if useful
do not return body
```

Correct behavior on/after open date:

```text
return full content if user has access
```

---

## 11. Data Privacy and Logging Rules

This product stores intimate content.

Do not log:

- letter bodies;
- memory descriptions;
- private notes;
- photo URLs if they are signed/private;
- Pix copy-and-paste codes in normal logs;
- webhook secrets;
- JWTs;
- passwords;
- AWS credentials;
- Mercado Pago tokens.

Use technical identifiers in logs:

```text
userId
accountId
muralId
orderId
transactionId
eventId
```

Avoid logging actual romantic/private content.

---

## 12. Critical Version Control Rules

Never commit secrets or sensitive files.

### Must Not Be Committed

Do not commit:

```text
.env
.env.*
*.env
application-prod.yml with real secrets
application-production.yml with real secrets
application.properties containing real secrets
application.yml containing real secrets
secrets.yml
credentials.yml
*.pem
*.key
*.p12
*.jks
*.keystore
id_rsa
id_ed25519
aws_credentials
Mercado Pago access tokens
AWS access keys
JWT secrets
database passwords
signed S3 URLs
private webhook payload dumps
real customer data
real intimate letters/memories/photos
```

### Required Practice

Use environment variables for secrets.

Examples of allowed placeholders:

```text
${DB_URL}
${DB_USER}
${DB_PASSWORD}
${JWT_SECRET}
${MERCADO_PAGO_ACCESS_TOKEN}
${MERCADO_PAGO_WEBHOOK_SECRET}
${AWS_ACCESS_KEY_ID}
${AWS_SECRET_ACCESS_KEY}
${AWS_REGION}
${AWS_S3_BUCKET}
```

### `.gitignore`

The repository must ignore at least:

```text
.env
.env.*
*.env
target/
.idea/
.vscode/
*.iml
*.log
logs/
uploads/
local-storage/
*.pem
*.key
*.p12
*.jks
*.keystore
```

Before committing, check staged files.

Recommended commands:

```powershell
git status
git diff --cached
```

If any secret or private data is staged, stop and remove it before continuing.

---

## 13. Database and Flyway Rules

Use Flyway for schema changes.

Migration files must be placed in:

```text
src/main/resources/db/migration
```

Use names like:

```text
V1__create_users.sql
V2__create_accounts_and_murals.sql
V3__create_billing_tables.sql
V4__create_payment_tables.sql
V5__create_media_tables.sql
V6__create_memory_tables.sql
```

### Migration Rules

- Do not edit an already-applied migration in a shared environment.
- Create a new migration for schema changes.
- Use UUID primary keys unless a strong reason is given.
- Add indexes for foreign keys and frequent lookup columns.
- Add unique constraints where business rules require them.
- All tenant content tables must include `mural_id`.
- Prefer explicit constraints and not-null rules for critical fields.

### Timestamp Rules

Use:

```text
Instant
```

for audit timestamps such as:

```text
createdAt
updatedAt
paidAt
approvedAt
deletedAt
```

Use:

```text
LocalDate
```

for domain dates such as:

```text
relationshipStartDate
memoryDate
openDate
eventDate
```

---

## 14. Coding Style

Use standard Spring Boot conventions.

### Java Style

- 4-space indentation;
- constructor injection;
- no field injection;
- DTOs at API boundaries;
- services for business rules;
- repositories only for persistence;
- controllers must remain thin;
- use Bean Validation for request DTOs;
- use meaningful exception types;
- use a global exception handler.

### IDs

Use UUIDs for public IDs and entity IDs unless explicitly approved otherwise.

### DTO Naming

Use clear request/response names:

```text
CreateMuralRequest
MuralResponse
CreateMemoryRequest
MemoryResponse
CreateCheckoutOrderRequest
CheckoutOrderResponse
PresignMediaUploadRequest
PresignMediaUploadResponse
```

### Entity Naming

Use singular entity names:

```text
User
Account
Mural
MuralMember
Offer
CheckoutOrder
PaymentTransaction
CustomerEntitlement
BillingAgreement
PaymentWebhookEvent
MediaFile
Memory
Letter
FutureLetter
Song
Movie
CalendarEvent
```

### Package Naming

Prefer singular package names:

```text
user
account
mural
member
billing
payment
media
memory
letter
futureletter
song
movie
calendar
dashboard
```

---

## 15. API Design Rules

Use REST-style endpoints.

Recommended base paths:

```text
/api/auth
/api/me
/api/murals
/api/offers
/api/checkout
/api/webhooks/mercado-pago
/api/murals/{muralId}/media
/api/murals/{muralId}/memories
/api/murals/{muralId}/letters
/api/murals/{muralId}/future-letters
/api/murals/{muralId}/songs
/api/murals/{muralId}/movies
/api/murals/{muralId}/calendar-events
/api/murals/{muralId}/dashboard
```

### Required Endpoint Groups

#### Auth

```text
POST /api/auth/register
POST /api/auth/login
GET  /api/me
```

#### Mural

```text
POST /api/murals/draft
GET  /api/murals
GET  /api/murals/{muralId}
PUT  /api/murals/{muralId}/settings
```

#### Invite

```text
POST /api/murals/{muralId}/invites
POST /api/mural-invites/{token}/accept
```

#### Offers and Checkout

```text
GET  /api/offers
POST /api/checkout/orders
GET  /api/checkout/orders/{orderId}
```

#### Mercado Pago Webhook

```text
POST /api/webhooks/mercado-pago
```

#### Media

```text
POST   /api/murals/{muralId}/media/presign
POST   /api/murals/{muralId}/media/{mediaId}/complete
GET    /api/murals/{muralId}/media
DELETE /api/murals/{muralId}/media/{mediaId}
```

#### Memories

```text
GET    /api/murals/{muralId}/memories
POST   /api/murals/{muralId}/memories
PUT    /api/murals/{muralId}/memories/{memoryId}
DELETE /api/murals/{muralId}/memories/{memoryId}
PATCH  /api/murals/{muralId}/memories/{memoryId}/favorite
```

#### Letters

```text
GET    /api/murals/{muralId}/letters
POST   /api/murals/{muralId}/letters
GET    /api/murals/{muralId}/letters/{letterId}
PATCH  /api/murals/{muralId}/letters/{letterId}/read
DELETE /api/murals/{muralId}/letters/{letterId}
```

#### Future Letters

```text
GET   /api/murals/{muralId}/future-letters
POST  /api/murals/{muralId}/future-letters
GET   /api/murals/{muralId}/future-letters/{futureLetterId}
PATCH /api/murals/{muralId}/future-letters/{futureLetterId}/open
```

#### Songs

```text
GET    /api/murals/{muralId}/songs
POST   /api/murals/{muralId}/songs
PUT    /api/murals/{muralId}/songs/{songId}
DELETE /api/murals/{muralId}/songs/{songId}
PATCH  /api/murals/{muralId}/songs/{songId}/favorite
```

#### Movies

```text
GET    /api/murals/{muralId}/movies
POST   /api/murals/{muralId}/movies
PUT    /api/murals/{muralId}/movies/{movieId}
DELETE /api/murals/{muralId}/movies/{movieId}
PATCH  /api/murals/{muralId}/movies/{movieId}/favorite
```

#### Calendar

```text
GET    /api/murals/{muralId}/calendar-events
POST   /api/murals/{muralId}/calendar-events
PUT    /api/murals/{muralId}/calendar-events/{eventId}
DELETE /api/murals/{muralId}/calendar-events/{eventId}
```

---

## 16. Environment Variables

Use environment variables for runtime configuration.

Expected variables include:

```text
DB_URL
DB_USER
DB_PASSWORD

JWT_SECRET
JWT_EXPIRATION_MINUTES

MERCADO_PAGO_ACCESS_TOKEN
MERCADO_PAGO_WEBHOOK_SECRET
MERCADO_PAGO_NOTIFICATION_URL

AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
AWS_S3_BUCKET

APP_FRONTEND_URL
APP_BACKEND_URL
CORS_ALLOWED_ORIGINS
```

Never commit real values.

Use example files only with fake placeholders, such as:

```text
.env.example
```

Example files must not contain real credentials.

---

## 17. Testing Rules

Use JUnit 5 and Spring Boot Test.

Security-related tests are important.

Minimum high-value test areas:

- registration/login;
- password hashing;
- mural membership access;
- blocked access to another mural;
- customer entitlement access check;
- Mercado Pago webhook idempotency;
- future letter body locking;
- S3 media file validation;
- media limit validation;
- repository methods scoped by mural id.

Test naming:

```text
*Test.java
```

Use:

```text
*IT.java
```

only if a separate integration-test phase is introduced.

Run before PR or major handoff:

```powershell
.\mvnw.cmd test
```

---

## 18. Implementation Order

Follow this order unless the user explicitly changes priority.

### Phase 1 — Foundation

1. base configuration;
2. application profiles;
3. PostgreSQL configuration;
4. Flyway setup;
5. global exception handling;
6. package structure.

### Phase 2 — Auth and SaaS Base

1. users;
2. accounts;
3. murals;
4. mural members;
5. JWT security;
6. current user;
7. mural access validation.

### Phase 3 — Billing and Payment

1. offers;
2. checkout_orders;
3. payment_transactions;
4. customer_entitlements;
5. billing_agreements;
6. payment_webhook_events;
7. Mercado Pago Pix checkout;
8. Mercado Pago webhook;
9. entitlement activation.

### Phase 4 — S3 Media

1. media_files;
2. S3 configuration;
3. presigned upload URL;
4. complete upload;
5. list media;
6. delete media;
7. media limits.

### Phase 5 — Core Romantic Modules

1. dashboard;
2. memories;
3. letters;
4. future letters;
5. songs;
6. movies;
7. calendar events.

### Phase 6 — Frontend Integration Support

1. CORS;
2. stable DTO responses;
3. endpoint consistency;
4. API error shape;
5. integration notes for frontend fetch calls.

---

## 19. Agent Workflow Rules

Agents must follow these working rules.

### Before Changing Files

Before editing files, provide a short plan describing:

- files likely to change;
- migrations likely to be created;
- dependencies likely to be added;
- risks;
- test commands to run.

Do not implement new modules without user approval.

### Change Size

Prefer small, reviewable changes.

Do not create a large multi-module implementation in one step unless explicitly requested.

### Migrations

Before creating or changing a migration, explain:

- table purpose;
- important columns;
- constraints;
- indexes;
- relationship to the MVP.

### Dependencies

Do not add dependencies without explaining why.

Do not introduce heavy infrastructure unless required by the MVP.

### Secrets

Never ask the user to paste real secrets into the repository.

If secrets are needed, tell the user to configure environment variables locally or in the hosting platform.

### Error Handling

When fixing errors:

1. explain the error;
2. identify likely cause;
3. propose a fix;
4. wait for approval if the user has not already authorized implementation.

---

## 20. Pull Request and Commit Rules

Use concise imperative commit messages:

```text
Add JWT authentication
Create mural billing tables
Implement Pix checkout creation
Add S3 presigned upload flow
```

Pull requests or handoff summaries should include:

- summary;
- changed modules;
- migrations added;
- environment variables added;
- test evidence;
- known limitations;
- next steps.

Before committing, always run:

```powershell
git status
git diff --cached
```

Do not commit generated files, secrets, local configs, logs, target files, or private data.

---

## 21. Definition of Done for MVP Backend

The backend MVP is considered functional when:

1. user can register and login;
2. user can create a draft mural;
3. user can select the one-time mural offer;
4. backend can create Pix checkout with Mercado Pago;
5. Pix payment confirmation through webhook activates entitlement;
6. mural access is blocked without active entitlement;
7. mural access is allowed with active entitlement;
8. user can upload photos to private S3 using presigned URL;
9. media metadata is stored correctly;
10. user can create and list memories;
11. user can create and read regular letters;
12. future letter body is hidden before open date;
13. user can manage songs;
14. user can manage movies;
15. user can manage basic calendar events;
16. all tenant content queries are scoped by mural id;
17. no secrets are committed;
18. tests or manual evidence validate the critical flow.

---

## 22. Final Safety Rules

Prioritize:

```text
security
data isolation
payment correctness
privacy
small steps
clear migrations
working MVP
```

Do not prioritize:

```text
overengineering
microservices
active recurrence
advanced admin
extra modules
visual redesign
premature optimization
```

When uncertain, preserve the MVP decisions in this file and ask for clarification before expanding scope.

## Public Repository Safety Rules

This repository is intended to be public. Agents must treat all commits as public disclosure and must not commit critical data under any circumstance.

Before suggesting `git add`, `git commit`, or `git push`, agents must inspect the intended diff and summarize what will be included. Do not stage or commit automatically without explicit user approval.

Never commit:

- real `.env` files or local environment overrides;
- passwords, tokens, JWT secrets, API keys, cloud credentials, webhook secrets, or private keys;
- certificates, keystores, database dumps, backups, signed URLs, or private payloads;
- real user data, intimate letter or memory content, payment data, or production identifiers;
- local IDE state, logs, build artifacts, or machine-specific files.

Use `.env.example` only for safe variable names and fictitious placeholder values. If a sensitive file or value is found, stop and report the risk before changing or removing important project files.
