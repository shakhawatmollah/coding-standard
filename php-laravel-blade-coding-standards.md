# PHP, Laravel, Blade Coding Standards & Best Practices Guide

**Tech Stack:** PHP 8.2+ | Laravel 11+ | Blade | MySQL/PostgreSQL | Redis | Queue Workers

**Version:** 1.0  
**Last Updated:** February 12, 2026

---

## Table of Contents

1. [General Principles](#general-principles)
2. [PHP Language Standards](#php-language-standards)
3. [Laravel Application Architecture](#laravel-application-architecture)
4. [Blade Template Standards](#blade-template-standards)
5. [Validation, Security, and Auth](#validation-security-and-auth)
6. [Database and Eloquent Standards](#database-and-eloquent-standards)
7. [Caching, Queues, and Jobs](#caching-queues-and-jobs)
8. [Testing Standards](#testing-standards)
9. [Performance and Observability](#performance-and-observability)
10. [Code Review Checklist](#code-review-checklist)

---

## General Principles

### Rule Strength
- **MUST / MUST NOT**: Mandatory. Non-compliance blocks merge.
- **SHOULD / SHOULD NOT**: Strong recommendation with written rationale for deviations.
- **MAY**: Optional based on context.

### Core Engineering Values
- Readability over cleverness.
- Secure by default.
- Explicit contracts and predictable behavior.
- Consistent naming and folder structure.
- Testability and maintainability first.

---

## PHP Language Standards

### Naming and File Conventions

**MUST:**
- Use PSR-12 coding style.
- Use `PascalCase` for classes, `camelCase` for methods/variables.
- Use descriptive names (`calculateInvoiceTotal`, not `calc`).
- Keep one class per file and namespace by feature/domain.

**MUST NOT:**
- Use ambiguous abbreviations.
- Keep dead/commented-out code in committed files.

### Type Safety and Strictness

**MUST:**
- Declare strict types in PHP files where applicable.
- Use scalar/return types and nullable types explicitly.
- Prefer value objects/DTOs over loose arrays for domain data.

```php
<?php

declare(strict_types=1);

final class PriceCalculator
{
    public function calculate(float $base, float $taxRate): float
    {
        return $base + ($base * $taxRate);
    }
}
```

---

## Laravel Application Architecture

### Layering and Responsibilities

**MUST:**
- Keep controllers thin (HTTP orchestration only).
- Move business logic into services/actions.
- Keep repositories/query objects focused on data access.
- Use Form Requests for validation.

**MUST NOT:**
- Place complex business logic in controllers or Blade templates.
- Access DB directly from views.

### Project Structure (Feature-Oriented)

```text
app/
|-- Domain/
|-- Http/
|   |-- Controllers/
|   |-- Requests/
|   \-- Middleware/
|-- Services/
|-- Actions/
|-- Policies/
|-- Jobs/
|-- Events/
|-- Listeners/
\-- Models/
```

---

## Blade Template Standards

### Template Design Rules

**MUST:**
- Keep Blade focused on presentation only.
- Use `@extends`, `@section`, `@include`, and components for reuse.
- Escape all untrusted output with `{{ }}`.
- Use named components for repeated UI patterns.

**MUST NOT:**
- Place business logic or DB calls in Blade.
- Use raw echo (`{!! !!}`) unless content is trusted and sanitized.

```blade
{{-- Good --}}
<x-alert type="success" :message="$message" />

{{-- Bad: unsafe unless sanitized/trusted --}}
{!! $userInput !!}
```

### Blade Components

**SHOULD:**
- Use components for forms, alerts, cards, and table fragments.
- Keep component APIs explicit via props.
- Prefer slots for flexible layout content.

---

## Validation, Security, and Auth

### Input Validation

**MUST:**
- Validate all inbound request data via Form Requests.
- Use custom rules for domain-specific invariants.
- Validate file uploads (MIME, size, dimensions where needed).

### Security Baselines

**MUST:**
- Use CSRF protection for state-changing routes.
- Enforce authorization through policies/gates.
- Hash passwords using Laravel `Hash` facade.
- Store secrets in environment/secret manager, never in code.

**MUST NOT:**
- Trust client-supplied authorization claims without server checks.
- Return sensitive fields from APIs/logs.

### Authentication & Session

**SHOULD:**
- Use Laravel Sanctum/Passport based on use case.
- Rotate tokens/credentials and enforce expiration.
- Use MFA for admin or privileged roles.

---

## Database and Eloquent Standards

### Modeling

**MUST:**
- Use migrations for all schema changes.
- Define constraints, indexes, and foreign keys explicitly.
- Use `$fillable`/`$guarded` intentionally.

### Query Discipline

**MUST:**
- Prevent N+1 with eager loading.
- Select only needed columns.
- Paginate large result sets.

```php
$users = User::query()
    ->select(['id', 'name', 'email'])
    ->with(['roles:id,name'])
    ->paginate(25);
```

**MUST NOT:**
- Use unbounded queries in user-facing endpoints.
- Use raw SQL with string concatenation.

---

## Caching, Queues, and Jobs

### Caching

**SHOULD:**
- Cache expensive queries and computed view models.
- Use clear cache keys and TTL strategy.
- Invalidate cache on write paths.

### Queues and Jobs

**MUST:**
- Make jobs idempotent.
- Configure retries/backoff and failure handling.
- Use queue segregation for latency-sensitive workloads.

**MUST NOT:**
- Run long synchronous tasks in request lifecycle.

---

## Testing Standards

### Required Testing

**MUST:**
- Add feature tests for critical HTTP flows.
- Add unit tests for domain/service logic.
- Cover auth, authorization, validation, and error paths.

### Test Quality

**SHOULD:**
- Use factories/seeders for deterministic data setup.
- Isolate side effects with fakes/mocks (`Bus::fake`, `Mail::fake`).
- Keep tests readable and behavior-focused.

---

## Performance and Observability

### Performance

**MUST:**
- Track slow queries and optimize indexes.
- Use eager loading and avoid repeated hydration.
- Optimize Blade rendering for large loops/components.

### Observability

**MUST:**
- Use structured logs with request correlation IDs.
- Log business-critical failures and queue dead-letter events.
- Expose health checks for app, DB, cache, queue.

---

## Code Review Checklist

### Author Checklist
- [ ] Follows PSR-12 and project conventions.
- [ ] Validation and authorization implemented.
- [ ] No secrets/hardcoded credentials in code.
- [ ] Tests added/updated and passing.
- [ ] Migrations safe and reversible.
- [ ] Blade output is escaped by default.

### Reviewer Checklist
- [ ] Controller is thin; logic in service/action layers.
- [ ] No N+1 or unbounded queries.
- [ ] Policies/gates are enforced for sensitive actions.
- [ ] Error handling and logging are adequate.
- [ ] Backward compatibility impact is documented.

---

## Commit Message Convention

Use conventional commits:

```text
type(scope): subject
```

Examples:
- `feat(auth): add mfa challenge flow`
- `fix(blade): escape user profile bio output`
- `refactor(user): move registration logic to action`
- `test(order): add feature tests for checkout validation`
