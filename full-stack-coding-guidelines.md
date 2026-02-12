# Full-Stack Coding Guidelines

**Audience:** Large engineering teams (Junior → Staff/Principal)

**Objective:** Create **predictable, readable, secure, testable, and scalable** systems that survive long-term team growth.

---

## 1. Core Engineering Principles (Non-Negotiable)

### Must Follow
- **Readability > Cleverness**
- **Explicit > Implicit**
- **Fail Fast, Fail Loud (internally)**
- **Security by Default**
- **Consistency beats personal preference**

### Must NOT
- ❌ Rely on tribal knowledge
- ❌ Write code only the author understands
- ❌ Optimize prematurely

---

## 2. Naming Conventions (VERY IMPORTANT)

### Files & Folders

#### Do
- kebab-case for files
- plural folders, singular entities

```
user.controller.ts
user.service.ts
user.repository.ts
user.routes.ts
email-verification.service.ts
```

#### Don’t
```
UserController.ts
userservice.ts
userServiceNEW.ts
```

---

### Classes & Interfaces

#### Do
```ts
class UserService {}
interface UserRepository {}
```

#### Don’t
```ts
class userservice {}
class user_service {}
```

---

### Methods / Functions

#### Rules
- Verb-first
- One responsibility only
- Max ~20–30 lines

#### Do
```ts
createUser()
validatePassword()
findUserByEmail()
```

#### Don’t
```ts
doEverything()
handle()
process()
```

---

### Variables

#### Do
```ts
const isEmailVerified: boolean;
const accessTokenExpiresAt: Date;
```

#### Don’t
```ts
const flag;
const data;
let tmp;
```

---

## 3. API Design Standards (REST)

### Endpoint Naming

#### Do
```
GET    /api/v1/users
GET    /api/v1/users/{id}
POST   /api/v1/users
PUT    /api/v1/users/{id}
DELETE /api/v1/users/{id}
```

#### Don’t
```
GET /getUsers
POST /createUserNow
GET /user?id=1
```

---

### HTTP Status Codes

#### Must Use Correctly
- 200 OK
- 201 Created
- 400 Bad Request
- 401 Unauthorized
- 403 Forbidden
- 404 Not Found
- 409 Conflict
- 422 Validation Error
- 500 Internal Server Error

---

### API Response Shape (Standardized)

```json
{
  "success": true,
  "data": {},
  "error": null,
  "meta": {
    "requestId": "uuid"
  }
}
```

❌ Never return raw DB errors

---

## 4. Node.js / TypeScript Deep Rules

### Language Rules

#### Must
- TypeScript only
- `strict: true`
- No `any`
- Prefer `readonly`

#### Forbidden
```ts
// ❌
function handler(req, res) {}
let data: any;
```

---

### Controllers

#### Responsibilities
- Parse input
- Call service
- Return response

#### Must NOT
- Contain business logic
- Access database directly

```ts
// GOOD
async createUser(req: Request, res: Response) {
  const dto = CreateUserSchema.parse(req.body);
  const user = await userService.create(dto);
  res.status(201).json(user);
}
```

---

### Services

- Business logic only
- No HTTP concerns

---

### Repositories

- DB access only
- No business logic
- Always return domain objects

---

### Error Handling

#### Must
- Centralized error handler
- Custom error classes

```ts
class ValidationError extends AppError {}
```

❌ No try/catch per route unless justified

---

## 5. Comments & Documentation

### When to Comment
- WHY > WHAT
- Non-obvious logic
- Security decisions

```ts
// Password comparison is timing-safe to prevent side-channel attacks
```

### When NOT to Comment
```ts
// increments i by 1
 i++;
```

---

## 6. Angular Deep Standards

### Component Rules

- One component = one responsibility
- < 300 lines per component
- No logic in HTML

---

### Services

- Stateless
- No component state

---

### RxJS

#### Must
- Unsubscribe (takeUntil)
- Prefer async pipe

