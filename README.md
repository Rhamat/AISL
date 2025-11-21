# AISL — Artificial Intelligence Serialization Language

> A token-efficient, AI-native data serialization format designed for machine consumption.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-green.svg)]()

---

## What is AISL?

**AISL (Artificial Intelligence Serialization Language)** is a serialization format purpose-built for AI systems. Unlike JSON, YAML, or XML—which prioritize human readability—AISL optimizes for token efficiency and deterministic parsing by large language models.

### Core Principles

- **Token-efficient**: Eliminates quotes, braces, and redundant syntax that consume tokens without adding semantic value.
- **Human-optional**: Optimized for AI consumption; human readability is a secondary concern.
- **Flattened hierarchy**: Represents nested structures as dot-notation key paths (`contact.address.city`).
- **Linear structure**: Each record is a single logical unit with optional multi-line continuation.
- **Deterministic parsing**: Predictable syntax reduces hallucination risk and parsing errors.
- **Lossless conversion**: Any AISL document can be converted back to JSON without data loss.

---

## Motivation

### The Problem with Existing Formats

Modern AI systems process structured data constantly—API responses, configuration files, database records, and more. The formats we use today were designed decades ago for human developers:

| Format | Primary Design Goal |
|--------|---------------------|
| JSON   | Human-readable data interchange |
| YAML   | Human-friendly configuration |
| XML    | Document markup with extensibility |
| TOML   | Human-readable configuration |

These formats introduce inefficiencies when consumed by AI:

1. **Repeated structural tokens**: Every `{`, `}`, `[`, `]`, and `"` consumes tokens without semantic value.
2. **Redundant key repetition**: In arrays of objects, the same keys appear repeatedly.
3. **Whitespace and indentation**: Formatting for humans wastes context window space.
4. **Ambiguous nesting**: Deep hierarchies increase parsing complexity and hallucination risk.
5. **Irregular structure**: Mixed types and optional fields confuse pattern recognition.

### The AISL Solution

AISL addresses these issues by:

- Linearizing hierarchical data into flat key-value pairs
- Using minimal delimiters (`|`, `%`, `~`)
- Eliminating quotes around strings by default
- Providing optional type hints for unambiguous parsing
- Creating predictable, scannable record structures

The result: **fewer tokens, clearer structure, more reliable AI processing**.

---

## Key Features

### Token Efficiency

AISL removes syntactic overhead that provides no semantic value to AI systems:

```
# JSON (43 tokens typical)
{"user": {"name": "Alice", "age": 32, "active": true}}

# AISL (approximately 15 tokens)
record:user|name=Alice|age:int=32|active:bool=true%
```

### Flattened Hierarchy

Nested structures become dot-notation paths:

```
# Instead of nested objects:
contact.address.street=123 Main St|contact.address.city=Boston|contact.address.zip=02101%
```

### Delimiters

| Symbol | Purpose | Example |
|--------|---------|---------|
| `\|`   | Field separator | `name=Alice\|age=32` |
| `%`    | Record terminator | `...active=true%` |
| `~`    | Array element marker | `tags~python\|tags~ai` |
| `\`    | Line continuation | `name=Alice \` (continues next line) |
| `.`    | Hierarchy separator | `user.contact.email` |

### Optional Type Hints

Explicit types eliminate ambiguity:

| Hint | Meaning | Example |
|------|---------|---------|
| `:int` | Integer | `age:int=32` |
| `:float` | Floating point | `price:float=19.99` |
| `:bool` | Boolean | `active:bool=true` |
| `:str` | String (explicit) | `code:str=007` |
| `:null` | Null value | `deleted:null=` |

### Deterministic Parsing

Every AISL document has exactly one valid interpretation. There are no optional commas, no flexible whitespace rules, and no ambiguous type coercion. This predictability reduces AI hallucinations and missing-field errors.

---

## Syntax Reference

### Single-Line Record

The most common pattern—a complete record on one line:

```
record:user-001|name=Alice|age:int=32|status=active%
```

**Structure:**
- `record:user-001` — Record identifier (optional but recommended)
- `|` — Field separator
- `key=value` — Field assignment
- `%` — Record terminator

### Multi-Line Record

For records with many fields, use line continuation:

```
record:user-001\
  name=Alice Johnson\
  age:int=32\
  contact.email=alice@example.com\
  contact.phone=555-0123\
  preferences.theme=dark\
  preferences.notifications:bool=true%
```

The `\` at line end indicates continuation. The record terminates at `%`.

### Arrays

Use the `~` operator to denote array membership:

```
record:user-001|name=Alice|emails~alice@personal.com|emails~alice@work.com|emails~alice@school.edu%
```

This represents:
```json
{
  "name": "Alice",
  "emails": ["alice@personal.com", "alice@work.com", "alice@school.edu"]
}
```

### Nested Arrays

Combine dot-notation with array markers:

```
record:order-500\
  items~0.product=Widget\
  items~0.qty:int=2\
  items~0.price:float=9.99\
  items~1.product=Gadget\
  items~1.qty:int=1\
  items~1.price:float=24.99%
