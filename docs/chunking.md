# Chunking

Chunking is the first and most critical step in any RAG pipeline. How you split your documents directly affects retrieval quality.

## Common Problems

### 1. Chunks that are too large

**Symptom:** Retrieval returns chunks that are mostly noise. The LLM receives a wall of text, most of which is irrelevant to the actual question.

**Why it happens:** Large chunks dilute the signal-to-noise ratio. A chunk with 2000 tokens might contain only 50 tokens that actually answer the query.

**Impact:** LLM gets confused, hallucinates, or misses the answer buried in noise.

**Fix:** Reduce chunk size. Use hierarchical chunking (see [hierarchy.md](hierarchy.md)) to preserve both detail and context.

### 2. Chunks that are too small

**Symptom:** Retrieved chunks lack sufficient context. The LLM cannot answer because the relevant information is split across multiple chunks that don't get retrieved together.

**Why it happens:** Very small chunks (e.g., 50 tokens) lose the broader semantic context. A single sentence might not contain enough information to be useful.

**Impact:** Incomplete answers, "I don't know" responses when the information actually exists in the document.

**Fix:** Use chunk overlap, parent-child hierarchy, or increase chunk size.

### 3. Semantic boundary violations

**Symptom:** A chunk cuts through the middle of a table, a function definition, a paragraph, or a section header.

**Why it happens:** Naive fixed-size splitting ignores document structure. It splits at arbitrary character positions regardless of content.

**Impact:** 
- Tables lose row/column relationships
- Code snippets become syntactically invalid
- Paragraphs lose their argument flow
- Embeddings become misleading because the chunk is semantically broken

**Fix:** Use semantic chunking that respects document structure:

```yaml
chunking:
  strategy: "semantic"
  semantic:
    threshold: 0.5   # split when embedding distance exceeds threshold
```

Or enable structure preservation in parsing:

```yaml
parsing:
  preserve_tables: true
  preserve_headings: true
```

## Recommended Configuration

```yaml
chunking:
  strategy: "hierarchical"   # best of both worlds
  chunk_size: 512            # child chunks for retrieval
  chunk_overlap: 64          # overlap between adjacent chunks
  hierarchy:
    parent_chunk_size: 2048  # parent chunks for context
```

See [hierarchy.md](hierarchy.md) for details on hierarchical chunking.

## Further Reading

- [Hierarchy](hierarchy.md) - preserve document structure across chunk sizes
- [Parsing](parsing.md) - how to preserve tables and headings during parsing