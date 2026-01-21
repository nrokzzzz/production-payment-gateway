Payment Gateway – Enterprise-Grade Fintech Platform
Introduction

This project showcases a fully functional, production-style payment gateway modeled after modern platforms like Razorpay and Stripe. It enables merchants to onboard, create orders, accept UPI and card payments, host secure checkout flows, process transactions asynchronously, deliver signed webhooks with retries, issue refunds, enforce idempotency, and embed payments using a lightweight JavaScript SDK.

The goal is to simulate real-world fintech system design—including secure API authentication, transaction state machines, background job workers, webhook delivery with cryptographic signatures, retry strategies with exponential backoff, and cross-origin embeddable widgets.

Goals
Phase 1 – Core Gateway

Merchant authentication using API key & secret

Order creation and retrieval

Payment initiation (UPI & Card)

Hosted checkout interface

Persistent storage with a relational database

System health endpoint

Phase 2 – Production Capabilities

Asynchronous processing using Redis + workers

Webhooks with HMAC verification and retry logic

Full and partial refunds

Idempotent request handling

Embeddable JavaScript payment SDK

Feature-rich merchant dashboard

System Design

Dockerized Components

PostgreSQL – Data persistence

Redis – Job queue

API Service – Core backend

Worker Service – Background processing

Dashboard – Merchant portal

Checkout – Hosted payment UI

Directory Layout
payment-gateway/
├── docker-compose.yml
├── README.md
├── .env.example
├── backend/
│   ├── Dockerfile
│   ├── Dockerfile.worker
│   ├── pom.xml / build.gradle
│   └── src/main/java/com/gateway/
│       ├── PaymentGatewayApplication.java
│       ├── config/
│       ├── controllers/
│       ├── models/
│       ├── repositories/
│       ├── services/
│       ├── workers/
│       └── jobs/
├── frontend/
│   ├── Dockerfile
│   └── src/
├── checkout-page/
│   ├── Dockerfile
│   └── src/
└── checkout-widget/
    ├── src/
    │   ├── sdk/
    │   └── iframe-content/
    ├── webpack.config.js
    └── dist/checkout.js

Running the Stack
Launch Everything
docker-compose up -d

Active Services
Component	Container Name	Port
Database	pg_gateway	5432
Redis	redis_gateway	6379
API	gateway_api	8000
Worker	gateway_worker	—
Dashboard	gateway_dashboard	3000
Checkout	gateway_checkout	3001
Environment Setup

Copy .env.example to .env and configure:

DATABASE_URL=postgresql://gateway_user:gateway_pass@postgres:5432/payment_gateway
PORT=8000

# Preloaded test merchant
TEST_MERCHANT_EMAIL=test@example.com
TEST_API_KEY=key_test_abc123
TEST_API_SECRET=secret_test_xyz789

# Simulation settings
UPI_SUCCESS_RATE=0.90
CARD_SUCCESS_RATE=0.95
PROCESSING_DELAY_MIN=5000
PROCESSING_DELAY_MAX=10000

# Deterministic testing
TEST_MODE=false
TEST_PAYMENT_SUCCESS=true
TEST_PROCESSING_DELAY=1000

# Webhook testing
WEBHOOK_RETRY_INTERVALS_TEST=false

Data Model
Merchants

id (UUID, PK)

name, email (unique)

api_key, api_secret

webhook_url, webhook_secret

is_active

created_at, updated_at

Orders

id (order_XXXXXXXXXXXXXX)

merchant_id (FK)

amount, currency

receipt, notes

status

created_at, updated_at

Payments

id (pay_XXXXXXXXXXXXXX)

order_id, merchant_id

amount, currency

method, status

vpa

card_network, card_last4

error_code, error_description

captured

created_at, updated_at

Refunds

id (rfnd_XXXXXXXXXXXXXX)

payment_id, merchant_id

amount, reason

status

created_at, processed_at

Webhook Records

id (UUID)

merchant_id

event, payload

status, attempts

last_attempt_at, next_retry_at

response_code, response_body

created_at

Idempotency Store

key, merchant_id

response

created_at, expires_at

Preloaded Merchant
Field	Value
id	550e8400-e29b-41d4-a716-446655440000
name	Test Merchant
email	test@example.com

api_key	key_test_abc123
api_secret	secret_test_xyz789
webhook_secret	whsec_test_abc123
API Reference
Health

GET /health

{
  "status": "healthy",
  "database": "connected",
  "redis": "connected",
  "worker": "running",
  "timestamp": "2024-01-15T10:30:00Z"
}

Create Order

POST /api/v1/orders

Headers:

X-Api-Key: key_test_abc123
X-Api-Secret: secret_test_xyz789


Body:

{
  "amount": 50000,
  "currency": "INR",
  "receipt": "receipt_123"
}

Fetch Order

GET /api/v1/orders/{order_id}

Initiate Payment (Async)

POST /api/v1/payments

Headers:

X-Api-Key: key_test_abc123
X-Api-Secret: secret_test_xyz789
Idempotency-Key: unique_request_id


UPI:

{
  "order_id": "order_xxx",
  "method": "upi",
  "vpa": "user@paytm"
}


Card:

{
  "order_id": "order_xxx",
  "method": "card",
  "card": {
    "number": "4111111111111111",
    "expiry_month": "12",
    "expiry_year": "2025",
    "cvv": "123",
    "holder_name": "John Doe"
  }
}

Capture

POST /api/v1/payments/{payment_id}/capture

Refund

POST /api/v1/payments/{payment_id}/refunds

Fetch Refund

GET /api/v1/refunds/{refund_id}

Webhook Logs

GET /api/v1/webhooks

Retry Webhook

POST /api/v1/webhooks/{webhook_id}/retry

Webhook System
Events

payment.created

payment.pending

payment.success

payment.failed

refund.created

refund.processed

Each payload is signed using HMAC-SHA256 with the merchant’s webhook secret.

Header:

X-Webhook-Signature: <hex_signature>

Background Jobs

ProcessPaymentJob

DeliverWebhookJob

ProcessRefundJob

Deterministic testing:

TEST_MODE=true
TEST_PAYMENT_SUCCESS=true
TEST_PROCESSING_DELAY=1000
WEBHOOK_RETRY_INTERVALS_TEST=true

Checkout Experience
http://localhost:3001/checkout?order_id=order_xxx


Features:

UPI and Card flows

Live processing state

Polling for status

Success and failure screens

JavaScript SDK
<script src="http://localhost:3001/checkout.js"></script>
<button id="pay-button">Pay Now</button>

<script>
const checkout = new PaymentGateway({
  key: 'key_test_abc123',
  orderId: 'order_xyz',
  onSuccess: (res) => console.log(res),
  onFailure: (err) => console.log(err)
});

checkout.open();
</script>

Merchant Dashboard

Login: /login

Overview: /dashboard

Transactions: /dashboard/transactions

Webhooks: /dashboard/webhooks

Documentation: /dashboard/docs

Webhook Testing
node test-merchant/webhook-receiver.js


Use:

http://host.docker.internal:4000/webhook

Typical Mistakes

Using invalid ID formats

Missing the /health endpoint

Not seeding a test merchant

Omitting data-test-id attributes

Weak card validation

Hardcoding secrets instead of env vars

Starting API before database is ready