```

### Empty and Null Values

```
middle_name=|deleted_at:null=%
```

- `middle_name=` — Empty string
- `deleted_at:null=` — Explicit null

### Special Characters

If values contain delimiters, use explicit string type:

```
bio:str=Works at Acme | Loves coding%
```

---

## Format Comparison

| Aspect | JSON | YAML | XML | TOML | AISL |
|--------|------|------|-----|------|------|
| **Primary audience** | Humans | Humans | Documents | Humans | AI |
| **Verbosity** | Medium | Low | High | Low | Very Low |
| **Token efficiency** | Poor | Medium | Very Poor | Medium | Excellent |
| **Nested data** | Native | Native | Native | Limited | Flattened |
| **Schema support** | External | External | XSD/DTD | Native types | Type hints |
| **Human readability** | Good | Excellent | Fair | Excellent | Minimal |
| **AI parse reliability** | Good | Poor | Fair | Good | Excellent |
| **Whitespace sensitive** | No | Yes | No | No | No |

### Token Count Example

Representing the same data across formats:

**Data:** A user with name, email, age, and two tags.

| Format | Approximate Tokens (o200k) |
|--------|---------------------------|
| XML    | 65-75 |
| JSON   | 35-42 |
| YAML   | 28-35 |
| TOML   | 25-32 |
| AISL   | 18-24 |

*Actual savings depend on data shape, key length, and value types.*

---

## Token Testing

### Measuring Token Efficiency

To validate AISL's efficiency for your use case, test with OpenAI's `o200k_base` tokenizer (used by GPT-4o and similar models):

**Using Python:**

```python
import tiktoken

enc = tiktoken.get_encoding("o200k_base")

json_data = '{"name": "Alice", "age": 32, "active": true}'
aisl_data = 'record:user|name=Alice|age:int=32|active:bool=true%'

print(f"JSON tokens: {len(enc.encode(json_data))}")
print(f"AISL tokens: {len(enc.encode(aisl_data))}")
```

**Using the command line:**

```bash
# Install tiktoken
pip install tiktoken

# Compare files
python -c "
import tiktoken
enc = tiktoken.get_encoding('o200k_base')
with open('data.json') as f: print('JSON:', len(enc.encode(f.read())))
with open('data.aisl') as f: print('AISL:', len(enc.encode(f.read())))
"
```

### When AISL May Not Help

AISL is optimized for structured records. Token savings may be minimal or negative for:

- Very short, simple values
- Data with extremely long string values
- Deeply irregular schemas
- Single-field records

**The primary goal is AI-friendliness, not token minimization in all cases.**

---

## Usage Guide

JSON TO AISL CONVERTER - https://aisl-web.github.io/AISL/

### Supported Data Types

| Type | AISL Representation | Notes |
|------|---------------------|-------|
| String | `key=value` | Default type, no quotes needed |
| Integer | `key:int=42` | Whole numbers |
| Float | `key:float=3.14` | Decimal numbers |
| Boolean | `key:bool=true` | `true` or `false` |
| Null | `key:null=` | Explicit null value |
| Array | `key~value` | Multiple entries with same key |
| Object | `key.subkey=value` | Dot-notation flattening |

### Limitations

- **No comments**: AISL does not support inline comments
- **No multi-line strings**: Use `\n` escape sequences
- **Reserved characters**: `|`, `%`, `~`, `\` require escaping in values
- **Key restrictions**: Keys cannot contain `=`, `.`, or delimiter characters

---

## Specification

### ABNF Grammar (Simplified)

```abnf
document     = *record
record       = [record-id] fields "%"
record-id    = "record:" identifier "|"
fields       = field *("|" field)
field        = key [":" type] "=" value
key          = identifier *("." identifier)
type         = "int" / "bool" / "float" / "str" / "null"
value        = *VCHAR
identifier   = ALPHA *(ALPHA / DIGIT / "-" / "_")
```

### Escaping Rules

| Character | Escape Sequence |
|-----------|-----------------|
| `\|` (pipe) | `\\|` |
| `%` | `\\%` |
| `~` | `\\~` |
| `\` | `\\\\` |
| Newline | `\\n` |

---

## Roadmap

### Version 1.1

- [ ] Improved array syntax for complex nested arrays
- [ ] Binary data encoding (base64 type hint)
- [ ] Date/time type hints (`:date`, `:datetime`)

### Version 2.0

- [ ] Schema definition language (AISL-Schema)
- [ ] Streaming parser for large documents
- [ ] Compression hints for repeated values
- [ ] Reference syntax for deduplication

### Tooling

- [ ] Official CLI tool with validation
- [ ] VSCode extension with syntax highlighting
- [ ] Language bindings: JavaScript, Go, Rust, Java
- [ ] JSON Schema to AISL-Schema converter
- [ ] Online playground and converter

### Integrations

- [ ] OpenAI function calling format support
- [ ] LangChain document loader
- [ ] Database export plugins

---

## Contributing

We welcome contributions from the community.

### Reporting Issues

1. Search existing issues to avoid duplicates
2. Use the issue template
3. Include minimal reproduction examples
4. Specify your environment (Python version, OS)

### Proposing Specification Changes

1. Open a discussion in the Ideas category
2. Provide rationale with token-efficiency analysis
3. Include before/after examples
4. Consider backward compatibility

### Code Contributions

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Follow the code style (Black formatting, type hints)
4. Add tests for new functionality
5. Update documentation
6. Submit a pull request

### Style Guidelines for AISL Files

- Use lowercase keys with hyphens for multi-word names
- Include record identifiers for debugging
- Group related fields together
- Use type hints for non-string values
- One record per logical entity

---

## FAQ

**Q: Is AISL meant to replace JSON?**

No. JSON remains excellent for human-facing APIs and configuration. AISL is designed for AI-to-AI communication and contexts where token efficiency matters.

**Q: Can I use AISL for configuration files?**

You can, but TOML or YAML may be better choices if humans will edit the files directly.

**Q: How does AISL handle schema validation?**

Version 1.0 relies on type hints for basic validation. Full schema support is planned for v2.0.

**Q: Is AISL whitespace-sensitive?**

No. Whitespace around delimiters is ignored, except within values.

---

## License

MIT License

Copyright (c) 2025 AISL Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

---

<p align="center">
  <strong>AISL</strong> — Data serialization for the age of AI.
</p>
