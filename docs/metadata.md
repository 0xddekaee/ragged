# Metadata

Metadata is often overlooked but is one of the most powerful tools in RAG. It adds structured information that vector search alone cannot provide.

## Problem 6: Metadata Tidak Dimanfaatkan

### Symptom

Your documents have rich metadata:

```json
{
  "product": "Pro",
  "version": "3.2",
  "language": "en",
  "date": "2026-08-01",
  "source": "api-docs.pdf"
}
```

But retrieval only uses embeddings. All this structured information goes unused.

### Why it happens

Many RAG implementations skip metadata entirely, treating all documents as plain text. This wastes the opportunity to filter and rank results.

### Impact

- Outdated documents are retrieved alongside current ones
- Wrong-language documents appear in English queries
- Documents from the wrong product are returned

### Fix: Metadata Filtering

Apply metadata filters **before or during** vector search:

#### Option A: Pre-filter (recommended)

Filter the candidate set before vector search:

```yaml
retrieval:
  use_metadata_filter: true
  metadata_filters:
    - field: "deprecated"
      operator: "eq"
      value: false
    - field: "language"
      operator: "eq"
      value: "en"
```

#### Option B: Post-filter

Retrieve candidates first, then filter:

```python
candidates = vector_search(query_embedding, k=50)
candidates = [c for c in candidates if c.metadata["language"] == "en"]
candidates = [c for c in candidates if not c.metadata["deprecated"]]
```

Pre-filter is generally better because it reduces the search space and avoids missing relevant results that fall outside the top-k.

## Metadata Fields to Track

At minimum, index these fields:

```yaml
metadata:
  index_fields:
    - "source"          # which document this came from
    - "version"         # document version
    - "product"         # which product/feature
    - "language"        # language of the document
    - "date"            # last updated date
    - "deprecated"      # whether the doc is deprecated
```

## Filtering Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `eq` | Equal to | `version == "3.2"` |
| `neq` | Not equal to | `deprecated != true` |
| `gt` | Greater than | `date > "2026-01-01"` |
| `gte` | Greater or equal | `date >= "2026-01-01"` |
| `lt` | Less than | `date < "2027-01-01"` |
| `lte` | Less or equal | `date <= "2026-12-31"` |
| `in` | In list | `product in ["Pro", "Enterprise"]` |
| `contains` | Field contains value | `tags contains "security"` |

## Metadata + Vector Search Flow

```
┌──────────────┐
│   Query      │
└──────┬───────┘
       ▼
┌──────────────┐
│  Embed Query │
└──────┬───────┘
       ▼
┌──────────────┐
│ Metadata     │  ← Filter candidates BEFORE vector search
│   Filter     │
└──────┬───────┘
       ▼
┌──────────────┐
│ Vector Search│  Only searches filtered subset
│  (filtered)  │
└──────┬───────┘
       ▼
┌──────────────┐
│   Result     │
└──────────────┘
```

## Further Reading

- [Versioning](versioning.md) - how to use metadata for document versioning
- [Retrieval](retrieval.md) - metadata filtering in the retrieval pipeline
- [Context](context.md) - including metadata in the LLM context