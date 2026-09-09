# Contributing to ragedd

Thanks for your interest in contributing! This project documents common RAG problems and provides configuration solutions.

## How to Contribute

### 1. Documentation

Documentation is the primary contribution area. When adding or updating docs:

- Follow the existing structure: **Problem → Why it happens → Impact → Fix**
- Use YAML code blocks for configuration examples
- Include concrete examples, not just theory
- Reference related docs at the bottom

### 2. Configuration

When adding new configuration options:

- Add them to `configs/rag.yaml` first
- Update `examples/production.yaml` if it's a production-relevant option
- Document the option in the relevant `docs/*.md` file

### 3. Issues

Report issues at: https://github.com/kilo-org/ragedd/issues

### 4. Pull Requests

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Ensure all documentation is updated
5. Submit a pull request

## Documentation Standards

Each doc file should follow this structure:

```markdown
# Topic

## Problem N: Description

### Symptom
- What users experience

### Why it happens
- Root cause

### Impact
- Consequences

### Fix
```yaml
# Configuration example
```

## Further Reading
- [Related doc](related.md)
```

## Code Style

This project is primarily configuration and documentation. Follow these conventions:

- YAML: 2-space indentation
- Comments: Explain the "why", not the "what"
- Naming: Use snake_case for config keys
- Values: Use human-readable strings, not codes

## License

By contributing, you agree that your contributions will be licensed under the MIT License.