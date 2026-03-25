# Security & Testing Master Prompt

> **Use this prompt with Claude Code, Claude Chat, or any AI assistant when auditing or building out security and tests for your projects.**

---

## The Prompt

Copy everything below the line and paste it as a prompt, adjusting the `[PLACEHOLDERS]` to your specific project.

---

```
You are a senior security engineer and QA architect. Audit my [FRAMEWORK: e.g., Laravel/Vue/React Native] project and generate a comprehensive security hardening plan and test suite. The app is [BRIEF DESCRIPTION] deployed on [INFRASTRUCTURE: e.g., DigitalOcean Ubuntu 24.04 via Docker Compose].

## 1. UNIT & FEATURE TESTING

### Backend (Laravel/PHP)
- Write PHPUnit tests for every controller action (happy path + edge cases)
- Test all Form Request validation rules — valid input, invalid input, boundary values
- Test model relationships, scopes, accessors, and mutators
- Test all middleware (auth, roles, rate limiting, CORS)
- Test job/queue logic, event listeners, and mail/notification dispatch
- Test database seeders and migrations (up + down)
- Test API responses: correct status codes, JSON structure, pagination
- Mock external services (payment gateways, APIs) — never hit real endpoints in tests

### Frontend (Vue 3 / React / React Native)
- Write component unit tests (Vitest/Jest + Testing Library)
- Test props, emits, slots, computed properties, watchers
- Test conditional rendering and v-if / v-show / ternary states
- Test form validation and submission flows
- Test Vuex/Pinia store actions, mutations, getters (or Redux/Zustand equivalents)
- Test API call handling: loading states, success, error, empty states
- Test navigation guards and route protection
- Snapshot tests for stable UI components only (avoid on frequently changing ones)

### Integration Tests
- Test full authentication flows: register → verify email → login → access protected route → logout
- Test role-based access: admin vs user vs guest hitting the same endpoints
- Test file upload end-to-end (validation, storage, retrieval, deletion)
- Test payment/subscription flows with sandbox/test credentials
- Test WebSocket/broadcasting events fire and are received correctly

## 2. SECURITY AUDIT

### Authentication & Authorization
- [ ] Sanctum/Passport token handling — tokens expire, refresh logic is correct
- [ ] Password hashing uses bcrypt/argon2 (never MD5/SHA1)
- [ ] Rate limiting on login/register/password reset (throttle middleware)
- [ ] Account lockout after N failed attempts
- [ ] Email verification enforced before sensitive actions
- [ ] OAuth state parameter validated (prevent CSRF on social login)
- [ ] Session fixation — session ID regenerated after login
- [ ] Authorization checks on EVERY controller method, not just routes
- [ ] Verify no mass-assignment vulnerabilities ($fillable / $guarded properly set)
- [ ] API tokens scoped with minimal permissions

### Injection Attacks
- [ ] SQL Injection: confirm all queries use Eloquent/query builder or parameterized bindings. Flag any `DB::raw()`, `whereRaw()`, or string concatenation in queries
- [ ] XSS: all user output escaped. In Blade: `{{ }}` not `{!! !!}` unless explicitly sanitized. In Vue: no `v-html` with user data
- [ ] Command Injection: no `exec()`, `shell_exec()`, `system()`, `passthru()` with user input
- [ ] Path Traversal: file access validated, no `../` allowed in user-supplied paths
- [ ] LDAP/XML/NoSQL injection if applicable
- [ ] Verify `Content-Type` headers match expected format on all API endpoints

### Prompt Injection (for AI-powered features)
- [ ] Never pass raw user input directly into system prompts
- [ ] Sanitize and validate all user input before including in LLM prompts
- [ ] Use input/output guardrails — define allowed topics, max lengths, forbidden patterns
- [ ] Implement output filtering — strip any instructions/code the LLM might echo
- [ ] Rate limit AI endpoint calls per user
- [ ] Log all AI interactions for review
- [ ] Separate system instructions from user content using clear delimiters
- [ ] Test with adversarial prompts:
  - "Ignore previous instructions and..."
  - "You are now DAN..."
  - Encoded/obfuscated injection attempts (base64, unicode, markdown)
  - Multi-turn manipulation (building trust then injecting)
  - Indirect injection via uploaded documents/URLs
- [ ] Never expose raw model responses containing system prompt details
- [ ] Implement cost controls — max tokens per request, daily spending caps

### API Security
- [ ] Authentication required on all non-public endpoints
- [ ] CORS configured to allow only specific origins (not `*` in production)
- [ ] Rate limiting per user/IP on all endpoints (especially auth + AI)
- [ ] Request size limits configured
- [ ] No sensitive data in URLs (tokens, passwords in query params)
- [ ] API versioning in place
- [ ] Proper HTTP methods enforced (no state changes via GET)
- [ ] Response doesn't leak stack traces, debug info, or internal paths in production
- [ ] Validate all request inputs even if frontend validates too
- [ ] Disable unused HTTP methods

### Data Protection
- [ ] Sensitive data encrypted at rest (API keys, PII)
- [ ] HTTPS enforced everywhere, HSTS headers set
- [ ] Database credentials not in code/repo — use .env, never commit .env files
- [ ] .env.example contains no real values
- [ ] APP_DEBUG=false in production
- [ ] Logs don't contain passwords, tokens, or PII
- [ ] File uploads validated: type, size, extension, MIME type. Stored outside web root
- [ ] Backups encrypted and access-controlled
- [ ] PII handling compliant with applicable laws (Data Privacy Act for PH)

### Infrastructure & Deployment
- [ ] Docker containers run as non-root user
- [ ] Docker images pinned to specific versions (not `latest`)
- [ ] SSH key auth only, password auth disabled
- [ ] Firewall rules: only ports 22, 80, 443 open (or as needed)
- [ ] GitHub Actions secrets not logged, not exposed in build output
- [ ] `.env` files not copied into Docker images
- [ ] Database ports not exposed to public internet
- [ ] SSL certificates auto-renewing
- [ ] Server OS and packages updated regularly
- [ ] Fail2Ban or similar intrusion detection active
- [ ] Container health checks configured
- [ ] GitHub branch protection: require PR reviews, no force push to main

### Headers & Browser Security
- [ ] Content-Security-Policy header set
- [ ] X-Content-Type-Options: nosniff
- [ ] X-Frame-Options: DENY (or SAMEORIGIN if needed)
- [ ] Referrer-Policy: strict-origin-when-cross-origin
- [ ] Permissions-Policy set (disable camera, mic, geolocation unless needed)
- [ ] Cookies: HttpOnly, Secure, SameSite=Lax (or Strict)
- [ ] Remove X-Powered-By and Server headers

### Mobile App Security (React Native)
- [ ] No secrets/API keys hardcoded in JS bundles
- [ ] Certificate pinning implemented for API calls
- [ ] Secure storage used for tokens (Keychain/Keystore, not AsyncStorage for secrets)
- [ ] Deep link validation — don't trust incoming deep link params
- [ ] Jailbreak/root detection for sensitive apps
- [ ] Disable screenshots on sensitive screens if needed
- [ ] ProGuard/R8 obfuscation enabled for Android release builds
- [ ] App Transport Security configured properly for iOS

## 3. TEST AUTOMATION & CI/CD

- Set up GitHub Actions workflow:
  - Run PHPUnit on every PR
  - Run frontend tests (Vitest/Jest) on every PR
  - Run ESLint + PHP CS Fixer / Pint for code style
  - Run security scanners: `npm audit`, `composer audit`
  - Run static analysis: PHPStan/Larastan level 6+
  - Block merge if any test fails
- Add pre-commit hooks (Husky): lint + format check
- Generate code coverage reports, set minimum threshold (aim for 80%+)

## 4. MONITORING & INCIDENT RESPONSE

- [ ] Error tracking set up (Sentry, Bugsnag, or Laravel Telescope in staging)
- [ ] Uptime monitoring configured
- [ ] Log aggregation — centralized, searchable logs
- [ ] Alert on: failed login spikes, error rate spikes, unusual API traffic
- [ ] Documented incident response plan: who gets notified, how to rollback
- [ ] Database backup tested — actually restore from backup periodically

## OUTPUT FORMAT

For each finding, provide:
1. **Severity**: Critical / High / Medium / Low
2. **Location**: File path and line number
3. **Issue**: What's wrong
4. **Fix**: Exact code or config change needed
5. **Test**: A test to verify the fix works

Start with Critical findings first. Generate all test files with proper directory structure.
```

