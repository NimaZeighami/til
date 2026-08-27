# Reading API Documentation from Coworkers or Companies

> A practical guide for finding, reading, validating, and understanding API documentation when integrating with an internal service, a coworker's API, or a third-party company.

---

## Table of Contents

* [1. Executive Introduction](#1-executive-introduction)
* [2. Documentation Hierarchy: Best to Worst](#2-documentation-hierarchy-best-to-worst)
* [3. The Universal API Reading Checklist](#3-the-universal-api-reading-checklist)
* [4. Rank 1 — OpenAPI / Swagger](#4-rank-1--openapi--swagger)
* [5. Rank 2 — Official Developer Portal](#5-rank-2--official-developer-portal)
* [6. Rank 3 — Postman / Insomnia Collections](#6-rank-3--postman--insomnia-collections)
* [7. Rank 4 — Source Code & Annotations](#7-rank-4--source-code--annotations)
* [8. Rank 5 — SDK Source Code](#8-rank-5--sdk-source-code)
* [9. Rank 6 — Network Inspection](#9-rank-6--network-inspection)
* [10. Rank 7 — Slack / Unstructured Notes](#10-rank-7--slack--unstructured-notes)
* [11. The 60-Second API Extraction Workflow](#11-the-60-second-api-extraction-workflow)
* [12. How to Handle Conflicting Documentation](#12-how-to-handle-conflicting-documentation)
* [13. Internal vs. External APIs](#13-internal-vs-external-apis)
* [14. Security Rules When Inspecting APIs](#14-security-rules-when-inspecting-apis)
* [15. Final Decision Flow](#15-final-decision-flow)
* [16. Final Mental Model](#16-final-mental-model)

---

# 1. Executive Introduction

When integrating with an API, the difficult part is usually not writing the HTTP request.

The difficult part is understanding the **actual API contract**:

```text
Where is the API?
        ↓
How do I authenticate?
        ↓
Which endpoints exist?
        ↓
What do I send?
        ↓
What do I receive?
        ↓
What can go wrong?
        ↓
What special rules apply?
```

This problem usually appears in two situations.

### Internal APIs

For example:

* Another team owns the service.
* A coworker built the API.
* The backend exists in a company repository.
* Documentation may be incomplete or outdated.
* The API may only be available in a private environment.

### External / Third-Party APIs

For example:

* A company provides an API for your application.
* Documentation is hosted in a Developer Portal.
* A vendor provides Postman collections or SDKs.
* Authentication and production access may require separate setup.

The goal is not simply to find **something** that describes the API.

The goal is to find the **most reliable and complete representation of the API contract**, then verify it against actual behavior when necessary.

---

## What You Are Actually Looking For

Regardless of the documentation format, try to eventually answer these questions:

```text
API Identity
├── What API is this?
├── Which version?
└── Which environment?

Connectivity
├── Base URL
└── Required headers

Authentication
├── API Key?
├── Bearer Token?
├── OAuth?
└── How are credentials obtained/refreshed?

Endpoints
├── What operations exist?
├── Which HTTP methods?
└── Which paths?

Request
├── Path parameters
├── Query parameters
├── Headers
└── Body

Response
├── Status codes
├── Response schema
└── Error schema

Operational Rules
├── Pagination
├── Rate limits
├── Retries
├── Idempotency
└── Timeouts

Async Behavior
├── Webhooks
├── Events
└── Delivery/retry rules

Lifecycle
├── Versioning
├── Deprecation
└── Breaking changes
```

> [!IMPORTANT]
> Do not start implementing an integration after finding only one working request. A single request rarely represents the complete API contract.

---

# 2. Documentation Hierarchy: Best to Worst

A practical ranking of the major ways to acquire API documentation:

| Rank  | Method                        | Reliability | Best Use                                |
| ----- | ----------------------------- | ----------: | --------------------------------------- |
| **1** | OpenAPI / Swagger             |   Very High | Formal API contract                     |
| **2** | Official Developer Portal     |   Very High | Third-party and public APIs             |
| **3** | Postman / Insomnia Collection |        High | Testing and executable examples         |
| **4** | Source Code & Annotations     |        High | Internal APIs and actual implementation |
| **5** | SDK Source Code               |        High | Authentication and client behavior      |
| **6** | Network Inspection            |      Medium | Discovering actual runtime behavior     |
| **7** | Slack / Unstructured Notes    |         Low | Context and filling documentation gaps  |

### Important distinction

The ranking does **not** mean that a lower-ranked source is always less accurate.

For example:

* OpenAPI is usually the best **formal contract**.
* Source code may be the best way to understand **implementation details**.
* Network traffic may be the best way to verify **what actually happens at runtime**.
* Slack may contain a critical business rule that exists nowhere else.

The ranking mainly describes the best **starting point** for understanding an API.

---

# 3. The Universal API Reading Checklist

Before considering an API "understood", try to collect the following information.

## 3.1 API Identity

* [ ] API name
* [ ] API version
* [ ] Documentation version
* [ ] Owner/team/company
* [ ] Last updated date, if available
* [ ] Environment

---

## 3.2 Environment & Connectivity

* [ ] Production Base URL
* [ ] Staging Base URL
* [ ] Sandbox Base URL
* [ ] API version path
* [ ] Required protocol
* [ ] Required ports, if applicable

Example:

```text
Production:
https://api.example.com/v2

Sandbox:
https://sandbox.example.com/v2
```

---

## 3.3 Authentication

* [ ] Authentication mechanism
* [ ] API Key requirements
* [ ] Bearer Token requirements
* [ ] OAuth flow
* [ ] Token endpoint
* [ ] Required scopes
* [ ] Token expiration
* [ ] Refresh mechanism
* [ ] Required permissions

---

## 3.4 Requests

For every important endpoint:

* [ ] HTTP method
* [ ] Endpoint path
* [ ] Path parameters
* [ ] Query parameters
* [ ] Required headers
* [ ] Optional headers
* [ ] Request body
* [ ] Content-Type
* [ ] Required fields
* [ ] Optional fields
* [ ] Validation rules
* [ ] Idempotency requirements

---

## 3.5 Responses

* [ ] Success status codes
* [ ] Response body
* [ ] Response schema
* [ ] Required fields
* [ ] Nullable fields
* [ ] Data types
* [ ] Pagination
* [ ] Metadata
* [ ] Empty response behavior

---

## 3.6 Errors

Check at least:

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Entity
429 Too Many Requests
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
```

Also determine:

* [ ] Error response structure
* [ ] Error codes
* [ ] Error messages
* [ ] Which errors are retryable
* [ ] Which errors are permanent
* [ ] Rate-limit behavior

---

## 3.7 Operational Behavior

* [ ] Rate limits
* [ ] Retry rules
* [ ] Timeout recommendations
* [ ] Idempotency
* [ ] Pagination
* [ ] Sorting
* [ ] Filtering
* [ ] Ordering
* [ ] Eventual consistency
* [ ] Request size limits
* [ ] Response size limits

---

## 3.8 Webhooks & Events

If the API is asynchronous:

* [ ] Webhook endpoint
* [ ] Event types
* [ ] Payload schema
* [ ] Authentication
* [ ] Signature verification
* [ ] Delivery guarantees
* [ ] Retry behavior
* [ ] Duplicate event handling
* [ ] Event ordering
* [ ] Event IDs

---

## 3.9 Lifecycle

* [ ] Versioning strategy
* [ ] Current API version
* [ ] Deprecated endpoints
* [ ] Deprecation timeline
* [ ] Breaking-change policy
* [ ] Changelog
* [ ] Migration guides

---

# 4. Rank 1 — OpenAPI / Swagger

## Priority

**#1 — Preferred source for a formal API contract**

OpenAPI is usually the best starting point when it is available and maintained.

Common files:

```text
openapi.yaml
openapi.yml
openapi.json
swagger.yaml
swagger.json
```

It may also be exposed through a UI:

```text
/swagger
/swagger-ui
/swagger/index.html
/api-docs
```

Or as a raw specification:

```text
/openapi.json
/openapi.yaml
```

---

## Why It Ranks #1

OpenAPI is:

* Structured
* Machine-readable
* Standardized
* Easy to validate
* Easy to generate documentation from
* Easy to import into API tools
* Useful for code generation
* Useful for automated testing

A sufficiently complete OpenAPI document can describe:

```text
Base URLs
Endpoints
HTTP methods
Parameters
Headers
Authentication
Request bodies
Response bodies
Schemas
Status codes
Errors
Examples
```

---

## Core Components

A typical OpenAPI document looks roughly like this:

```text
OpenAPI Document
├── openapi
├── info
├── servers
├── tags
├── paths
│   ├── /users
│   ├── /orders
│   └── /payments
├── components
│   ├── schemas
│   ├── securitySchemes
│   ├── responses
│   ├── parameters
│   └── requestBodies
└── security
```

---

## Standard Reading Guide

### Step 1 — Identify the API Version

Start with:

```yaml
openapi: 3.0.3

info:
  title: Example API
  version: 2.1.0
```

There are two different concepts to distinguish:

```text
OpenAPI specification version
        vs.
Actual API version
```

For example:

```yaml
openapi: 3.0.3
info:
  version: 2.1.0
```

This does **not** mean the API is OpenAPI 2.1.

It means:

```text
OpenAPI format: 3.0.3
API/document version: 2.1.0
```

---

### Step 2 — Find the Base URL

Look at:

```yaml
servers:
  - url: https://api.example.com/v2
```

This gives you the base URL.

An endpoint such as:

```yaml
/users
```

would therefore become:

```text
https://api.example.com/v2/users
```

If multiple servers exist, identify which one corresponds to:

```text
Production
Staging
Sandbox
Development
```

---

### Step 3 — Find Authentication

Look for:

```yaml
securitySchemes:
```

For example:

```yaml
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
```

This commonly means:

```http
Authorization: Bearer <token>
```

Other common types include:

```text
apiKey
http
oauth2
openIdConnect
```

Also inspect:

```yaml
security:
```

because authentication can be defined globally or overridden at the operation level.

---

### Step 4 — Read `paths`

The `paths` section contains the API operations.

Example:

```yaml
paths:
  /users:
    get:
      ...
    post:
      ...

  /users/{id}:
    get:
      ...
    patch:
      ...
```

This represents:

```text
GET    /users
POST   /users
GET    /users/{id}
PATCH  /users/{id}
```

Build a quick endpoint map before reading individual schemas.

---

### Step 5 — Inspect Parameters

Parameters can come from different locations:

```text
path
query
header
cookie
```

Example:

```yaml
parameters:
  - name: id
    in: path
    required: true
    schema:
      type: string
```

Or:

```yaml
parameters:
  - name: page
    in: query
    required: false
    schema:
      type: integer
```

The resulting request might be:

```http
GET /users/123?page=2
```

---

### Step 6 — Inspect `requestBody`

Look for:

```yaml
requestBody:
```

A common structure is:

```text
requestBody
    ↓
content
    ↓
application/json
    ↓
schema
```

Example:

```yaml
requestBody:
  required: true
  content:
    application/json:
      schema:
        $ref: '#/components/schemas/CreateUserRequest'
```

Now navigate to:

```yaml
components:
  schemas:
    CreateUserRequest:
```

That schema defines the actual request structure.

---

### Step 7 — Understand `$ref`

This is extremely important when reading large OpenAPI files.

Example:

```yaml
schema:
  $ref: '#/components/schemas/ErrorResponse'
```

This means:

> Go to the `ErrorResponse` definition inside the **same OpenAPI document**.

The path is:

```text
#
└── components
    └── schemas
        └── ErrorResponse
```

For example:

```yaml
components:
  schemas:
    ErrorResponse:
      type: object
      properties:
        message:
          type: string
```

The `$ref` is therefore not a normal web URL.

It is a **JSON Pointer reference** into the OpenAPI document.

---

### External `$ref`

You may also encounter:

```yaml
$ref: './schemas/error.yaml#/ErrorResponse'
```

This means:

```text
Current OpenAPI file
        ↓
schemas/error.yaml
        ↓
ErrorResponse
```

So there are two common forms:

```text
#/components/schemas/User
```

means:

```text
Same file
```

while:

```text
./schemas/user.yaml#/User
```

means:

```text
Another file
```

---

## Fast Navigation Through `$ref`

For a large OpenAPI file, manually searching every `$ref` is inefficient.

Useful options include:

### Swagger Editor

Open the OpenAPI document in an OpenAPI-aware editor and use its schema navigation and rendered documentation.

### VS Code

Use an OpenAPI-aware extension/language server.

Then, where supported:

```text
Cmd/Ctrl + Click
```

on the `$ref` can navigate to its definition.

You can also quickly search for the schema name:

```text
Cmd/Ctrl + F

ErrorResponse
```

### Practical mental shortcut

When you see:

```yaml
$ref: '#/components/schemas/ErrorResponse'
```

immediately think:

```text
Find:
components → schemas → ErrorResponse
```

---

### Step 8 — Inspect `responses`

Look at:

```yaml
responses:
```

Example:

```yaml
responses:
  '200':
    description: Successful response

  '400':
    description: Invalid request

  '401':
    description: Unauthorized

  '500':
    description: Internal server error
```

Do not only read `200`.

The non-success responses often contain important information about how the API is actually supposed to be used.

---

### Step 9 — Follow Response Schemas

For example:

```yaml
'200':
  content:
    application/json:
      schema:
        $ref: '#/components/schemas/User'
```

Follow:

```text
User
```

and inspect:

```text
Required fields
Types
Nested objects
Arrays
Nullable fields
Enums
```

---

### Step 10 — Inspect Examples

Look for:

```yaml
example:
examples:
```

Examples are extremely useful for understanding the expected payload.

But remember:

> Examples demonstrate usage; schemas define the contract.

---

## Fast-Track Guide — Under 60 Seconds

When time is limited, inspect these in order:

```text
1. info
2. servers
3. security / securitySchemes
4. paths
5. requestBody
6. parameters
7. responses
8. components/schemas
```

You should quickly extract:

```text
Base URL
Authentication
Endpoint
HTTP Method
Required Headers
Parameters
Request Body
Success Response
Important Errors
```

### 60-second mental map

```text
WHERE?
→ servers

WHO AM I?
→ securitySchemes

WHAT CAN I CALL?
→ paths

WHAT DO I SEND?
→ parameters + requestBody

WHAT DO I GET?
→ responses + schemas

WHAT CAN GO WRONG?
→ error responses
```

---

## Common OpenAPI Mistakes

### Mistake 1 — Confusing OpenAPI Version with API Version

```yaml
openapi: 3.0.3
info:
  version: 1.4.0
```

These are different things.

---

### Mistake 2 — Ignoring Global Security

Authentication may be defined globally:

```yaml
security:
  - bearerAuth: []
```

You may not see authentication repeated under every endpoint.

---

### Mistake 3 — Reading Only `paths`

The endpoint definitions often reference schemas elsewhere.

Always follow:

```text
$ref
```

---

### Mistake 4 — Assuming Every `$ref` Is a URL

This:

```yaml
$ref: '#/components/schemas/User'
```

is an internal document reference, not a URL to open in a browser.

---

### Mistake 5 — Assuming OpenAPI Is Always Current

OpenAPI is only as trustworthy as its maintenance.

Always check:

```text
Version
Environment
Last update
Deployment
```

when possible.

---

# 5. Rank 2 — Official Developer Portal

## Priority

**#2 — Preferred source for third-party APIs and vendor integrations**

Many companies provide a dedicated Developer Portal instead of handing you a raw OpenAPI file.

Typical structure:

```text
Developer Portal
├── Getting Started
├── Quickstart
├── Authentication
├── API Reference
├── Guides
├── SDKs
├── Webhooks
├── Errors
├── Rate Limits
├── Changelog
└── Sandbox
```

---

## Why It Ranks #2

A good Developer Portal often contains information that a raw OpenAPI specification does not fully explain:

* Business rules
* Authentication setup
* Account configuration
* Production onboarding
* Sandbox behavior
* Webhook behavior
* Rate limits
* Migration instructions
* Versioning
* Deprecation
* SDK usage
* Workflow examples

It is particularly valuable for external APIs because it is usually maintained by the API provider.

---

## Core Components

Look for:

```text
Quickstart
Authentication
API Reference
Guides
Base URLs
Request Examples
Response Examples
Error Reference
Webhooks
Rate Limits
SDK Documentation
Changelog
Migration Guides
```

---

## Standard Reading Guide

### Step 1 — Start with Quickstart

Look for:

```text
Getting Started
Quickstart
First API Call
Introduction
```

This usually shows the provider's recommended integration flow.

---

### Step 2 — Identify Environments

Determine whether the API has:

```text
Sandbox
Development
Staging
Production
```

Do not assume:

```text
Sandbox URL = Production URL
```

---

### Step 3 — Understand Authentication

Determine:

```text
How credentials are created
How credentials are sent
How tokens expire
How tokens are refreshed
Which scopes are required
Which permissions are required
```

---

### Step 4 — Map the API

Create a simple endpoint map:

```text
GET  /products
POST /orders
GET  /orders/{id}
POST /refunds
```

This gives you a high-level picture before diving into individual endpoints.

---

### Step 5 — Read Operational Documentation

Do not stop at the API Reference.

Also inspect:

```text
Rate Limits
Retries
Idempotency
Pagination
Webhooks
Errors
Versioning
Changelog
```

These details often determine whether an integration is reliable in production.

---

## Fast-Track Guide — Under 60 Seconds

Search the portal for:

```text
Authentication
Base URL
Quickstart
API Reference
Errors
Webhooks
Rate Limits
```

Then extract:

```text
Production URL
Authentication
Main Endpoint
Required Request
Success Response
Error Behavior
```

---

## Common Mistakes

### Mistake 1 — Starting with Every Endpoint

Read:

```text
Quickstart
→ Authentication
→ Core workflow
→ API Reference
```

instead of trying to read the entire portal.

### Mistake 2 — Ignoring Changelogs

A working endpoint may have changed behavior between API versions.

### Mistake 3 — Ignoring Business Rules

An API may technically accept a request while the business rules make that request invalid.

---

# 6. Rank 3 — Postman / Insomnia Collections

## Priority

**#3 — Excellent for executable examples and fast exploration**

Postman and Insomnia collections provide practical API requests that can often be executed immediately.

Common Postman format:

```text
collection.json
```

Typical structure:

```text
Collection
├── Authentication
├── Users
├── Products
├── Orders
├── Payments
└── Webhooks
```

---

## Why It Ranks #3

A collection is valuable because it provides **executable examples**.

For example:

```http
POST https://api.example.com/orders

Authorization: Bearer {{token}}
Content-Type: application/json

{
  "productId": "123",
  "quantity": 2
}
```

You can often run this request immediately and compare the result with the documented example.

---

## Core Components

```text
Collection
├── Folders
├── Requests
│   ├── Method
│   ├── URL
│   ├── Headers
│   ├── Query Parameters
│   ├── Body
│   └── Examples
├── Variables
├── Environments
├── Authorization
├── Pre-request Scripts
└── Tests
```

---

## Standard Reading Guide

### Step 1 — Inspect Variables

Look for:

```text
base_url
api_url
token
access_token
client_id
client_secret
environment
```

Example:

```text
{{base_url}}/v1/orders
```

First determine what:

```text
{{base_url}}
```

actually represents.

---

### Step 2 — Check Environment Files

You may have:

```text
Development
Staging
Production
```

Do not accidentally use production credentials or URLs while testing.

---

### Step 3 — Inspect Collection-Level Authorization

Authentication may be defined once at the collection level.

Individual requests may simply inherit it.

Also check:

```text
Folder-level authorization
Request-level authorization
```

---

### Step 4 — Map the Folders

Folders often reveal the API's capabilities:

```text
Authentication
Products
Customers
Orders
Payments
Refunds
```

This provides a quick API map.

---

### Step 5 — Inspect Individual Requests

For each important request, extract:

```text
HTTP Method
URL
Headers
Query Parameters
Path Parameters
Body
Response Example
```

---

### Step 6 — Inspect Scripts

Look at:

```text
Pre-request Scripts
Tests
```

For example:

```javascript
pm.environment.set("token", response.token);
```

This may reveal how authentication or variables are managed.

---

## Fast-Track Guide — Under 60 Seconds

Check:

```text
1. Environment Variables
2. Collection Authorization
3. Authentication Request
4. One main business request
5. Request Body
6. Response Example
```

You should quickly identify:

```text
Base URL
Authentication
Token Endpoint
Required Headers
Main Endpoint
Request JSON
Response JSON
```

---

## Postman / Insomnia Pitfalls

### Hidden Variables

A request like:

```text
{{base_url}}/orders/{{order_id}}
```

cannot be fully understood until you inspect the variables.

### Inherited Authentication

Do not assume a request has no authentication because its own Authorization section appears empty.

It may inherit authentication from:

```text
Collection
Folder
Environment
```

### Scripts

Pre-request and test scripts may be essential to making the collection work.

---

# 7. Rank 4 — Source Code & Annotations

## Priority

**#4 — Excellent for internal APIs when formal documentation is missing or incomplete**

If you have access to the backend repository, the source code can reveal how the API is actually implemented.

Example:

```typescript
@Post("/orders")
createOrder(@Body() request: CreateOrderRequest) {
  ...
}
```

This tells you that the API exposes:

```text
POST /orders
```

---

## Why It Ranks #4

Source code can reveal:

* Routes
* Controllers
* HTTP methods
* Request models
* Validation
* Authentication middleware
* Response models
* Serialization
* Business rules
* Implementation details

However, source code is not automatically the public API contract.

A repository may contain:

```text
Unreleased code
Feature branches
Dead code
Internal endpoints
Different deployment versions
```

Therefore:

> Source code is often the best source for understanding implementation, but not necessarily the definitive source for what is currently deployed.

---

## Core Components

Depending on the framework:

```text
Routes
Controllers
Handlers
Middleware
Guards
DTOs
Schemas
Validators
Serializers
Response Models
Authentication
```

---

## Common Framework Patterns

### NestJS

```typescript
@Controller("orders")
@Post()
@Get(":id")
```

### Spring

```java
@RestController
@RequestMapping("/orders")
@PostMapping
@GetMapping("/{id}")
```

### ASP.NET

```csharp
[ApiController]
[Route("api/orders")]
[HttpPost]
[HttpGet("{id}")]
```

### FastAPI

```python
@app.post("/orders")
@app.get("/orders/{id}")
```

---

## Standard Reading Guide

### Step 1 — Find Routes and Controllers

Search for:

```text
Controller
Router
Route
GET
POST
PUT
PATCH
DELETE
```

---

### Step 2 — Reconstruct the Full Route

Routes may be composed from multiple levels.

For example:

```text
Application prefix:
 /api

Controller:
 /orders

Method:
 POST /

Final endpoint:
 POST /api/orders
```

---

### Step 3 — Find the Request DTO

Follow the request parameter:

```text
Controller
    ↓
Request DTO
    ↓
Validation
```

Look for:

```text
DTO
Schema
Validator
Interface
Type
Model
```

---

### Step 4 — Find Authentication

Search for:

```text
auth
authorize
middleware
guard
JWT
Bearer
API key
OAuth
permission
role
scope
```

Authentication may be applied before the controller and therefore may not be visible inside the endpoint itself.

---

### Step 5 — Follow the Response

Trace:

```text
Handler
    ↓
Service
    ↓
Mapper / Serializer
    ↓
Response DTO
```

This helps determine what the API actually returns.

---

## Fast-Track Guide — Under 60 Seconds

Search the repository for framework-specific route declarations:

```text
@Controller
router.get(
router.post(
@PostMapping
@GetMapping
[HttpPost]
[HttpGet]
@app.get(
@app.post(
```

Then follow:

```text
Route
 ↓
Handler
 ↓
Request DTO
 ↓
Validation
 ↓
Authentication
 ↓
Response DTO
```

---

## Important Limitation

Source code can tell you:

```text
"What the code says should happen."
```

Network inspection can tell you:

```text
"What actually happened in this runtime."
```

A deployed API may differ from the current repository.

Always consider:

```text
Repository version
        vs.
Deployed version
        vs.
API documentation version
```

---

# 8. Rank 5 — SDK Source Code

## Priority

**#5 — Useful for understanding client behavior and hidden implementation details**

If a company provides an official SDK, it can reveal how the API is expected to be consumed.

Common SDK languages:

```text
JavaScript / TypeScript
Python
Go
Java
C#
PHP
Ruby
```

---

## Why It Ranks #5

An SDK can expose:

* Endpoint paths
* HTTP methods
* Authentication
* Request models
* Response models
* Serialization
* Error handling
* Retries
* Configuration
* Default headers

Example:

```typescript
client.orders.create({
  productId: "123"
})
```

may internally become:

```http
POST /orders
```

---

## Core Components

```text
SDK
├── Client
├── Authentication
├── Resources
│   ├── users
│   ├── orders
│   └── payments
├── Request Models
├── Response Models
├── Error Models
├── Configuration
└── HTTP Transport
```

---

## Standard Reading Guide

### Step 1 — Find the Main Client

Search for:

```text
Client
ApiClient
HttpClient
createClient
```

---

### Step 2 — Find Authentication

Search for:

```text
Authorization
Bearer
apiKey
access_token
client_id
client_secret
OAuth
```

---

### Step 3 — Find Resources

Look for methods such as:

```text
create
get
list
update
delete
```

Then follow the implementation.

---

### Step 4 — Find the HTTP Layer

Determine:

```text
Base URL
HTTP library
Headers
Serialization
Timeout
Retry behavior
```

---

### Step 5 — Inspect Models

Search for:

```text
Request
Response
DTO
Schema
Interface
Type
Model
```

---

## Fast-Track Guide — Under 60 Seconds

Search for:

```text
baseURL
Authorization
/api/
POST
GET
create(
```

Then follow:

```text
SDK Method
    ↓
Resource
    ↓
HTTP Client
    ↓
URL
    ↓
Headers
    ↓
Request Body
    ↓
Response Model
```

---

## Important Limitation

An SDK can be:

```text
Outdated
Behind the API
Ahead of the documentation
Incomplete
An abstraction over undocumented behavior
```

Therefore, use it as a strong implementation reference, but verify version compatibility.

---

# 9. Rank 6 — Network Inspection

## Priority

**#6 — Useful for discovering actual runtime behavior**

When documentation is missing or incomplete, inspect the HTTP traffic generated by an application you are authorized to use.

For web applications, the easiest starting point is usually:

```text
Browser DevTools
→ Network
→ Fetch/XHR
```

---

## Why It Ranks #6

Network inspection shows what actually happened:

```text
URL
HTTP Method
Query Parameters
Headers
Request Body
Cookies
Response Body
Status Code
Timing
```

This is particularly useful for finding:

* Undocumented endpoints
* Actual request formats
* Required headers
* Unexpected response fields
* Client-side API usage
* Differences between documentation and runtime behavior

---

## Core Components

### Request

```text
Request
├── URL
├── Method
├── Query Parameters
├── Headers
├── Cookies
└── Body
```

### Response

```text
Response
├── Status
├── Headers
└── Body
```

---

## Standard Reading Guide

### Step 1 — Open Network Tools

In a browser:

```text
Developer Tools
→ Network
→ Fetch/XHR
```

---

### Step 2 — Perform One Action

For example:

```text
Login
Search
Load Product
Create Order
Update Profile
```

---

### Step 3 — Inspect the Request

Record:

```text
Method
URL
Headers
Query
Body
Authentication
```

---

### Step 4 — Inspect the Response

Record:

```text
Status Code
Response Headers
Response JSON
Error Structure
Pagination
Metadata
```

---

### Step 5 — Follow the Full Workflow

One request is often not enough.

For example:

```text
Login
   ↓
Create Order
   ↓
Get Order
   ↓
Receive Status Update
```

Understanding the sequence can be more important than understanding a single endpoint.

---

## Fast-Track Guide — Under 60 Seconds

```text
1. Open Network
2. Filter Fetch/XHR
3. Perform the desired action
4. Select the request
5. Inspect URL
6. Inspect Headers
7. Inspect Payload
8. Inspect Response
```

When available:

```text
Copy
→ Copy as cURL
```

This gives you a reproducible representation of the request.

---

## Important Limitation

Network traffic only shows requests that actually happened.

It does not automatically reveal:

```text
All available endpoints
Future endpoints
Unused functionality
Business rules
Full schema
Rate limits
Deprecation policy
```

It is therefore best used as a **verification and discovery tool**, not as the primary documentation source when a proper contract exists.

---

# 10. Rank 7 — Slack / Unstructured Notes

## Priority

**#7 — Last resort and supplementary source**

Examples:

```text
Slack
Microsoft Teams
Email
Tickets
Notion
Google Docs
Markdown notes
Spreadsheets
Chat messages
```

These sources can contain useful information, but they should rarely be treated as the primary API contract.

---

## Why It Ranks #7

Unstructured information is often:

```text
Incomplete
Outdated
Context-dependent
Contradictory
Hard to search
Missing schemas
Missing versions
Missing environments
```

For example:

> "Just call `/order` with the token."

This does not answer:

```text
GET or POST?
Which Base URL?
Which headers?
Which token?
Which request body?
Which response?
Which errors?
Which version?
Which environment?
```

---

## Core Components

Look for:

```text
Endpoint URL
cURL Example
Request JSON
Response JSON
Base URL
Authentication
Known Errors
Business Rules
API Version
Environment
```

---

## Standard Reading Guide

### Step 1 — Check the Date

Look for:

```text
Author
Date
Last Updated
API Version
Environment
```

Old information is not automatically wrong, but it requires verification.

---

### Step 2 — Separate Facts from Assumptions

Example:

```text
Confirmed:
Production URL = https://api.example.com

Unverified:
"I think the token expires after 24 hours."
```

Do not treat both as equally reliable.

---

### Step 3 — Cross-Check

Verify important information against:

```text
OpenAPI
Developer Portal
Postman
Source Code
SDK
Actual Network Requests
```

---

### Step 4 — Convert Notes into Structured Information

Turn:

```text
"We usually send this header and then call the order endpoint..."
```

into:

```text
Base URL
Authentication
Endpoint
Headers
Request
Response
Errors
Business Rules
Open Questions
```

---

## Fast-Track Guide — Under 60 Seconds

Search for:

```text
swagger
openapi
postman
base URL
token
auth
bearer
endpoint
curl
webhook
sandbox
production
```

Then extract only:

```text
URL
Auth
Endpoint
Request
Response
```

Everything else should be verified before relying on it.

---

# 11. The 60-Second API Extraction Workflow

When you receive an unfamiliar API and need to understand the basics immediately, use this workflow.

```text
0–10 sec
↓
Find Base URL

10–20 sec
↓
Find Authentication

20–35 sec
↓
Find Main Endpoint

35–45 sec
↓
Find Request

45–55 sec
↓
Find Response

55–60 sec
↓
Find Errors / Special Rules
```

---

## The Six Questions

### 1. Where is it?

```text
Base URL:
https://api.example.com/v2
```

### 2. How do I authenticate?

```text
Bearer Token
```

### 3. What do I call?

```text
POST /orders
```

### 4. What do I send?

```json
{
  "productId": "123",
  "quantity": 2
}
```

### 5. What do I receive?

```json
{
  "id": "order_123",
  "status": "created"
}
```

### 6. What can go wrong?

```text
400
401
403
409
422
429
500
```

---

## The 60-Second Output

After one minute, you should be able to produce something like:

```text
API:
Payment API v2

Base URL:
https://api.example.com/v2

Authentication:
Bearer Token

Endpoint:
POST /payments

Headers:
Authorization
Content-Type
Idempotency-Key

Request:
{
  ...
}

Success:
201 Created

Important Errors:
400
401
409
429
500
```

If you cannot create this summary, you probably have not yet identified the core API contract.

---

# 12. How to Handle Conflicting Documentation

Conflicting information is common in real-world integrations.

You may find:

```text
OpenAPI says:
POST /orders

Postman says:
POST /v2/orders

Source code says:
POST /api/orders

Slack says:
POST /order
```

Do not simply choose one.

---

## Step 1 — Check the Version

Determine:

```text
API Version
Documentation Version
Repository Version
SDK Version
```

---

## Step 2 — Check the Environment

A difference may simply be:

```text
Development
vs.
Staging
vs.
Production
```

---

## Step 3 — Check the Deployment

Source code may represent:

```text
Current repository
```

while the running API represents:

```text
Deployed version
```

These may differ.

---

## Step 4 — Identify the Authority

A useful hierarchy is:

```text
Current deployed behavior
        +
Official current contract
        +
Official implementation
        +
Executable examples
        +
Human notes
```

The exact authority depends on what you are trying to establish.

For example:

| Question                                         | Best Evidence                             |
| ------------------------------------------------ | ----------------------------------------- |
| What is officially supported?                    | Current official documentation / contract |
| What schema should clients use?                  | OpenAPI / official API reference          |
| How is it implemented internally?                | Source code                               |
| What request does the application actually send? | Network inspection                        |
| Why does this business rule exist?               | Official business documentation / owner   |
| What does an old integration use?                | Existing client code / historical notes   |

---

## Step 5 — Never Silently Resolve Conflicts

If the difference matters, record it:

```text
> [!WARNING]
> OpenAPI documents `/v2/orders`, while the current staging client
> sends `/api/orders`. Verify the deployed API version before implementation.
```

This is much safer than silently choosing one.

---

# 13. Internal vs. External APIs

The best discovery strategy depends on who owns the API.

---

## Internal API

For an internal service, use:

```text
1. OpenAPI / Swagger
2. Source Code
3. Postman / Insomnia
4. Internal Developer Portal
5. SDK
6. Network Inspection
7. Slack / Notes
```

Why?

Because you may have access to the actual implementation and deployment environment.

---

## External API

For a third-party company, use:

```text
1. Official Developer Portal
2. OpenAPI / Swagger
3. Postman / Insomnia
4. Official SDK
5. Vendor examples
6. Network Inspection
7. Support messages / Notes
```

Why?

Because the provider's official documentation is usually the authoritative source for:

```text
Authentication
API versions
Production access
Rate limits
Supported behavior
Deprecation
Business rules
```

---

## Quick Comparison

| Situation                               | Start Here            |
| --------------------------------------- | --------------------- |
| Internal backend                        | OpenAPI / Source Code |
| External SaaS API                       | Developer Portal      |
| API needs immediate testing             | Postman / Insomnia    |
| No documentation, but repository exists | Source Code           |
| Official client library exists          | SDK                   |
| Existing web application calls the API  | Network Inspection    |
| Nothing else exists                     | Notes / Slack         |

---

# 14. Security Rules When Inspecting APIs

API documentation and runtime traffic can contain sensitive information.

Be especially careful with:

```text
API Keys
Access Tokens
Refresh Tokens
Session Cookies
Client Secrets
Passwords
Private Certificates
Authorization Headers
Personal Data
Production Credentials
```

---

## Never Commit Secrets

A copied request may contain:

```http
Authorization: Bearer eyJ...
Cookie: session=...
X-API-Key: abc123...
```

Do not put this directly into:

```text
Git
GitHub
Documentation
Slack
Tickets
Screenshots
Postman collections
```

---

## Redact Sensitive Values

Instead of:

```http
Authorization: Bearer eyJhbGciOi...
```

use:

```http
Authorization: Bearer <REDACTED>
```

Instead of:

```text
X-API-Key: abc123
```

use:

```text
X-API-Key: <API_KEY>
```

---

## Be Careful With "Copy as cURL"

Browser DevTools and Postman can generate useful cURL commands, but they may include:

```text
Authorization headers
Cookies
API keys
Session identifiers
Personal data
```

Before saving or sharing a cURL command:

```text
Copy
   ↓
Inspect
   ↓
Redact secrets
   ↓
Remove unnecessary personal data
   ↓
Then save/share
```

> [!WARNING]
> Only inspect or reproduce API traffic that you are authorized to access. Never use credentials or session data belonging to another person or system without authorization.

---

# 15. Final Decision Flow

Use this flow whenever you need to understand an unfamiliar API.

```text
                 Need API Documentation
                          │
                          ▼
                 Is OpenAPI available?
                    /            \
                  Yes             No
                   │               │
                   ▼               ▼
               OpenAPI      Is there an official
                             Developer Portal?
                              /          \
                            Yes           No
                             │             │
                             ▼             ▼
                          Portal      Is there a
                                     Postman/Insomnia
                                       Collection?
                                      /         \
                                    Yes          No
                                     │            │
                                     ▼            ▼
                                  Collection   Do you have
                                               source code?
                                               /       \
                                             Yes        No
                                              │          │
                                              ▼          ▼
                                           Source      Is there
                                            Code       an SDK?
                                                       /   \
                                                     Yes    No
                                                      │      │
                                                      ▼      ▼
                                                     SDK   Can you
                                                           inspect
                                                           runtime
                                                           traffic?
                                                           /   \
                                                         Yes    No
                                                          │      │
                                                          ▼      ▼
                                                       Network  Notes /
                                                       Traffic  Slack
```

---

## Recommended Strategy

Do not think of the methods as mutually exclusive.

A strong integration usually combines multiple sources:

```text
OpenAPI
   +
Developer Portal
   +
Postman
   +
Source Code
   +
Runtime Verification
```

For example:

```text
OpenAPI
  ↓
Find endpoint and schema

Developer Portal
  ↓
Understand authentication and business rules

Postman
  ↓
Run a real example

Source Code
  ↓
Understand internal implementation

Network
  ↓
Verify actual runtime behavior
```

This is much more reliable than trusting one source blindly.

---

# 16. Final Mental Model

The goal is not:

> "Find the API documentation."

The real goal is:

> **Find the most reliable representation of the API contract, understand it, and verify the important parts.**

A useful hierarchy is:

```text
Formal Contract
      ↓
Official Documentation
      ↓
Executable Examples
      ↓
Official Implementation
      ↓
Runtime Behavior
      ↓
Human Context
```

Each layer answers different questions.

---

## Formal Contract

Usually tells you:

```text
What exists?
What are the schemas?
What parameters are required?
What responses are expected?
```

---

## Official Documentation

Usually tells you:

```text
How should I use it?
What business rules apply?
How do I authenticate?
What are the production requirements?
```

---

## Executable Examples

Usually tell you:

```text
What does a real request look like?
What headers are required?
What does a real response look like?
```

---

## Source Code

Usually tells you:

```text
How is it implemented?
Where is validation performed?
How does authentication work?
What happens internally?
```

---

## Runtime Behavior

Usually tells you:

```text
What actually happened?
What request did the client send?
What response did the server return?
```

---

## Human Context

Usually tells you:

```text
Why does this business rule exist?
What undocumented behavior does the team rely on?
What historical context matters?
```

But human context should be verified whenever possible.

---

# Golden Rules

```text
1. Prefer structured documentation over scattered messages.

2. Prefer official sources over assumptions.

3. Prefer the current API version over old examples.

4. Always identify the environment.

5. Never assume authentication from one request alone.

6. Follow every $ref in OpenAPI.

7. Read request and response schemas, not just endpoint names.

8. Read error responses, not just 200/201 responses.

9. Check rate limits, retries, pagination, and idempotency.

10. Treat webhooks as part of the API contract.

11. Do not assume source code equals deployed behavior.

12. Do not assume network traffic represents the entire API.

13. Verify conflicting information instead of guessing.

14. Redact credentials from examples and cURL commands.

15. Document unknowns explicitly instead of silently making assumptions.
```

---

# One-Page Cheat Sheet

```text
┌───────────────────────────────────────────────┐
│             API DISCOVERY CHECKLIST           │
├───────────────────────────────────────────────┤
│                                               │
│ 1. BASE URL                                   │
│    Where is the API?                         │
│                                               │
│ 2. VERSION                                    │
│    Which API version am I using?             │
│                                               │
│ 3. AUTH                                       │
│    API Key / Bearer / OAuth?                 │
│                                               │
│ 4. ENDPOINT                                   │
│    Method + Path                             │
│                                               │
│ 5. PARAMETERS                                 │
│    Path / Query / Headers                    │
│                                               │
│ 6. REQUEST                                    │
│    Body + Required Fields                    │
│                                               │
│ 7. RESPONSE                                   │
│    Status + Schema                           │
│                                               │
│ 8. ERRORS                                     │
│    4xx / 5xx + Error Schema                  │
│                                               │
│ 9. RULES                                      │
│    Rate Limit / Retry / Pagination           │
│    Idempotency / Timeout                     │
│                                               │
│ 10. WEBHOOKS                                  │
│     Events / Signatures / Retries            │
│                                               │
│ 11. LIFECYCLE                                 │
│     Versioning / Deprecation / Changelog     │
│                                               │
└───────────────────────────────────────────────┘
```

---

# Final Summary

When receiving an unfamiliar API, use this order of attack:

```text
                    API
                     │
                     ▼
             Find the best source
                     │
        ┌────────────┴────────────┐
        ▼                         ▼
   OpenAPI / Portal          No formal docs
        │                         │
        ▼                         ▼
   Read the contract        Postman / SDK / Code
        │                         │
        └────────────┬────────────┘
                     ▼
              Understand Auth
                     │
                     ▼
              Map Endpoints
                     │
                     ▼
          Read Request Schemas
                     │
                     ▼
          Read Response Schemas
                     │
                     ▼
             Read Error Cases
                     │
                     ▼
        Check Operational Rules
                     │
                     ▼
          Check Webhooks/Events
                     │
                     ▼
           Verify Runtime Behavior
                     │
                     ▼
              Start Coding
```

> **Contract first → Understand → Verify → Implement.**

This approach minimizes assumptions, reduces integration bugs, and makes it much easier to work with both internal APIs and third-party services.
