# boba-auth-backend

## Environment setup

Create `.env` in the project root (or copy from `.env.example`) and set:

```bash
PORT=4000
NODE_ENV=development
OTP_TTL_MS=300000
OTP_MAX_ATTEMPTS=5
OTP_RESEND_COOLDOWN_MS=30000
JWT_SECRET=dev_jwt_secret_change_me
REFRESH_TOKEN_TTL_MS=604800000
AWS_ACCESS_KEY_ID=your_access_key_id
AWS_SECRET_ACCESS_KEY=your_secret_access_key
AWS_REGION=ap-south-1
REDIS_URL=redis://localhost:6379
RATE_LIMIT_WINDOW_MS=60000
RATE_LIMIT_MAX_REQUESTS=100
OTP_SEND_MAX_REQUESTS=5
OTP_VERIFY_MAX_REQUESTS=10
AUTH_MAX_REQUESTS=30
DUMMY_OTP=123456
```

Notes:
- `AWS_*` values are used for real OTP SMS delivery through Amazon SNS.
- If SNS send fails, backend falls back to `DUMMY_OTP` for development testing.
- OTP values are never returned in API responses.
- Redis is optional but recommended for production session/token storage.

## Quick API test (`curl`)

Start server:

```bash
npm start
```

Set base URL:

```bash
BASE_URL="http://localhost:4000"
```

### 1) Send OTP

```bash
curl -X POST "$BASE_URL/api/otp/send" \
  -H "Content-Type: application/json" \
  -d '{"phone_number":"+919999999999"}'
```

### 2) Verify OTP (returns `auth_token` + `refresh_token`)

Use the `otp_session_id` and OTP received on phone (or `DUMMY_OTP` if fallback mode is active):

```bash
curl -X POST "$BASE_URL/api/otp/verify" \
  -H "Content-Type: application/json" \
  -d '{"phone_number":"+919999999999","session_id":"<otp_session_id>","otp":"<otp>"}'
```

### 3) Resend OTP

```bash
curl -X POST "$BASE_URL/auth/resend-otp" \
  -H "Content-Type: application/json" \
  -d '{"phone_number":"+919999999999","session_id":"<otp_session_id>"}'
```

### 4) Cancel OTP session

```bash
curl -X POST "$BASE_URL/auth/cancel" \
  -H "Content-Type: application/json" \
  -d '{"session_id":"<otp_session_id>"}'
```

### 5) Refresh access token

Use `refresh_token` returned by `/api/otp/verify`:

```bash
curl -X POST "$BASE_URL/auth/refresh-token" \
  -H "Content-Type: application/json" \
  -d '{"refresh_token":"<refresh_token>"}'
```

### 6) Health check

```bash
curl "$BASE_URL/health"
```

### 7) Logout (revoke refresh token)

```bash
curl -X POST "$BASE_URL/auth/logout" \
  -H "Content-Type: application/json" \
  -d '{"refreshToken":"<refresh_token>"}'
```

## Automated OTP flow test script

Run:

```bash
./scripts/test-otp-flow.sh
```

Useful overrides:

```bash
# If OTP came on real phone:
OTP_CODE=123456 ./scripts/test-otp-flow.sh

# Different base URL / number:
BASE_URL=http://localhost:4000 PHONE_NUMBER=+919999999999 ./scripts/test-otp-flow.sh
```

