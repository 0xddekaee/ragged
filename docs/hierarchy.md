# Hierarchy

Document hierarchy is the structure that connects chunks to their parent sections. Preserving this structure is critical for accurate retrieval.

## Problem 9: Parent-Child Relationship Hilang

### Symptom

A retrieved chunk says:

```
"The timeout can be increased to 60 seconds."
```

But the LLM has no context about:
- What system this applies to (database, API, frontend?)
- What section this belongs to (Connection Settings?)
- What the broader configuration looks like

### Why it happens

When you split documents into small chunks for indexing, you often discard the parent structure. Each chunk becomes an isolated island of text.

### Impact

- Chunks lose semantic context
- LLM cannot understand the broader topic
- Answers are incomplete or misleading

### Example

```
Database Configuration
  └── Connection Settings
       └── Timeout
            └── "The timeout can be increased to 60 seconds."
```

If you only index the leaf chunk, the LLM doesn't know this is about database connection timeouts.

### Fix: Hierarchical Chunking

Index both parent and child chunks, and retrieve them together:

```yaml
hierarchy:
  enabled: true
  store_parents: true          # index parent chunks too
  parent_chunk_size: 2048      # size of parent chunks
  child_chunk_size: 512        # size of child chunks
```

## How It Works

```
┌─────────────────────────────────────┐
│  Parent Chunk (2048 tokens)         │
│  "Database Configuration..."        │
│  Covers all database settings       │
└──────────┬──────────────────────────┘
           │
           ├── Parent Chunk (2048 tokens)
           │   "Connection Settings..."
           │   └── Child Chunk (512 tokens)
           │       "The timeout can be increased to 60 seconds."
           │
           ├── Child Chunk (512 tokens)
           │   "Max connections: 100"
           │
           └── Child Chunk (512 tokens)
               "Idle timeout: 300 seconds"
```

During retrieval:
1. Find relevant child chunks via vector search
2. Fetch their parent chunks
3. Include both in the context

## Configuration

```yaml
chunking:
  strategy: "hierarchical"
  chunk_size: 512              # child chunk size
  chunk_overlap: 64
  hierarchy:
    parent_chunk_size: 2048    # parent chunk size
    child_chunk_size: 512      # child chunk size

hierarchy:
  enabled: true
  store_parents: true
```

## Benefits

- **Context preservation:** Child chunks are always tied to their parent topic
- **Multi-level retrieval:** You can search at different granularity levels
- **Better answers:** LLM understands the broader context
- **Source navigation:** Users can trace back to the original document section

## Implementation Pattern

```python
# During indexing
parent_chunk = create_parent_chunk(document, section)
child_chunks = split_into_children(parent_chunk)

for child in child_chunks:
    index(child, parent_id=parent_chunk.id)

# During retrieval
child_results = vector_search(query, k=10)
parent_results = [fetch_parent(child.parent_id) for child in child_results]
context = deduplicate(parent_results + child_results)
```

## Further Reading

- [Chunking](chunking.md) - how chunk size affects hierarchy
- [Context](context.md) - assembling hierarchical context
- [Parsing](parsing.md) - preserving document structure during parsing