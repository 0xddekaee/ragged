# Retrieval

Retrieval is the core of RAG. It determines which documents the LLM sees. Poor retrieval poisons the entire pipeline regardless of how good your LLM is.

## Problem 2: Naive Top-K

### The naive approach

```
query → embedding → top-k → LLM
```

You embed the query, find the k most similar chunks by vector distance, and pass them to the LLM.

**Why it seems reasonable:** Simple, fast, works in many cases.

**The problem:** Highest similarity does not mean highest relevance.

Two chunks can be very similar to each other and both be similar to the query, yet only one contains the actual answer. The other is a redundant near-duplicate.

### Example

```
Query: "How do I reset my password?"

Chunk A: "To reset your password, click 'Forgot Password' on the login page."
Chunk B: "Password reset is available on the login page. Click the link below."
Chunk C: "User authentication requires a password reset flow."
```

All three chunks have high cosine similarity to the query. But Chunk B and C add no new information beyond Chunk A. Meanwhile, Chunk D ("Account settings → Security → Change password") might be slightly less similar but contains important additional context that gets excluded.

### Fix: Diversity-aware retrieval

Use Maximum Marginal Relevance (MMR) to balance relevance with diversity:

```yaml
retrieval:
  use_mmr: true
  mmr_lambda: 0.7          # 0 = pure diversity, 1 = pure relevance
  fetch_k: 50             # candidates to consider before MMR
```

MMR works by:
1. Fetching a larger candidate set (`fetch_k`)
2. Iteratively selecting chunks that are relevant AND dissimilar to already-selected chunks
3. `mmr_lambda` controls the tradeoff

## Problem 3: Retrieval Redundancy

### Symptom

Top-k results are filled with near-duplicates:

```
Chunk A → relevant ✓
Chunk B → almost identical to A
Chunk C → almost identical to A
Chunk D → almost identical to A
Chunk E → almost identical to A
```

Result: The LLM sees the same information five times, while other important information never makes it into context.

### Why it happens

Embedding models tend to cluster similar phrasings close together in vector space. If Chunk A and Chunk B use similar words, their embeddings will be close regardless of whether they convey new information.

### Fix: MMR + Reranking

Combine diversity-aware retrieval with a cross-encoder reranker:

```yaml
retrieval:
  use_mmr: true
  mmr_lambda: 0.7
  fetch_k: 50

reranking:
  enabled: true
  provider: "cohere"
  model: "rerank-v3.5"
  top_n: 5
```

The pipeline becomes:

```
query → embedding → top 50 (fetch_k) → MMR → top 10 → reranker → top 5 → LLM
```

## Problem 5: Query Terlalu Pendek

### Symptom

A query like `"pricing"` retrieves very different results than `"What is the monthly pricing for the Pro plan?"`

**Why it happens:** Short queries have sparse semantic information. The embedding model has less context to work with, so results are noisier.

### Fix: Query expansion and rewriting

```yaml
query:
  expand: true    # expand short queries using LLM
  rewrite: true   # rewrite queries for better retrieval
```

Example:

```
Original: "pricing"
Expanded:  "pricing plans for Pro and Enterprise tiers, monthly and annual billing"
```

The expanded query retrieves much more targeted results.

## Retrieval Pipeline Summary

```
┌─────────────┐
│   Query     │
└──────┬──────┘
       ▼
┌──────────────┐
│ Query Expand │  (if enabled)
│   & Rewrite  │
└──────┬───────┘
       ▼
┌──────────────┐
│  Embed Query │
└──────┬───────┘
       ▼
┌──────────────┐
│ Vector Search│  fetch_k candidates
└──────┬───────┘
       ▼
┌──────────────┐
│ Metadata     │  (if enabled)
│   Filter     │
└──────┬───────┘
       ▼
┌──────────────┐
│     MMR      │  (if enabled)
│   Rerank     │
└──────┬───────┘
       ▼
┌──────────────┐
│ Cross-Encoder│  (if enabled)
│  Reranker    │
└──────┬───────┘
       ▼
┌──────────────┐
│ Final Top-N  │
│   → LLM      │
└──────────────┘
```

## Further Reading

- [Chunking](chunking.md) - how chunk size affects retrieval
- [Embeddings](embeddings.md) - embedding model selection
- [Metadata](metadata.md) - filtering with metadata
- [Context](context.md) - assembling the final context