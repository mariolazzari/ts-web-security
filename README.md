# Web security in TypeScript

## OWASP

### Setup

```sh
npm install
npm run db:reset
npm start
npm run dev
```

### Security consequences

Sometimes companies get away with sloppy security... and sometimes they don't. When things go wrong, the consequences extend far beyond the engineering resources spent to fix the vulnerable code. For example:

- A SQL injection exposes 10,000 email addresses and order histories.
  - Your engineering team investigates the breach and closes the vulnerability.
  - Your legal and privacy teams determine whether affected users or regulators must be notified.
  - Your customer support team handles a flood of worried users.
  - Your company loses customer trust and future sales.

- An XSS bug lets an attacker run code in users' browsers. The attacker may steal sensitive data or act as those users.
  - Your security team investigates which accounts were affected and cuts off the attacker's access.
  - Your support team helps users restore control of their accounts.

### OWASP Top 10

- A01: Broken Access Control – Users can do things they shouldn't be allowed to do.
- A02: Security Misconfiguration – The system is deployed or configured in an unsafe way.
- A03: Software Supply Chain Failures – Vulnerabilities enter through outdated or compromised third-party libraries or build tools.
- A04: Cryptographic Failures – Sensitive data is exposed because cryptographic protections are missing or misused.
- A05: Injection – Untrusted input is treated as executable instructions.
- A06: Insecure Design – The system's design lacks the controls needed to resist attacks.
- A07: Authentication Failures – The system doesn't reliably verify who a user is.
- A08: Software or Data Integrity Failures – The application trusts software or data without verifying its integrity.
- A09: Security Logging and Alerting Failures – Attacks happen without triggering the records or alerts needed for a response.
- A10: Mishandling of Exceptional Conditions – Unexpected conditions expose details, bypass controls, or leave the system in a bad state.

#### NoSniff

Missing browser security headers are a concrete example of A02: Security Misconfiguration. One of the simplest headers to add is X-Content-Type-Options: nosniff.

This header tells browsers to respect a response's declared Content-Type instead of guessing from its contents.

This helps prevent security vulnerabilities related to MIME sniffing. For example, without nosniff, a browser might treat attacker-controlled content as executable code despite its declared type!

## Authentication

### Stateless vs. Stateful Authentication

#### Stateless Authentication

Every request carries a credential that the server can validate without looking up a server-side session record. Typically:

- After successful login, the server issues a signed token to the client.
- The client sends the signed token with each request.
- The server accepts or rejects the request based on the token without loading a session record.

#### Stateful Authentication

The server remembers you. Typically:

- After successful login, the server creates a session record and sends a secret session identifier to the client
- The client sends the secret session identifier with future requests.
- The server accepts or rejects the request after checking if the session identifier maps to a valid session record (usually in a database or in-memory store).

### Sessions

An authenticated session is created when a user successfully logs in. It's a record stored on the server that says "When a user presents this session ID, let them access this account while the session is valid."

In a typical browser flow, the client stores and sends that ID in a cookie, and the server re-checks it against the session in the database on every request.

#### Session ID

A session ID is a reference to server-side data (often a row in a sessions table) that stores:

- A unique ID for the session (id, which is secret and given to the client)
- Which user authenticated (user_id)
- When the session expires (expires_at)
- Whether the session has been revoked (revoked_at, can be NULL if the session is still valid)

### Cookies

From the server's point of view, cookies are convenient. From an attacker's point of view, session cookies are exploitable.

- If an attacker steals a bearer session cookie and reuses it, the server (usually) can't tell those requests apart from the user's. Some common mistakes are:
- If a session cookie is readable through document.cookie, injected JavaScript can steal it.
- If a session cookie is sent over plain HTTP, an attacker who can observe the connection can capture it.
- If a cookie has an unnecessarily broad domain or long expiration, an attacker has more opportunities to steal and reuse it.
- If the browser attaches a cookie to a malicious cross-site request, cross-site request forgery (CSRF) is possible... which we'll talk about later.

#### Cookie Security Flags

Modern browsers give us tools to limit how cookies behave. The most important ones are:

- HttpOnly (boolean): Prevents JavaScript from reading the cookie through document.cookie.
- Secure (boolean): Sends the cookie only over HTTPS (and localhost, for development).
- SameSite (Strict, Lax, None): Restricts when the browser sends the cookie with cross-site requests. Strict is the most restrictive. Lax also permits top-level cross-site navigations using safe methods, which many login flows need.
