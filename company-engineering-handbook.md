# Engineering Handbook

## Purpose
This handbook defines **how we design, build, ship, operate, and scale software**. It is a binding reference for all engineers and overrides personal preference. The goal is long-term reliability, security, and team scalability.

---

## How to Use This Handbook
- **Mandatory** for all engineers
- Enforced via tooling, code review, and CI/CD
- Deviations require a written ADR

---

# Part I – Engineering Principles

## 1. Core Values
- Readability over cleverness
- Security by default
- Explicit over implicit
- Blameless culture
- Continuous improvement

## 2. Non‑Negotiables
- No tests → no merge
- No review → no merge
- Broken main = incident
- Production safety over delivery speed

---

# Part II – Coding Standards

## 3. Naming Conventions

### Files & Folders
- kebab-case for files
- plural folders, singular entities

### Classes
- PascalCase

### Methods
- Verb-first
- One responsibility

### Variables
- Descriptive, typed, intention‑revealing

---

## 4. API Design Standards

- RESTful, resource‑based
- Versioned (`/api/v1`)
- Correct HTTP status codes
- Standardized response envelope

---

## 5. Backend Standards (Node.js / TypeScript)

- TypeScript only (`strict: true`)
- Controller → Service → Repository
- Centralized error handling
- Input validation mandatory

---

## 6. Frontend Standards (Angular)

- Feature‑based structure
- Standalone components preferred
- `OnPush` change detection
- No business logic in templates

---

## 7. Database Standards (PostgreSQL)

- snake_case schema
- Explicit constraints
- Indexed query paths
- No `SELECT *`

---

# Part III – Platform & Operations

## 8. Containerization (Docker)

- Multi‑stage builds
- Non‑root containers
- Small base images

---

## 9. CI/CD

- Deterministic pipelines
- Mandatory lint, test, scan, build
- Immutable artifacts

---

## 10. Nginx & Edge Security

- HTTPS only
- Security headers
- Reverse proxy only

---

# Part IV – Production Excellence

## 11. Observability

- Structured JSON logs
- Metrics: latency, error rate, saturation
- Correlation IDs required

---

## 12. Incident Management

### Severity Levels
- SEV‑1: Outage / breach
- SEV‑2: Partial outage
- SEV‑3: Degradation

### Incident Process
1. Acknowledge
2. Mitigate
3. Communicate
4. Resolve
5. Postmortem

---

# Part V – Team & Culture

## 13. Ownership Model

- Every service has an owner
- Ownership includes production support

---

## 14. Pull Requests

- ≤ 400 lines preferred
- Senior review required
- 24h review SLA

---

## 15. Decision Making (ADR)

Required for:
- Architecture changes
- Breaking changes
- Security decisions

---

## 16. Technical Debt Policy

- Visible
- Budgeted (20–30%)
- Continuously reduced

---

## 17. Engineering Culture

- Psychological safety
- Blameless postmortems
- No hero culture
- Respectful communication

---

# Appendix

- Appendix A: Rule‑by‑Rule Examples
- Appendix B: Language‑Level Rules
- Appendix D: DevOps & Incident Rules
- Appendix E: Team Scaling & Culture

---

# Appendix A – Coding Standards (Deep DO / DON’T)

## A1. File & Folder Naming

### DO ✅

| Rule                           | Example                                           |
| ------------------------------ | ------------------------------------------------- |
| Use lowercase + kebab-case     | `user-profile.component.ts`                       |
| One file = one responsibility  | `auth.service.ts`                                 |
| Suffix files by type           | `.controller.ts`, `.service.ts`, `.repository.ts` |
| Match folder name with feature | `/users/user.service.ts`                          |

### DON’T ❌

| Anti-Pattern              | Why                                 |
| ------------------------- | ----------------------------------- |
| `UserService.ts`          | Breaks cross-platform compatibility |
| `utils.ts` dumping ground | Encourages god objects              |
| Mixed naming styles       | Cognitive load                      |
| Deep nesting > 5 levels   | Hard to navigate                    |

---

## A2. Classes, Methods & Functions

### DO ✅

| Rule                          | Example              |
| ----------------------------- | -------------------- |
| One responsibility per method | `validatePassword()` |
| Verb-based method names       | `createUser()`       |
| Max 30–40 lines per method    | Forces clarity       |
| Early return over nested ifs  | Improves readability |

```ts
// GOOD
validateEmail(email: string): boolean {
  if (!email) return false;
  return EMAIL_REGEX.test(email);
}
```

### DON’T ❌

```ts
// BAD
processUserData(user) {
  // validation + DB + API calls + logging
}
```

Reasons:

* Impossible to test
* Hard to reuse
* High bug risk

---

## A3. Variables & Constants

### DO ✅