#### Forbidden
```ts
.subscribe(x => this.data = x); // ❌ without cleanup
```

---

### State Management

- NgRx / Signals for shared state
- No global mutable services

---

## 7. Database (PostgreSQL) Deep Rules

### Schema

- No polymorphic tables
- No JSON blobs for relational data
- Explicit constraints

---

### Queries

#### Must
- EXPLAIN ANALYZE for slow queries
- Pagination required

#### Forbidden
```sql
SELECT * FROM large_table;
```

---

### Migrations

- Forward-only
- No manual DB edits

---

## 8. Docker & Containers

### Image Rules

- One process per container
- Health checks required

```dockerfile
HEALTHCHECK CMD node healthcheck.js
```

---

### Secrets

- Env vars only
- Never bake secrets

---

## 9. CI/CD Deep Discipline

### Mandatory Gates
- Lint
- Unit tests
- Security scan
- Build

❌ Any failure blocks merge

---

### Environments

- dev
- staging
- production

Config must never differ in code, only env

---

## 10. Nginx Deep Rules

### Security

- HSTS enabled
- CSP headers
- Request size limits

```nginx
client_max_body_size 2m;
```

---

## 11. Logging & Monitoring

### Logs

- JSON only
- Correlation IDs mandatory

❌ No PII in logs

---

## 12. Testing Standards

### Required Coverage
- Services: Unit tests
- APIs: Integration tests
- UI: Component + e2e

❌ No test → No merge

---

## 13. Code Review Checklist (Mandatory)

- Naming follows convention
- No security leaks
- Tests included
- No duplicated logic
- Error handling correct

---

## 14. Team Enforcement Rules

- CI is law
- Reviews are mandatory
- Broken main = incident
- Security bugs = highest priority

---

**This guideline is enforceable via tooling and is not optional.**

---

# Appendix A: Rule-by-Rule Examples (Good vs Bad)

## A1. File & Folder Naming Examples

### Backend (Node.js)

✅ GOOD
```
user.controller.ts
user.service.ts
user.repository.ts
password-reset.service.ts
```

❌ BAD
```
UserController.ts
userService.ts
user_repo.ts
passwordResetServiceNEW.ts
```

Reason: Consistent kebab-case avoids OS conflicts and improves discoverability.

---

## A2. Method Design Examples

### Single Responsibility

✅ GOOD
```ts
async hashPassword(password: string): Promise<string> {
  return argon2.hash(password);
}
```

❌ BAD
```ts
async handleUser(password: string, email: string) {
  const hash = argon2.hash(password);
  db.save({ email, password: hash });
  sendEmail(email);
}
```

Rule violated: Multiple responsibilities, untestable.

---

## A3. Variable Naming Examples

✅ GOOD
```ts
const isAccountLocked: boolean;
const failedLoginAttempts: number;
const tokenExpiresAt: Date;
```

❌ BAD
```ts
const flag;
const count;
const exp;
```

---

## A4. API Endpoint Examples

### Resource-Based Design

✅ GOOD
```
POST /api/v1/auth/login
POST /api/v1/auth/logout
POST /api/v1/auth/refresh-token
```

❌ BAD
```
POST /loginUser
GET /doLogout
POST /refresh
```

---

## A5. Controller vs Service Boundary

### Controller

✅ GOOD
```ts
async login(req: Request, res: Response) {
  const dto = LoginSchema.parse(req.body);
  const result = await authService.login(dto);
  res.status(200).json(result);
}
```

❌ BAD
```ts
async login(req, res) {
  const user = await User.findOne({ email: req.body.email });
  if (!user) throw 'error';
}
```

---

## A6. Error Handling Examples

✅ GOOD
```ts
throw new ConflictError('Email already exists');
```

❌ BAD
```ts
throw 'User error';
res.send(err);
```

---

## A7. Commenting Examples

