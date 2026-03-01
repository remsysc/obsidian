
**One-liner:** Design your API around your domain's rules, not around what REST "should" look like.

## Why It Matters
Blindly following REST conventions leads to APIs that fight your business logic.
A publish action isn't a PATCH — it's a state transition with its own validation rules.
Model it as such.

## Core Concept
Industry standards are **starting points**, not laws.
Ask "what does this action mean in my domain?" before picking an HTTP verb.
```
POST   /api/v1/posts/drafts              // create draft — loose validation
PATCH  /api/v1/posts/{id}               // update draft fields
POST   /api/v1/posts/{id}/publication   // publish — strict validation (title required, etc.)
DELETE /api/v1/posts/{id}               // delete
GET    /api/v1/posts                    // list
GET    /api/v1/posts/{id}               // single post
```

Why `POST /posts/{id}/publication` instead of `PATCH /posts/{id}` with `status: PUBLISHED`?
- Publication has its own validation rules (a draft doesn't)
- It's a **command**, not a field update — this maps to CQRS thinking
- Easier to add publication-specific side effects (emails, webhooks) later

## Gotchas
- Don't over-engineer sub-resources. Only break out a sub-resource
  if it has meaningfully different behavior or validation
- `POST` on a sub-resource is correct for state transitions — you're not
  being non-RESTful, you're being pragmatic

## Tags
#backend #api-design #rest #system-design #domain-modeling

## Links
- [[State Machine Pattern for Entity Status]]
- [[CQRS — Commands vs Queries]]
- [[System-Owned vs User-Provided Fields]]