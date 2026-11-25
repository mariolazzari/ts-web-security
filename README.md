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
