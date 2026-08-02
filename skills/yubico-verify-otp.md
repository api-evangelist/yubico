---
name: Verify a Yubico OTP with YubiCloud
description: Validate a YubiKey one-time password against Yubico's hosted YubiCloud service and correctly interpret the signed response.
api: openapi/yubico-yubicloud-openapi.yml
operations:
  - verifyOtp
---

# Verify a Yubico OTP

Use this skill to check whether a 44-character Yubico OTP produced by a YubiKey is valid.

## Prerequisites
- A free Client ID (`id`) and shared secret from https://upgrade.yubico.com/getapikey/.
- The OTP string captured from the YubiKey (44 ModHex characters).

## Steps
1. Generate a fresh, unique `nonce` (16-40 alphanumeric characters) for this request.
2. Call `verifyOtp` — `GET https://api.yubico.com/wsapi/2.0/verify` with query parameters `id`, `otp`, and `nonce`. In production, also compute the HMAC-SHA1 signature `h` over the sorted parameters using your shared secret.
3. Parse the plain-text response (newline-delimited `key=value` pairs), not JSON. Read `status`, `otp`, `nonce`, and `h`.
4. Confirm the returned `nonce` equals the nonce you sent, and verify the response signature `h` against your shared secret. Reject the response if either check fails.
5. Treat the login as successful only when `status=OK`.

## Interpreting status
- `OK` — accept.
- `BAD_OTP`, `REPLAYED_OTP`, `BAD_SIGNATURE`, `REPLAYED_REQUEST` — reject the attempt.
- `MISSING_PARAMETER`, `NO_SUCH_CLIENT`, `OPERATION_NOT_ALLOWED` — fix the request/credentials.
- `BACKEND_ERROR`, `NOT_ENOUGH_ANSWERS` — transient; retry with a new nonce.

See `errors/yubico-error-codes.yml` for the full status catalog and `conventions/yubico-conventions.yml` for signing and replay-protection rules.
