# Webhook Server Implementation

A secure Express.js server implementation that handles webhook requests with signature verification and response generation.

## Features

- 🔐 Secure webhook endpoint with signature verification
- 🔄 Dynamic response generation based on event type
- ⚡ Express.js server with TypeScript support
- 🔑 Environment-based configuration
- 🛡️ Error handling and logging

## Prerequisites

- Node.js (v20 or higher)
- npm
- TypeScript

## Environment Variables

Create a `.env` file in the root directory with the following variables:

```env
SERVER_PORT=4000
WEBHOOK_SECRET=your_webhook_secret_here
NODE_ENV=development
```

## Server Implementation

### Webhook Endpoint

The server exposes a POST endpoint at `/` that handles incoming webhook requests. Here's the implementation:

```typescript
app.post("/", (req: Request, res: Response) => {
  const sigHeader = req.headers["x-signature"];
  const data = req.body;
  const webhookSecret = process.env.WEBHOOK_SECRET;

  if (!webhookSecret) {
    return res.status(500).send("Webhook secret is not set");
  }

  const isValid = verifySignature(data, sigHeader, webhookSecret);
  if (!isValid) return res.status(403).send("Invalid signature");

  return res.status(200).json({
    ...data,
    text:
      data.eventType == "request"
        ? "Hello, how are you?"
        : "I am doing well, thank you for asking.",
  });
});
```

### Signature Verification

The server implements HMAC SHA-256 signature verification to ensure the authenticity of incoming webhook requests:

```typescript
function verifySignature(
  rawBody: string,
  signature: any,
  secret: string
): boolean {
  try {
    const cleanSignature = signature?.trim();
    if (!cleanSignature) return false;

    const hmac = crypto.createHmac("sha256", secret);
    const rawBodyStr =
      typeof rawBody === "string" ? rawBody : JSON.stringify(rawBody);
    hmac.update(rawBodyStr);
    const expected = hmac.digest("base64");

    return crypto.timingSafeEqual(
      Buffer.from(cleanSignature),
      Buffer.from(expected)
    );
  } catch (error) {
    console.error("Signature verification error:", error);
    return false;
  }
}
```

### Server Startup

The server includes professional error handling and logging during startup:

```typescript
app
  .listen(PORT, () => {
    console.log(`🚀 Server is running on http://localhost:${PORT}`);
    console.log(`📝 Environment: ${process.env.NODE_ENV || "development"}`);
  })
  .on("error", (error: NodeJS.ErrnoException) => {
    if (error.code === "EADDRINUSE") {
      console.error(`❌ Port ${PORT} is already in use`);
    } else {
      console.error("❌ Server failed to start:", error.message);
    }
    process.exit(1);
  });
```

## Security Features

1. **Signature Verification**: All incoming webhook requests must include a valid signature in the `x-signature` header
2. **Environment Variables**: Sensitive configuration is managed through environment variables
3. **Error Handling**: Comprehensive error handling for both startup and runtime errors
4. **Type Safety**: TypeScript implementation for better type safety and development experience

## Response Format

The server responds with a JSON object that includes:

- All original request data
- A dynamic text response based on the event type:
  - For `request` events: "Hello, how are you?"
  - For other events: "I am doing well, thank you for asking."

## Error Responses

The server returns appropriate HTTP status codes:

- `200`: Successful request
- `403`: Invalid signature
- `500`: Server configuration error (missing webhook secret)

## Development

To start the development server:

```bash
npm install
npm run dev
```

## Production

For production deployment:

```bash
npm install
npm run dev
```

## Testing

To test the webhook endpoint, you'll need to:

1. Generate a valid signature using your webhook secret
2. Include the signature in the `x-signature` header
3. Send a POST request to the endpoint

### Generating a Valid Signature

Here's how to generate a valid signature using Node.js:

```javascript
const crypto = require('crypto');

// Use the same secret as in your .env file
const webhookSecret = 'your_webhook_secret_here';

// The payload you want to send
const payload = JSON.stringify({ eventType: "request" });

// Generate the signature
const hmac = crypto.createHmac('sha256', webhookSecret);
hmac.update(payload);
const signature = hmac.digest('base64');

console.log('Generated signature:', signature);
```

You can also use this one-liner in your terminal:

```bash
echo -n '{"eventType":"request"}' | openssl dgst -sha256 -hmac "your_webhook_secret_here" -binary | base64
```

Example test request with a properly generated signature:

```bash
curl -X POST http://localhost:4000 \
  -H "Content-Type: application/json" \
  -H "x-signature: YOUR_GENERATED_SIGNATURE" \
  -d '{"eventType": "request"}'
```
