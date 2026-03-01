## The Real Lesson Here

> **"Industry standards are starting points, not absolute rules."**

Always ask **business requirements first**, THEN apply patterns. The best API design serves your domain — not the other way around.

## ✅ Your Correct Design All Along

```
POST   /api/v1/posts/drafts              // create draft, loose
PATCH  /api/v1/posts/{id}               // update draft, loose  
POST   /api/v1/posts/{id}/publication   // publish, strict
DELETE /api/v1/posts/{id}               // delete
GET    /api/v1/posts                    // get all
GET    /api/v1/posts/{id}               // get one
```

[[System Owned vs User Provided Fields]]