✅ GOOD
```ts
// Using constant-time comparison to prevent timing attacks
comparePasswords(input, storedHash);
```

❌ BAD
```ts
// call function
comparePasswords(input, storedHash);
```

---

## A8. Angular Component Examples

### Change Detection

✅ GOOD
```ts
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
```

❌ BAD
```ts
@Component({}) // default change detection
```

---

## A9. RxJS Subscription Examples

✅ GOOD
```ts
this.user$ = this.userService.getUser();
```

❌ BAD
```ts
this.userService.getUser().subscribe(u => this.user = u);
```

---

## A10. SQL Query Examples

✅ GOOD
```sql
SELECT id, email FROM users WHERE email = $1;
```

❌ BAD
```sql
SELECT * FROM users WHERE email = 'test@test.com';
```

---

## A11. Docker Examples

✅ GOOD
```dockerfile
FROM node:20-alpine
USER node
```

❌ BAD
```dockerfile
FROM node:latest
USER root
```

---

## A12. Logging Examples

✅ GOOD
```ts
logger.info({ userId, requestId }, 'Login successful');
```

❌ BAD
```ts
console.log(user);
```

---

## A13. Security Examples

### Password Storage

✅ GOOD
```ts
await argon2.hash(password);
```

❌ BAD
```ts
crypto.createHash('md5').update(password).digest('hex');
```

---

## A14. Testing Examples

✅ GOOD
```ts
it('should reject invalid email', () => {
  expect(() => validateEmail('bad')).toThrow();
});
```

❌ BAD
```ts
it('works', () => {});
```

---

## A15. PR Review Red Flags

🚩 Any of the following requires rejection:
- Unclear naming
- Missing tests
- Business logic in controllers/components
- Console logs
- Hard-coded values

---

**Appendix A is mandatory reading for all engineers.**

---

# Appendix B: Language-Level & Architectural Rules (Deep)

## B1. TypeScript Language Rules

### Types

✅ GOOD
```ts
type UserId = string & { readonly brand: unique symbol };
```

❌ BAD
```ts
type UserId = string;
```

Rule: Use branded/value types for identifiers where feasible.

---

### Enums

✅ GOOD
```ts
enum UserStatus {
  ACTIVE = 'ACTIVE',
  SUSPENDED = 'SUSPENDED'
}
```

❌ BAD
```ts
enum UserStatus { Active, Suspended }
```

Rule: Never rely on numeric enums.

---

### DTOs vs Domain Models

✅ GOOD
```ts
interface CreateUserDto {
  email: string;
  password: string;
}
```

❌ BAD
```ts
class User extends BaseEntity {}
```

Rule: Never expose domain models directly to APIs.

---

### Immutability

✅ GOOD
```ts
readonly roles: readonly Role[];
```

❌ BAD
```ts
roles.push(role);
```

---

## B2. Angular Deep Rules

### Inputs & Outputs

✅ GOOD
```ts
@Input({ required: true }) user!: User;
@Output() saved = new EventEmitter<User>();
```

❌ BAD
```ts
@Input() data: any;
```

---

### Template Rules

✅ GOOD
```html
<button (click)="onSave()">Save</button>
```

❌ BAD
```html
<button (click)="userService.save(user)">Save</button>
```

Rule: Templates never call services.

---

### Signals vs RxJS

- Signals: local UI state
- RxJS: async streams, APIs

❌ Mixing both without reason is forbidden

---

## B3. Backend Architecture Rules

### Layer Boundaries

- Controller → Service → Repository → DB
- Direction is one-way only

❌ Repository calling Service is forbidden

---

### Dependency Injection

✅ GOOD
```ts
constructor(private readonly userRepo: UserRepository) {}
```

❌ BAD
```ts
const repo = new UserRepository();
```

---

## B4. SQL & Database Performance Rules

### Indexing

✅ GOOD
```sql
CREATE INDEX idx_users_email ON users(email);
```

❌ BAD
```sql
-- No index on frequently queried column
```

