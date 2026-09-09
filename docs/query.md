# Query

The query is the user's intent. How you process it before retrieval dramatically affects results.

## Problem 5: Query Terlalu Pendek

### Symptom

```
User query: "pricing"
```

Retrieval returns a mix of:
- Pricing pages
- Product pricing tiers
- Discount offers
- Billing information

But which one does the user actually want? Without more context, the results are noisy.

### Why it happens

Short queries (1-2 words) have:
- Low information density
- High ambiguity
- Sparse semantic signal for embedding models

### Fix: Query Expansion

Use an LLM to expand short queries before retrieval:

```yaml
query:
  expand: true
  max_query_length: 512
```

Example transformation:

```
Input:  "pricing"
Output: "What are the pricing plans? Include details about monthly
         and annual billing for all tiers (Free, Pro, Enterprise).
         Show storage limits and API rate limits."
```

The expanded query retrieves much more targeted results.

### Fix: Query Rewriting

Rewrite queries to match document terminology:

```yaml
query:
  rewrite: true
```

Example:

```
Original: "gimana reset password?"
Rewritten: "How to reset password? Describe the password recovery
           flow, including steps for forgotten password, email
           verification, and account unlock."
```

## When to Use Each Technique

| Technique | Best for | Cost |
|-----------|----------|------|
| Query expansion | Short queries (<5 words) | 1 LLM call per query |
| Query rewriting | Queries with ambiguous phrasing | 1 LLM call per query |
| Both | Very short or unclear queries | 2 LLM calls per query |

## Configuration

```yaml
query:
  expand: true
  rewrite: true
  max_query_length: 512
```

## Further Reading

- [Retrieval](retrieval.md) - how query quality affects retrieval
- [Context](context.md) - how to format the final context