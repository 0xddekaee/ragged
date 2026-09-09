# ragedd

A configuration-driven RAG (Retrieval-Augmented Generation) toolkit that helps you build better RAG pipelines by addressing the 10 most common pitfalls in production RAG systems.

## Overview

ragedd is not a library or framework. It is a **configuration and documentation project** that teaches you how to fix the most common RAG problems through well-structured configs and detailed documentation.

## The 10 Common RAG Problems

| # | Problem | Documentation |
|---|---------|---------------|
| 1 | Chunking yang salah | [chunking.md](docs/chunking.md) |
| 2 | Naive top-k retrieval | [retrieval.md](docs/retrieval.md) |
| 3 | Retrieval redundancy | [retrieval.md](docs/retrieval.md) |
| 4 | Embedding mismatch | [embeddings.md](docs/embeddings.md) |
| 5 | Query terlalu pendek | [query.md](docs/query.md) |
| 6 | Metadata tidak dimanfaatkan | [metadata.md](docs/metadata.md) |
| 7 | Context poisoning | [context.md](docs/context.md) |
| 8 | Document versioning | [versioning.md](docs/versioning.md) |
| 9 | Parent-child relationship hilang | [hierarchy.md](docs/hierarchy.md) |
| 10 | Tables dihancurkan parsing | [parsing.md](docs/parsing.md) |

## Project Structure

```
ragedd/
├── configs/              # Actual configuration
│   └── rag.yaml
├── examples/             # Example configurations
│   ├── minimal.yaml
│   └── production.yaml
├── docs/                 # Documentation
│   ├── chunking.md
│   ├── retrieval.md
│   ├── embeddings.md
│   ├── query.md
│   ├── metadata.md
│   ├── context.md
│   ├── versioning.md
│   ├── hierarchy.md
│   └── parsing.md
└── .github/              # CI/CD
```

## Quick Start

1. Copy the production configuration:

```bash
cp configs/rag.yaml your-project.yaml
```

2. Adapt it for your use case:

```yaml
# Edit these values for your setup
embeddings:
  provider: "openai"        # Your embedding provider
  model: "text-embedding-3-small"

vector_store:
  provider: "qdrant"        # Your vector database
  collection: "your-collection"

llm:
  provider: "openai"        # Your LLM provider
  model: "gpt-4o"
```

3. Read the documentation for each problem area:

```bash
# Start with the most relevant docs for your use case
open docs/chunking.md
open docs/retrieval.md
```

## Configuration Layers

Each configuration section addresses a specific problem:

```yaml
# Problem 1: Chunking
chunking: ...

# Problem 2-3: Retrieval
retrieval: ...

# Problem 4: Embeddings
embeddings: ...

# Problem 5: Query
query: ...

# Problem 6: Metadata
metadata: ...

# Problem 7: Context
context: ...

# Problem 8: Versioning
versioning: ...

# Problem 9: Hierarchy
hierarchy: ...

# Problem 10: Parsing
parsing: ...
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

See [LICENSE](LICENSE) for details.