Rule: Any WHERE/JOIN column must be indexed.

---

### Pagination

✅ GOOD
```sql
LIMIT 20 OFFSET 0;
```

❌ BAD
```sql
SELECT * FROM logs;
```

---

## B5. Security Rules (OWASP-Aligned)

### Input Validation

✅ GOOD
```ts
LoginSchema.parse(req.body);
```

❌ BAD
```ts
req.body.email
```

---

### Authorization

✅ GOOD
```ts
if (!currentUser.can('DELETE_USER')) throw new ForbiddenError();
```

❌ BAD
```ts
if (isAdmin) {}
```

---

## B6. Configuration Rules

### Environment Variables

✅ GOOD
```ts
process.env.DB_HOST ?? throw new Error('Missing DB_HOST');
```

❌ BAD
```ts
const dbHost = 'localhost';
```

---

## B7. Testing Discipline

### Unit Tests

- No mocks of value objects
- Mock external services only

---

### Integration Tests

- Real DB (containerized)
- No shared state

---

## B8. Performance Rules

### Backend

- Avoid N+1 queries
- Cache read-heavy endpoints

---

### Frontend

- Lazy-load feature modules
- Avoid unnecessary change detection

---

## B9. Code Smells (Immediate Refactor Required)

🚩 God classes
🚩 Methods > 50 lines
🚩 Boolean flags changing behavior
🚩 Deep nesting (>3 levels)

---

## B10. Enforcement

Violations result in:
- PR rejection
- Mandatory refactor
- Tech-debt ticket

---

**Appendix B is strictly enforced across all projects.**

---

# Appendix D: DevOps, Operations & Incident Rules

## D1. Environment Strategy

### Environments (Mandatory)

- local
- dev
- staging
- production

Rules:
- Each environment is isolated
- No shared databases
- No cross-environment credentials

❌ Direct production testing is forbidden

---

## D2. Configuration Management

### Rules

- Configuration via environment variables only
- Same artifact deployed to all environments
- Feature differences handled by config, not code

✅ GOOD
```bash
DB_HOST=prod-db.internal
```

❌ BAD
```ts
if (env === 'prod') {
  enableSecurity();
}
```

---

## D3. Secrets Management

### Mandatory

- Secrets stored in vault / secret manager
- Rotatable without redeploy
- Never logged

❌ Forbidden
- Secrets in repo
- Secrets in Docker images
- Secrets in CI logs

---

## D4. CI/CD Pipeline Rules (Strict)

### Required Stages

1. Static analysis (lint)
2. Unit tests
3. Integration tests
4. Security scan (SAST + dependency)
5. Build
6. Deploy

❌ Any skipped stage blocks merge

---

### Pipeline Discipline

- Pipelines must be deterministic
- No flaky tests tolerated
- Builds are immutable

---

## D5. Deployment Strategy

### Allowed

- Blue/Green
- Rolling
- Canary (preferred for critical services)

### Forbidden

- In-place mutable servers
- Manual prod deployments

---

## D6. Database Deployment Rules

### Migrations

- Automatic via pipeline
- Forward-only
- One migration per change

❌ Hot-fixing prod DB manually is forbidden

---

## D7. Monitoring & Alerting

### Mandatory Metrics

- Latency (p95, p99)
- Error rate
- Throughput
- Saturation (CPU, memory)

---

### Alerts

- Alerts must be actionable
- No alert without owner

❌ Pager spam is an incident itself

---

## D8. Logging in Production

### Rules

- Structured JSON logs only
- Correlation / request ID required
- Log levels enforced

❌ No debug logs in prod

---

## D9. Incident Severity Levels

### SEV-1
- Production outage
- Security breach

### SEV-2
- Partial outage
- Data inconsistency

### SEV-3
- Degraded performance

---

## D10. Incident Response Process

### Mandatory Steps

1. Acknowledge
2. Mitigate
3. Communicate
4. Resolve
5. Postmortem

