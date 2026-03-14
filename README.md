# gemini-google-search-tool

[![Python Version](https://img.shields.io/badge/python-3.14+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Code style: ruff](https://img.shields.io/badge/code%20style-ruff-000000.svg)](https://github.com/astral-sh/ruff)
[![Type checked: mypy](https://img.shields.io/badge/type%20checked-mypy-blue.svg)](https://github.com/python/mypy)
[![AI Generated](https://img.shields.io/badge/AI-Generated-blueviolet.svg)](https://www.anthropic.com/claude)
[![Built with Claude Code](https://img.shields.io/badge/Built_with-Claude_Code-5A67D8.svg)](https://www.anthropic.com/claude/code)

A professional CLI tool and Python library for querying Gemini with Google Search grounding, connecting AI responses to real-time web content with automatic citations.

## Table of Contents

- [About](#about)
- [Use Cases](#use-cases)
- [Features](#features)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
  - [CLI Usage](#cli-usage)
  - [Library Usage](#library-usage)
- [Development](#development)
- [Testing](#testing)
- [Resources](#resources)
- [Contributing](#contributing)
- [License](#license)

## About

### What is Google Search Grounding?

[Google Search grounding](https://ai.google.dev/gemini-api/docs/grounding) connects Gemini models to real-time web content through Google Search. When you query Gemini with grounding enabled, the model:

1. Automatically determines if a search would improve the answer
2. Generates and executes appropriate search queries
3. Processes search results and synthesizes information
4. Returns a grounded response with verifiable sources

This reduces hallucinations and provides up-to-date information beyond the model's training cutoff.

### Why This CLI-First Tool?

This tool provides a **CLI-first** architecture designed for both humans and AI agents:

- **Agent-Friendly Design**: Structured commands and rich error messages enable AI agents (like Claude Code) to reason and act effectively in ReAct loops
- **Composable Architecture**: JSON output to stdout, logs to stderr—perfect for piping and automation
- **Reusable Building Blocks**: Commands serve as primitives for Claude Code skills, MCP servers, shell scripts, and custom workflows
- **Dual-Mode Operation**: Use it as a CLI tool or import as a Python library
- **Production Quality**: Type-safe (strict mypy), tested (pytest), with comprehensive error handling

## Use Cases

- 📚 **Real-Time Information**: Access current events, news, and recent developments
- 🔍 **Fact Verification**: Ground responses in verifiable web sources
- 💡 **Research Assistance**: Gather information from multiple sources automatically
- 🤖 **Agent Integration**: Build AI workflows with reliable, up-to-date information
- 🔗 **Citation Management**: Track and display source attributions automatically

## Features

### Core Capabilities

- **Real-Time Web Search**: Automatic search execution with multi-query support
- **Automatic Citations**: Inline citations with source URIs and titles
- **Model Selection**: Choose between gemini-2.5-flash (fast) or gemini-2.5-pro (powerful)
- **Flexible Input**: Positional arguments or stdin for automation
- **Multiple Output Formats**: JSON (default) or markdown with citations
- **Verbose Metadata**: Full grounding details including search queries and supports

### CLI-First Design

- **Type Safety**: Strict mypy checking, comprehensive type hints
- **Error Handling**: Rich error messages with suggested fixes
- **Composability**: JSON to stdout, logs to stderr for piping
- **Documentation**: Agent-friendly help with inline examples
- **Testing**: Comprehensive test suite with pytest

## Installation

### Prerequisites

- Python 3.14 or higher
- [uv](https://github.com/astral-sh/uv) package manager (recommended)
- Google Gemini API key ([get one free](https://aistudio.google.com/app/apikey))

### Install Globally with uv

```bash
# Install from source
git clone https://github.com/dnvriend/gemini-google-search-tool.git
cd gemini-google-search-tool
uv tool install .

# Verify installation
gemini-google-search-tool --version
```

### Install with pip

```bash
pip install gemini-google-search-tool
```

### Install with mise (recommended for development)

```bash
cd gemini-google-search-tool
mise trust
mise install
uv sync
uv tool install .
```

### Shell Completion

Enable tab completion for your shell to improve the CLI experience:

**Bash (add to `~/.bashrc`):**
```bash
eval "$(gemini-google-search-tool completion bash)"
```

**Zsh (add to `~/.zshrc`):**
```bash
eval "$(gemini-google-search-tool completion zsh)"
```

**Fish (run once):**
```bash
mkdir -p ~/.config/fish/completions
gemini-google-search-tool completion fish > ~/.config/fish/completions/gemini-google-search-tool.fish
```

For more details:
```bash
gemini-google-search-tool completion --help
```

## Configuration

### API Key Setup

The tool requires a `GEMINI_API_KEY` environment variable.

**Option 1: Environment Variable**

```bash
export GEMINI_API_KEY='your-api-key-here'
```

**Option 2: macOS Keychain** (secure storage)

```bash
# Store in keychain
security add-generic-password -a "production" -s "GEMINI_API_KEY" -w "your-api-key"

# Retrieve and export
export GEMINI_API_KEY=$(security find-generic-password -a "production" -s "GEMINI_API_KEY" -w)
```

**Get a free API key**: [Google AI Studio](https://aistudio.google.com/app/apikey)

## Usage

### CLI Usage

#### Basic Query

```bash
gemini-google-search-tool query "Who won euro 2024?"
```

**Output:**
```json
{
  "response_text": "Spain won Euro 2024, defeating England 2-1 in the final...",
  "citations": [
    {"index": 1, "uri": "https://...", "title": "youtube.com"},
    {"index": 2, "uri": "https://...", "title": "wikipedia.org"}
  ]
}
```

Citations are always included in JSON output by default.

#### Query with Inline Citations

The `--add-citations` flag adds inline citations directly into the response text.

```bash
gemini-google-search-tool query "Latest AI developments" --add-citations
```

**Output:**
```json
{
  "response_text": "Recent AI developments include... ([1], [2])",
  "citations": [
    {"index": 1, "uri": "https://...", "title": "Source Title"},
    {"index": 2, "uri": "https://...", "title": "Another Source"}
  ]
}
```

#### Read from stdin

```bash
echo "Climate change updates" | gemini-google-search-tool query --stdin
```

#### Markdown Output

```bash
gemini-google-search-tool query "Quantum computing news" --text
```

**Output:**
```markdown
Recent advances in quantum computing include...

## Citations

1. [MIT Technology Review](https://...)
2. [Nature](https://...)
```

#### Verbose Output with Grounding Metadata

```bash
gemini-google-search-tool query "Latest tech trends" --verbose
```

**Output includes:**
- Response text
- Web search queries executed
- Grounding chunks with URIs
- Grounding supports (which parts of text are supported by which sources)

#### Use Pro Model

```bash
gemini-google-search-tool query "Analyze quantum computing impact" --pro
```

#### All Options Combined

```bash
gemini-google-search-tool query "Machine learning breakthroughs" \
  --add-citations \
  --pro \
  --verbose
```

### Library Usage

Import and use as a Python library:

```python
from gemini_google_search_tool import (
    GeminiClient,
    query_with_grounding,
    add_inline_citations,
)

# Initialize client
client = GeminiClient()  # Reads GEMINI_API_KEY from environment

# Query with grounding
response = query_with_grounding(
    client=client,
    prompt="Who won euro 2024?",
    model="gemini-2.5-flash",  # or "gemini-2.5-pro"
)

# Access response
print(response.response_text)
print(f"Citations: {len(response.citations)}")

# Access citations
for citation in response.citations:
    print(f"[{citation.index}] {citation.title}: {citation.uri}")

# Access metadata
if response.web_search_queries:
    print(f"Search queries: {response.web_search_queries}")

# Add inline citations
if response.grounding_segments:
    text_with_citations = add_inline_citations(
        response.response_text,
        response.grounding_segments,
        response.citations,
    )
    print(text_with_citations)
```

### Error Handling in Library

```python
from gemini_google_search_tool import GeminiClient, GeminiClientError, SearchError

try:
    client = GeminiClient()
    response = query_with_grounding(client, "Your query")
except GeminiClientError as e:
    print(f"Client error: {e}")
    # Handle authentication or configuration errors
except SearchError as e:
    print(f"Search error: {e}")
    # Handle query execution errors
```

## Development

### Setup Development Environment

```bash
# Clone repository
git clone https://github.com/dnvriend/gemini-google-search-tool.git
cd gemini-google-search-tool

# Install dependencies
make install

# Show available commands
make help
```

### Available Make Commands

```bash
make install          # Install dependencies
make format           # Format code with ruff
make lint             # Run linting with ruff
make typecheck        # Run type checking with mypy
make test             # Run tests with pytest
make check            # Run all checks (lint, typecheck, test)
make pipeline         # Full pipeline: format, check, build, install-global
make build            # Build package
make clean            # Remove build artifacts
```

### Project Structure

```
gemini-google-search-tool/
├── gemini_google_search_tool/
│   ├── __init__.py              # Public API exports
│   ├── cli.py                   # CLI entry point (Click group)
│   ├── core/                    # Core library (importable, CLI-independent)
│   │   ├── __init__.py
│   │   ├── client.py           # Gemini client management
│   │   └── search.py           # Search and citation logic
│   ├── commands/                # CLI command implementations
│   │   ├── __init__.py
│   │   └── query_commands.py   # Query command with Click decorators
│   └── utils.py                 # Shared utilities (output, validation)
├── tests/                       # Test suite
│   ├── __init__.py
│   └── test_utils.py
├── pyproject.toml               # Project configuration
├── Makefile                     # Development commands
├── README.md                    # User documentation (this file)
├── CLAUDE.md                    # Developer guide
└── LICENSE                      # MIT License
```

### Architecture Principles

- **Separation of Concerns**: Core library (`core/`) is independent of CLI
- **Exception-Based Errors**: Core functions raise exceptions (not `sys.exit`), CLI handles formatting
- **Importable Library**: Public API exposed via `__init__.py` for programmatic use
- **Type Safety**: Strict mypy checks, comprehensive type hints
- **Composability**: JSON to stdout, logs to stderr for piping

## Testing

Run the test suite:

```bash
# Run all tests
make test

# Run with verbose output
uv run pytest tests/ -v

# Run specific test file
uv run pytest tests/test_utils.py

# Run with coverage
uv run pytest tests/ --cov=gemini_google_search_tool
```

## Resources

### Official Documentation

- [Google Search Grounding Documentation](https://ai.google.dev/gemini-api/docs/grounding)
- [Gemini API Documentation](https://ai.google.dev/gemini-api/docs)
- [Google AI Studio](https://aistudio.google.com/) - Get API keys and test models

### Related Tools

- [google-genai Python SDK](https://github.com/googleapis/python-genai) - Official Gemini Python client
- [Click](https://click.palletsprojects.com/) - CLI framework
- [uv](https://github.com/astral-sh/uv) - Fast Python package manager

## Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Run the full pipeline (`make pipeline`)
5. Commit your changes (`git commit -m 'Add amazing feature'`)
6. Push to the branch (`git push origin feature/amazing-feature`)
7. Open a Pull Request

### Code Style

- Follow PEP 8 guidelines (enforced by ruff)
- Use type hints for all functions (strict mypy)
- Write docstrings for public functions (Google style)
- Format code with `ruff format`
- Pass all checks: `make check`

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Author

**Dennis Vriend**

- GitHub: [@dnvriend](https://github.com/dnvriend)
- Email: dvriend@ilionx.com

## Acknowledgments

- Built with [Click](https://click.palletsprojects.com/) for CLI framework
- Developed with [uv](https://github.com/astral-sh/uv) for fast Python tooling
- Uses [google-genai](https://github.com/googleapis/python-genai) SDK for Gemini API access
- Quality assurance with [ruff](https://github.com/astral-sh/ruff) and [mypy](https://github.com/python/mypy)

---

**Generated with AI**

This project was generated using [Claude Code](https://www.anthropic.com/claude/code), an AI-powered development tool by [Anthropic](https://www.anthropic.com/). Claude Code assisted in creating the project structure, implementation, tests, documentation, and development tooling.

Made with ❤️ using Python 3.14
