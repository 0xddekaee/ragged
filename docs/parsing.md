# Parsing

Parsing is the process of converting raw documents (PDF, HTML, markdown, etc.) into structured text for indexing. Poor parsing destroys information before it even reaches your RAG pipeline.

## Problem 10: Tables Dihancurkan ketika Parsing

### Symptom

Original table:

```
Plan  | Storage | Price
Free  | 10GB    | $0
Pro   | 1TB     | $20
```

After parsing:

```
Plan Storage Price Free 10GB $0 Pro 1TB $20
```

The table structure is destroyed. The relationship between columns and rows is lost.

### Why it happens

Naive parsers extract text linearly, without understanding table structure. They concatenate cells in reading order, losing the grid layout.

### Impact

- Embeddings still seem "similar" because the words are the same
- But the semantic meaning is degraded
- LLM receives unstructured text and cannot interpret the table
- Answers about table data are incorrect or incomplete

### Example

```
User: "What's the price for the Pro plan?"
Bad context: "Plan Storage Price Free 10GB $0 Pro 1TB $20"
Good context: "Plan: Pro, Storage: 1TB, Price: $20"
```

### Fix: Preserve Table Structure

```yaml
parsing:
  preserve_tables: true
  preserve_headings: true
```

## Parsing Best Practices

### 1. Preserve Tables

Convert tables to structured format:

```markdown
| Plan  | Storage | Price |
|-------|---------|-------|
| Free  | 10GB    | $0    |
| Pro   | 1TB     | $20   |
```

Or use key-value format:

```
Free plan: 10GB storage, $0/month
Pro plan: 1TB storage, $20/month
```

### 2. Preserve Headings

Keep the heading hierarchy intact:

```markdown
# Database Configuration

## Connection Settings

### Timeout

The timeout can be increased to 60 seconds.
```

This preserves the semantic context for chunking.

### 3. Extract Metadata

Extract title, author, date, and other metadata from source documents:

```yaml
parsing:
  extract_metadata: true
  languages: ["en", "id"]
```

## Supported Formats

| Format | Tables | Headings | Metadata | Notes |
|--------|--------|----------|----------|-------|
| Markdown | ✓ | ✓ | ✓ | Native support |
| HTML | ✓ | ✓ | ✓ | Use structured parsing |
| PDF | ⚠️ | ⚠️ | ⚠️ | Requires table detection |
| Plain text | ✗ | ✗ | ✗ | Limited structure |
| Word (DOCX) | ✓ | ✓ | ✓ | Good native support |

## Configuration

```yaml
parsing:
  preserve_tables: true
  preserve_headings: true
  extract_metadata: true
  languages: ["en"]   # for language detection
```

## Further Reading

- [Chunking](chunking.md) - how parsing affects chunking
- [Hierarchy](hierarchy.md) - preserving document structure
- [Metadata](metadata.md) - extracting metadata from documents