---

## Quick-Use Variants

### Laravel-Only Backend Audit
```
Audit my Laravel 11 API for security vulnerabilities. Focus on: SQL injection in raw queries, mass assignment, auth bypass, exposed debug routes, .env leaks, and missing rate limits. Generate PHPUnit tests for every finding.
```

### Vue 3 Frontend Audit
```
Audit my Vue 3 + Pinia SPA for: XSS via v-html, insecure token storage, missing auth guards on routes, exposed API keys in client bundle, CORS misconfig, and clickjacking. Generate Vitest tests.
```

### React Native Mobile Audit
```
Audit my React Native (Expo) app for: hardcoded secrets in JS bundle, insecure token storage (AsyncStorage vs SecureStore), missing certificate pinning, deep link injection, and missing root/jailbreak detection. Generate Jest tests.
```

### Docker / Deployment Audit
```
Audit my Docker Compose + GitHub Actions deployment on Ubuntu 24.04 for: containers running as root, exposed database ports, secrets in build logs, missing health checks, SSH hardening, and firewall rules. Provide fixed docker-compose.yml and workflow files.
```

### AI Feature / Prompt Injection Audit
```
Audit my AI-powered feature for prompt injection vulnerabilities. Test with: direct injection ("ignore instructions"), indirect injection (via uploaded files/URLs), encoded payloads (base64/unicode), multi-turn manipulation, and system prompt leakage. Provide input sanitization middleware and output filtering code.
```