---

## D11. Communication Rules During Incident

- Single incident commander
- Updates every 30 minutes
- Written timeline maintained

❌ Multiple people changing prod simultaneously is forbidden

---

## D12. Rollback Rules

- Rollback must be possible within minutes
- Previous artifact always available

❌ Hotfixing live containers is forbidden

---

## D13. Postmortem Rules (Blameless)

### Required Sections

- What happened
- Impact
- Root cause
- Detection gap
- Preventive actions

❌ No blame, no naming individuals

---

## D14. Ownership & On-Call

### Ownership

- Every service has a clear owner
- Ownership is documented

### On-Call

- Rotations are mandatory
- No single-person dependency

---

## D15. Operational Anti-Patterns (Forbidden)

🚫 SSH into prod containers
🚫 Manual DB edits
🚫 Disabling alerts to avoid noise
🚫 Deploying without monitoring

---

**Appendix D is mandatory for all production systems.**

---

# Appendix E: Team Scaling, Ownership & Engineering Culture

## E1. Team Structure & Scaling Model

### Squad Model (Recommended)

- 1 Tech Lead
- 4–7 Engineers
- Optional QA / DevOps support

Rules:
- Teams own services end-to-end
- No shared responsibility without ownership

❌ "Everyone owns everything" is forbidden

---

## E2. Service Ownership Model

### Mandatory

- Every service has:
  - Owner team
  - Tech Lead
  - Slack / contact channel
  - Runbook

Ownership includes:
- Code quality
- CI/CD health
- Production incidents

---

## E3. Pull Request (PR) Rules

### Size Limits

- ≤ 400 lines preferred
- > 600 lines requires justification

---

### Review Rules

- Minimum 1 senior reviewer
- Author cannot self-approve
- Review within 24 working hours

❌ Rubber-stamp approvals are forbidden

---

## E4. Decision-Making (ADR / RFC)

### When Required

- New architecture
- New framework
- Breaking API change
- Security-impacting changes

---

### ADR Template (Mandatory)

- Context
- Decision
- Alternatives considered
- Consequences

❌ Decisions in Slack only are forbidden

---

## E5. Technical Debt Policy

### Rules

- Tech debt must be visible
- Logged as first-class tickets
- Paid down continuously

Recommended budget:
- 20–30% of sprint capacity

---

## E6. Refactor vs Rewrite Rules

### Refactor When

- Behavior remains same
- Gradual improvement possible

### Rewrite When

- Domain fundamentally wrong
- Refactor cost > rewrite cost

❌ Rewrites without approval are forbidden

---

## E7. Senior Engineer Expectations

Seniors must:
- Raise code quality bar
- Mentor juniors
- Say "no" to bad ideas
- Protect long-term health

❌ Seniors bypassing rules is unacceptable

---

## E8. Junior Engineer Expectations

Juniors must:
- Follow guidelines strictly
- Ask early
- Write tests

❌ Copy-paste without understanding is forbidden

---

## E9. Knowledge Sharing

### Mandatory

- Design docs for major changes
- Postmortems shared org-wide
- Regular tech talks

---

## E10. Documentation Rules

- Docs live with code
- README required per service
- Runbooks mandatory for prod services

❌ Outdated docs are considered bugs

---

## E11. Engineering Values (Non-Negotiable)

- Psychological safety
- Blameless culture
- Continuous improvement
- Respectful communication

---

## E12. Anti-Patterns (Cultural)

🚫 Hero culture
🚫 Fear-based reviews
🚫 Silent disagreement
🚫 Knowledge hoarding

---

## E13. Enforcement & Accountability

- Repeated violations trigger review
- Guidelines override personal preference
- Exceptions require written approval

---

**Appendix E completes the Full-Stack Engineering Guidelines.**

---

# Final Note

These guidelines are designed to scale teams, systems, and careers.
Deviation is allowed only with conscious, documented decisions.

