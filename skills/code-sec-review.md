---
shortDescription: Security-focused code review procedure covering OWASP Top 10 attack categories.
usedBy: [reviewer]
version: 0.0.2
lastUpdated: 2026-04-07
---

## Purpose

The standard code review checks for correctness bugs — security is one bullet among many. This skill expands that bullet into a structured security review organized by attack category. It tells the reviewer exactly what to look for so that security findings come from a checklist rather than memory. This is a static review — it examines code as written. It does not run exploits, fuzz endpoints, or perform dynamic testing.

## Procedure

1. **Map the attack surface.** Before checking for specific vulnerabilities, understand what the change exposes. Scan the diff and answer:
   - Does this change introduce or modify an endpoint, handler, webhook, or queue consumer that accepts external input?
   - Does it add or change authentication, authorization, or session management logic?
   - Does it introduce a new data flow to or from an external system (database, API, file system, message broker)?
   - Does it handle file uploads, URL fetching, redirects, or email sending?

   If the answer to all four is no, the change has no new attack surface — skip to step 6 (Dependencies). The remaining passes apply only when attack surface exists.

2. **Injection analysis.** Trace every path where untrusted data enters the system and follow it to where it is consumed. Untrusted data includes: user input, HTTP headers, query parameters, request bodies, URL path segments, file contents, database results, API responses, environment variables, and message queue payloads.

   For each untrusted data flow, verify that the data is validated or sanitized before reaching any of these sinks:

   | Sink | What to check | Ref |
   |------|---------------|-----|
   | SQL / NoSQL query | Parameterized queries or ORM — no string concatenation or interpolation. For NoSQL, no user-controlled operators (`$ne`, `$gt`, `$regex`, `$where`). | CWE-89, CWE-943 |
   | OS command | No shell execution with user input. If unavoidable, allowlist of permitted values — never sanitize-and-pass. | CWE-78 |
   | HTML / template output | Context-aware escaping (HTML body, attribute, JS, URL, CSS contexts each need different encoding). Framework auto-escaping enabled and not bypassed (`dangerouslySetInnerHTML`, `\| safe`, `Html.Raw`, `v-html`). | CWE-79 |
   | HTTP header | No CRLF sequences (`\r\n`) in header values. Applies to `Set-Cookie`, `Location`, `Content-Disposition`, and custom headers. | CWE-113 |
   | URL redirect / forward | Destination validated against an allowlist of trusted domains. No open redirects via user-controlled `returnUrl`, `next`, `redirect_uri`, or `Location` header. | CWE-601 |
   | Server-side HTTP request | User-supplied URLs validated against an allowlist. Deny private/loopback ranges (`127.0.0.1`, `169.254.169.254`, `10.*`, `172.16-31.*`, `192.168.*`, `[::1]`, `[fd00::]`). Reject `file://`, `gopher://`, `dict://` schemes. | CWE-918 |
   | XML parser | External entity processing disabled (`disallow-doctype-decl`, `FEATURE_SECURE_PROCESSING`). DTD loading disabled. If the parser accepts user-supplied XML, verify XXE is mitigated. | CWE-611 |
   | Deserialization | No deserialization of untrusted data with native serializers (`pickle`, `ObjectInputStream`, `BinaryFormatter`, `unserialize`, `yaml.load`). Use safe alternatives (`json`, `yaml.safe_load`, allowlisted types). | CWE-502 |
   | Log output | User input in log messages sanitized against log injection (newlines, ANSI escape sequences). No PII, tokens, or credentials written to logs. | CWE-117 |
   | File path | No path traversal via `../` sequences. File access restricted to an expected directory using canonical path comparison. | CWE-22 |
   | Regular expression | No user-controlled regex patterns. If unavoidable, enforce bounded quantifiers and apply timeout. | CWE-1333 |
   | Template engine | User input never passed as template source — only as template data. Sandbox mode enabled where available. | CWE-1336 |

3. **Authentication and session management.** Check:
   - Authentication is enforced on every endpoint that requires it — not assumed from middleware ordering or convention. Verify the middleware, guard, or decorator is explicitly attached.
   - Passwords hashed with a modern KDF (bcrypt, scrypt, argon2) with adequate cost factor. Never MD5, SHA-1, or unsalted SHA-256.
   - Session tokens generated with a CSPRNG, have sufficient entropy (128+ bits), and are invalidated on logout, password change, and privilege escalation.
   - JWT: algorithm explicitly validated server-side (reject `none`, reject algorithm switching). Signature verified before any claim is trusted. Tokens have expiration (`exp`) and audience (`aud`) claims. Secret keys not embedded in source code.
   - OAuth: `state` parameter checked on callback to prevent CSRF. PKCE enforced for public clients. Redirect URI validated with exact match — no substring, no regex. Authorization codes are single-use.
   - Multi-factor authentication (if present) cannot be bypassed by skipping a step in the flow or by calling a downstream endpoint directly.

