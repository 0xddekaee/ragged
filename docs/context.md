# Context

Context is the final assembly of information sent to the LLM. Even with perfect retrieval, poor context formatting can lead to poor answers.

## Problem 7: Context Poisoning

### Symptom

The LLM receives conflicting information:

```
Source A: "The timeout is 30 seconds."
Source B: "The timeout was increased to 60 seconds in version 3.0."
```

The LLM doesn't know which source to trust and may:
- Hallucinate a compromise ("45 seconds")
- Pick the wrong answer
- Get confused and fail to answer

### Why it happens

- Outdated documents are not filtered out
- Multiple versions of the same document coexist in the index
- Documents from different sources conflict

### Impact

The LLM's response is unreliable, even though the correct information exists in the context.

### Fix: Context Curation

1. **Filter outdated documents** before retrieval:

```yaml
versioning:
  enabled: true
  keep_versions: ["latest"]   # only index current version
```

2. **Rank sources by recency** when assembling context:

```python
context_chunks = sorted(
    chunks,
    key=lambda c: c.metadata.get("date", ""),
    reverse=True
)
```

3. **Deduplicate similar chunks** after retrieval:

```yaml
reranking:
  enabled: true
  top_n: 5   # reduce redundancy before sending to LLM
```

## Context Assembly Best Practices

### 1. Order matters

Place the most relevant chunks first. The LLM pays more attention to early context.

```python
# Good: most relevant first
context = "\n\n".join(chunks)

# Bad: random order
context = "\n\n".join(random.sample(chunks, len(chunks)))
```

### 2. Include source attribution

Help the LLM distinguish between sources:

```yaml
context:
  template: |
    Use the following context to answer the question.
    Each source is labeled with its metadata.

    Source 1 [version=3.2, date=2026-08-01]:
    {chunk_1}

    Source 2 [version=2.0, date=2026-03-15]:
    {chunk_2}

    Question: {question}
```

### 3. Respect token limits

Don't exceed the LLM's context window:

```yaml
context:
  max_tokens: 4000    # leave room for the question and response
```

## Context Template Example

```yaml
context:
  max_tokens: 4000
  template: |
    You are a helpful assistant. Answer the question using only the
    provided context. If the answer is not in the context, say so.

    Context:
    {context}

    Question: {question}

    Answer:
```

## Further Reading

- [Retrieval](retrieval.md) - how retrieval affects context quality
- [Metadata](metadata.md) - using metadata in context
- [Versioning](versioning.md) - preventing outdated information in context