| Rule                        | Example             |
| --------------------------- | ------------------- |
| Use meaningful names        | `isEmailVerified`   |
| Prefer const over let       | Immutability        |
| Use enums for states        | `UserStatus.ACTIVE` |
| Prefix booleans with is/has | `hasPermission`     |

### DON’T ❌

| Example                  | Why                    |
| ------------------------ | ---------------------- |
| `data`, `tmp`, `value`   | No meaning             |
| Reassign function params | Side effects           |
| Magic numbers            | Breaks maintainability |

---

## A4. Comments & Documentation

### DO ✅

| Rule                      | Example         |
| ------------------------- | --------------- |
| Comment WHY, not WHAT     | Business intent |
| Use JSDoc for public APIs | Tooling support |
| Keep comments updated     | Trustworthiness |

```ts
/**
 * Locks user account after N failed attempts
 * Required by security policy SEC-04
 */
```

### DON’T ❌

```ts
// increment i by 1
i++;
```

If code needs explaining, **refactor it**.

---

## A5. Error Handling & Logging

### DO ✅

| Rule                        | Example                 |
| --------------------------- | ----------------------- |
| Throw typed errors          | `UnauthorizedError`     |
| Log structured JSON         | `requestId`, `userId`   |
| Handle errors at boundaries | Controller / middleware |

### DON’T ❌

| Anti-Pattern            | Why                |
| ----------------------- | ------------------ |
| `catch (e) {}`          | Swallows bugs      |
| `console.log()` in prod | No observability   |
| Logging secrets         | Security violation |

---

# Appendix B – API & Architecture Rules

## B1. API Endpoints

### DO ✅

| Rule                      | Example                 |
| ------------------------- | ----------------------- |
| RESTful nouns             | `/users/{id}`           |
| Version APIs              | `/api/v1`               |
| Correct HTTP verbs        | GET / POST / PUT        |
| Consistent response shape | `{ data, error, meta }` |

### DON’T ❌

| Example                         | Why              |
| ------------------------------- | ---------------- |
| `/getUserNow`                   | RPC anti-pattern |
| Mixing verbs & nouns            | Inconsistent     |
| Breaking backward compatibility | Client outages   |

---

## B2. Controllers, Services, Repositories

### DO ✅

| Layer      | Responsibility         |
| ---------- | ---------------------- |
| Controller | HTTP, auth, validation |
| Service    | Business rules         |
| Repository | DB access only         |

### DON’T ❌

* DB queries in controllers
* HTTP objects in services
* Business logic in repositories

---

# Appendix D – DevOps, CI/CD & Incident Rules

## D1. CI/CD Pipelines

### DO ✅

| Stage         | Required              |
| ------------- | --------------------- |
| Lint          | ESLint, Prettier      |
| Test          | Unit + integration    |
| Security scan | SAST, dependency scan |
| Build         | Immutable artifacts   |
| Deploy        | Automated only        |

### DON’T ❌

| Anti-Pattern        | Why                |
| ------------------- | ------------------ |
| Skipping tests      | Hidden regressions |
| Manual prod deploys | Human error        |
| Hotfix without PR   | No audit trail     |

---

## D2. Docker & Infrastructure

### DO ✅

| Rule               | Example        |
| ------------------ | -------------- |
| Small base images  | `node:alpine`  |
| Non-root user      | Security       |
| Multi-stage builds | Smaller images |
| Explicit ports     | Documentation  |

### DON’T ❌

* `latest` tags
* Secrets in images
* Snowflake servers

---

## D3. Incident Management

### DO ✅

| Rule                 | Description      |
| -------------------- | ---------------- |
| Severity levels      | SEV-1 to SEV-4   |
| Incident commander   | Single owner     |
| Status updates       | Every 30–60 min  |
| Blameless postmortem | Learning culture |

### DON’T ❌

* Blame individuals
* Fix without RCA
* Ignore near-misses

---

# Appendix E – Team Scaling, Ownership & Culture

## E1. Ownership

### DO ✅

| Rule                     | Meaning         |
| ------------------------ | --------------- |
| Named owner per service  | Accountability  |
| On-call rotation         | Shared load     |
| You build it, you run it | Quality mindset |

### DON’T ❌

* “Not my service”
* Throwing issues to ops
* Unowned codebases

---

## E2. Pull Requests & Reviews

### DO ✅

| Rule                   | Example             |
| ---------------------- | ------------------- |
| Small PRs              | < 500 LOC           |
| Context in description | Why + what          |
| Review for design      | Not just syntax     |
| Block unsafe code      | Even under pressure |

### DON’T ❌

* Rubber-stamp approvals
* Massive PRs
* Reviewing your own code

---

## E3. Engineering Culture

### DO ✅

* Psychological safety
* Ask questions
* Share knowledge
* Improve docs continuously

### DON’T ❌

* Hero culture
* Fear-based reviews
* Silent disagreement

---