4. **Access control.** Check:
   - Every data-mutating endpoint enforces authorization — not just authentication. The check verifies the authenticated identity has permission to act on the specific resource (prevents IDOR / BOLA).
   - Object references (IDs in URLs, request bodies, or query parameters) are validated against the authenticated user's permissions. Sequential or guessable IDs do not grant access without an authorization check.
   - Role and permission checks use deny-by-default — access is refused unless an explicit grant exists. No fallthrough to "allow."
   - Privilege escalation paths are guarded: admin-only endpoints verify admin role, not just valid authentication. Feature flags and tenant boundaries are enforced server-side.
   - Bulk and list endpoints filter results by the caller's permissions — no mass data exposure through unfiltered queries.
   - API responses return only the fields the caller is authorized to see. No over-fetching of internal IDs, email addresses, or metadata the client does not need.

5. **Data protection and cryptography.** Check:
   - No secrets (API keys, passwords, tokens, private keys, connection strings) in source code, config files committed to version control, or test fixtures. Secrets come from environment variables, secret managers, or encrypted config.
   - PII and sensitive data are not logged, not included in error messages returned to clients, and not cached in browser-accessible storage (`localStorage`, `sessionStorage`) without justification.
   - Encryption at rest uses AES-256 or equivalent with authenticated modes (GCM, CCM). Not ECB mode, not DES, not RC4, not Blowfish.
   - Encryption in transit enforces TLS 1.2+ with strong cipher suites. No fallback to plaintext. No certificate validation disabled in production code (`InsecureSkipVerify`, `verify=False`, `NODE_TLS_REJECT_UNAUTHORIZED=0`).
   - Random values used for security purposes (tokens, nonces, IVs, salts) come from a CSPRNG — not `Math.random()`, `rand()`, or `random.random()`.
   - Cryptographic keys have adequate length (RSA 2048+, ECDSA 256+, symmetric 128+ bits). No deprecated algorithms (MD5 or SHA-1 for signatures, DSA).
   - Error messages returned to users do not reveal stack traces, internal paths, database table names, or technology versions.

6. **Dependencies and configuration.** Check:
   - New or upgraded dependencies do not have known critical or high CVEs. If a lockfile changed, verify the update is intentional — not a supply chain injection.
   - CORS policy is scoped to specific origins — no wildcard (`*`) on endpoints that handle credentials. `Access-Control-Allow-Credentials: true` must not pair with `Access-Control-Allow-Origin: *`.
   - Security headers are set where applicable: `Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `X-Frame-Options` or CSP `frame-ancestors`.
   - Debug modes, verbose logging, and development-only middleware are not active in production configuration.
   - Rate limiting or throttling is present on authentication endpoints, password reset, and any endpoint that triggers an external side effect (email, SMS, payment).
   - File upload handling restricts allowed MIME types and file sizes. Uploaded files are stored outside the web root with randomized names — never served with user-controlled filenames or paths.

7. **Classify findings.** For each issue found, assign a severity using the same scale as the review verdict:
   - **Blocker** — exploitable vulnerability, missing authentication or authorization, secrets in source, use of broken cryptography. Must be fixed before merge.
   - **Warning** — defense-in-depth gap, missing security header, overly permissive CORS, or configuration that increases risk but is not directly exploitable. Should be addressed.
   - **Note** — suggestion for hardening, informational finding, or a pattern that could become a vulnerability if the code evolves.

## Guardrails

- Never approve code that disables TLS certificate validation, uses `none` JWT algorithm, or deserializes untrusted data with native serializers — these are always blockers regardless of context.
- Never flag a finding without tracing the actual data flow. A SQL query that only uses hardcoded values is not SQL injection. A redirect that only uses server-generated URLs is not an open redirect. Verify that untrusted data actually reaches the sink before reporting.
- Never perform dynamic testing (sending requests, fuzzing, running exploits) from this skill. This is a static code review. If dynamic testing is needed, note it as a recommendation in the findings.
- Hold your initial findings firmly. Do not soften severity on reflection. If you found a vulnerability, report it — do not rationalize it away.
