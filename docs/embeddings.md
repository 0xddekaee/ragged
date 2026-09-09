# Embeddings

The embedding model is the bridge between text and vector space. Choosing the wrong model is like using a bad lens: everything downstream is blurry.

## Problem 4: Embedding Mismatch

### Symptom

The user asks:

```
"gimana cara reset password?"
```

The document says:

```
"credential recovery procedure"
```

These are semantically identical, but the embedding model fails to map them close together in vector space.

### Why it happens

Different embedding models have different strengths:

- Some models are optimized for **semantic similarity** (paraphrase, intent)
- Some models are optimized for **lexical similarity** (keyword matching)
- Some models are fine-tuned for **specific domains** (code, legal, medical)

A model trained on general web text may not understand domain-specific terminology or code-switching (mixing languages).

### Impact

Relevant documents are missed. The LLM receives incomplete context and cannot answer correctly.

### Fix

1. **Choose the right model for your domain:**

```yaml
embeddings:
  provider: "voyage"           # voyage-large-2 excels at domain-specific retrieval
  model: "voyage-large-2"
```

2. **Use multilingual models if needed:**

```yaml
embeddings:
  provider: "openai"
  model: "text-embedding-3-large"   # supports 100+ languages
```

3. **Test before committing:**

```bash
# Test similarity between two semantically related texts
python -c "
from ragedd import embed
e = embed.Embedder(provider='openai', model='text-embedding-3-small')
v1 = e.embed('gimana cara reset password?')
v2 = e.embed('credential recovery procedure')
print(e.similarity(v1, v2))  # Should be high (>0.7)
"
```

## Model Comparison

| Model | Dimensions | Strengths | Weaknesses |
|-------|-----------|-----------|------------|
| text-embedding-3-small | 512-1536 | Fast, cheap, good general purpose | Weaker on domain-specific text |
| text-embedding-3-large | 256-3072 | Better accuracy, multilingual | More expensive |
| voyage-large-2 | 1024 | Excellent on domain tasks, code, legal | Proprietary |
| Cohere Embed v3 | 1024 | Multilingual, search-optimized | Proprietary |
| BAAI/bge-large-en | 1024 | Open source, strong performance | Requires GPU for speed |

## Configuration Tips

```yaml
embeddings:
  provider: "openai"
  model: "text-embedding-3-small"
  dimensions: 1536        # reduce dimensions for cost/speed
  normalize: true         # L2 normalize for cosine similarity
  batch_size: 64          # batch embeddings for efficiency
```

## Further Reading

- [Retrieval](retrieval.md) - how embeddings affect retrieval quality
- [Chunking](chunking.md) - how chunking affects embedding quality