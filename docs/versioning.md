# Versioning

Document versioning is one of the most impactful features for production RAG systems. It prevents outdated or deprecated information from poisoning your LLM's responses.

## Problem 8: Document Versioning

### Symptom

Your knowledge base contains:

```
API v1 documentation (deprecated)
API v2 documentation (current)
```

A user asks: "How do I create an API key?"

Retrieval returns the v1 documentation because it's textually similar. The LLM gives instructions for the deprecated v1 flow.

**User follows the instructions → breaks their integration → blames your RAG system.**

### Why it happens

Vector databases don't understand time or version. They only understand similarity. If v1 docs are textually similar to the query, they get retrieved regardless of being deprecated.

### Impact

- Users get wrong instructions
- LLM responses are unreliable
- Support tickets increase
- Trust in the RAG system erodes

### Fix: Version-Aware Indexing

Only index the current version of each document:

```yaml
versioning:
  enabled: true
  field: "version"              # field holding version string
  keep_versions: ["latest"]     # only index "latest"
  deprecation_field: "deprecated"
  deprecation_value: true       # skip docs marked as deprecated
```

### How It Works

1. **During ingestion:** Documents are tagged with version metadata
2. **Filtering:** Only documents matching `keep_versions` are indexed
3. **During retrieval:** Version filters are applied automatically

```yaml
retrieval:
  use_metadata_filter: true
  metadata_filters:
    - field: "deprecated"
      operator: "eq"
      value: false
```

## Versioning Strategies

### Strategy 1: Keep Only Latest (Recommended)

```yaml
versioning:
  keep_versions: ["latest"]
```

Pros:
- Clean index
- No outdated information
- Smaller vector database

Cons:
- Cannot answer historical questions
- Lose version-specific context

### Strategy 2: Keep All, Filter at Retrieval

```yaml
versioning:
  keep_versions: ["all"]

retrieval:
  metadata_filters:
    - field: "deprecated"
      operator: "eq"
      value: false
```

Pros:
- Full history available
- Can answer "how did this work in v1?" questions

Cons:
- Larger index
- Risk of outdated info if filter is misconfigured

### Strategy 3: Time-Based Decay

```yaml
versioning:
  keep_versions: ["latest", "previous"]
  deprecation_field: "deprecated"
  deprecation_value: true
```

Keep the current and immediately previous version. Deprecate older versions automatically.

## Metadata for Versioning

Ensure every document has:

```json
{
  "source": "api-docs.pdf",
  "version": "2.0",
  "deprecated": false,
  "date": "2026-08-01",
  "changelog_url": "https://docs.example.com/changelog"
}
```

## Further Reading

- [Metadata](metadata.md) - how to track version metadata
- [Context](context.md) - preventing outdated info in LLM context
- [Retrieval](retrieval.md) - metadata filtering during retrieval