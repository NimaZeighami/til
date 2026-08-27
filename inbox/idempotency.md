# Idempotency

Idempotency means that **sending the same request multiple times produces the same result as sending it once**.

It is especially important for APIs that create or modify data, such as:

* Creating orders
* Making payments
* Charging a customer
* Creating transactions
* Sending external provider requests

## Why Is It Needed?

Imagine a customer clicks **"Pay"**.

The frontend sends:

```http
POST /orders
```

The server creates the order successfully, but the response is lost because of a network problem.

The frontend does not know whether the order was created, so it retries:

```http
POST /orders
```

Without idempotency, the server may create **two orders**.

With idempotency, the server recognizes that both requests represent the **same operation** and returns the original result.

---

## How It Works

The client generates a unique `Idempotency-Key` for each operation:

```http
POST /orders
Idempotency-Key: 7f8c9d12-...
```

The server stores the result associated with that key.

If the same key is received again:

```http
POST /orders
Idempotency-Key: 7f8c9d12-...
```

the server does **not perform the operation again.

Instead, it returns the previously stored result.

### Simple Flow

```text
Client
  │
  │ POST /orders
  │ Idempotency-Key: ABC123
  ▼
Server
  │
  ├── Key does not exist
  │
  ├── Create order
  │
  ├── Save result for ABC123
  │
  ▼
Response: Order #1001


Client retries
  │
  │ POST /orders
  │ Idempotency-Key: ABC123
  ▼
Server
  │
  ├── Key ABC123 already exists
  │
  └── Return previous result
  ▼
Response: Order #1001
```

The operation happens **once**, even though the request was sent twice.

---

## Example

### First Request

```http
POST /orders
Idempotency-Key: abc-123
```

Server:

```text
Create order #1001
Save:
abc-123 → order #1001
```

Response:

```json
{
  "id": "1001",
  "status": "created"
}
```

### Retry

```http
POST /orders
Idempotency-Key: abc-123
```

The server finds `abc-123` and returns:

```json
{
  "id": "1001",
  "status": "created"
}
```

It does **not** create order `#1002`.

---

## The Key Must Be Unique Per Operation

A key should represent **one logical operation**.

Good:

```text
Payment #1 → abc-123
Payment #2 → xyz-456
```

Bad:

```text
Every payment → abc-123
```

If the same key is intentionally reused for a different operation, the server should reject it.

For example:

```http
Idempotency-Key: abc-123
```

was originally used to create:

```json
{
  "product": "gift-card",
  "amount": 100
}
```

Then the same key is used with:

```json
{
  "product": "subscription",
  "amount": 500
}
```

The server should return an error such as:

```http
409 Conflict
```

because the key is already associated with a different request.

---

## What Should Be Stored?

Typically, the server stores:

```text
idempotency_key
request fingerprint
status
response
created_at
```

For example:

```text
abc-123
→ request hash: 8a91...
→ status: 201
→ response: { "id": "1001" }
```

The exact implementation depends on the application.

---

## Important: Idempotency Is Not Authentication

Idempotency answers:

> "Have I already processed this operation?"

Authentication answers:

> "Who is making this request?"

They solve different problems.

You usually need both.

```text
Authentication
      +
Authorization
      +
Idempotency
```

---

## Idempotency vs Duplicate Requests

A duplicate request is not necessarily a problem.

For example:

```text
Request 1 → server processes it
Request 2 → same operation is retried
```

The goal of idempotency is to make the second request **safe**.

This is particularly important when the operation has side effects.

Examples:

```text
Create order      → side effect
Charge card       → side effect
Create transaction → side effect
Send payment      → side effect
```

---

## Common HTTP Methods

`GET`, `PUT`, and `DELETE` are generally designed to be idempotent.

`POST` is generally **not** idempotent by default.

For example:

```http
POST /orders
```

can create a new order every time it is called.

Adding an `Idempotency-Key` makes the specific operation safely retryable.

---

## In One Sentence

> **Idempotency makes retries safe by ensuring that the same logical operation is processed only once, even if the client sends the request multiple times.**
