

**One-liner:** Design your API around your domain's rules, not around what REST "should" look like.

## Why It Matters
Blindly following REST conventions leads to APIs that fight your business logic.
A publish action isn't a PATCH — it's a state transition with its own validation rules.

## Core Concept
```
POST   /api/v1/posts/drafts              // create draft — loose validation
PATCH  /api/v1/posts/{id}               // update draft fields
POST   /api/v1/posts/{id}/publication   // publish — strict validation
DELETE /api/v1/posts/{id}               // delete
GET    /api/v1/posts                    // list
GET    /api/v1/posts/{id}               // single post
```

Why `POST /posts/{id}/publication` instead of `PATCH /posts/{id}` with `status: PUBLISHED`?
- Publication has its own validation rules (a draft doesn't)
- It's a **command**, not a field update — maps to CQRS thinking
- Easier to add side effects (emails, webhooks) later without polluting the PATCH handler

## Gotchas
- Don't over-engineer sub-resources — only break one out if it has meaningfully different behavior or validation
- `POST` on a sub-resource is correct for state transitions — you're being pragmatic, not non-RESTful

## Tags
#backend #api-design #rest #system-design #domain-modeling

## Links
- [[State Machine Pattern for Entity Status]] [[missing]]
- [[CQRS - Commands vs Queries]] [[missing]]
- [[System-Owned vs User-Provided Fields]]