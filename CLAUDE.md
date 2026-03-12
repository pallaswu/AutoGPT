# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Auto-GPT is an experimental open-source Python application that autonomously chains together LLM "thoughts" to achieve user-defined goals using GPT-4.

## Common Commands

### Setup and Run
```bash
# Install dependencies
pip install -r requirements.txt

# Run the application
python -m autogpt

# Or use the shell script
./run.sh

# Run with specific options
python -m autogpt --continuous --speak --debug
```

### Testing
```bash
# Run all tests with coverage
pytest --cov=autogpt --cov-report term-missing

# Run unit tests only (without integration/slow)
pytest --without-integration --without-slow-integration

# Run a specific test file
pytest tests/test_agent.py

# Run a specific test function
pytest tests/test_agent.py::test_function_name

# Run integration tests
pytest tests/integration/
```

### Linting and Formatting
```bash
# Check formatting
black . --check
isort . --check

# Apply formatting
black .
isort .

# Run flake8 linting
flake8

# Install and run pre-commit hooks
pre-commit install
pre-commit run --all-files
```

### Docker
```bash
# Run with Docker Compose
docker-compose up

# Build Docker image
docker build -t auto-gpt .
```

## Architecture Overview

### Entry Points
- `autogpt/__main__.py` - Package entry point, delegates to `cli.py`
- `autogpt/cli.py` - Click CLI interface with options (--continuous, --speak, --debug, etc.)
- `autogpt/main.py` - Main application orchestration, initializes all components and starts the agent

### Core Components

**Agent System (`autogpt/agent/`)**
- `agent.py` - Main Agent class with the interaction loop that processes thoughts, commands, and actions
- `agent_manager.py` - Manages multiple sub-agents that can be created, messaged, and deleted

**Command System (`autogpt/commands/`)**
- `command.py` - Command registry and decorator for registering commands
- Command implementations:
  - `analyze_code.py` - Code analysis using LLM
  - `execute_code.py` - Python and shell code execution
  - `file_operations.py` - File read/write/append operations
  - `google_search.py` - Web search via Google or DuckDuckGo
  - `image_gen.py` - Image generation
  - `web_selenium.py` - Browser automation with Selenium
  - `git_operations.py` - Git commands
  - `twitter.py` - Twitter integration
  - `app.py` - Command orchestration and execution logic

**Memory System (`autogpt/memory/`)**
Pluggable memory backends with a common base interface:
- `local.py` - Local JSON file cache
- `pinecone.py` - Pinecone vector database
- `redismem.py` - Redis backend
- `weaviate.py` - Weaviate vector search engine
- `milvus.py` - Milvus vector database
- `no_memory.py` - No-op memory (disabled)

Configuration via `MEMORY_BACKEND` env variable.

**Configuration (`autogpt/config/`)**
- `config.py` - Singleton Config class, loads from `.env` and environment variables
- `ai_config.py` - AI personality/role configuration loaded from `ai_settings.yaml`

**LLM Integration**
- `llm_utils.py` - OpenAI API wrapper, token counting, and rate limiting
- `api_manager.py` - API key rotation and cost tracking
- `token_counter.py` - Token usage estimation for models

**Plugins (`autogpt/plugins.py`)**
Dynamic plugin loading system that scans for and loads external plugins at runtime.

**Prompts (`autogpt/prompts/`)**
- `prompt.py` - Constructs system prompts with available commands and constraints
- `generator.py` - PromptGenerator for building command documentation

**Workspace (`autogpt/workspace/`)**
Sandboxed file operations to keep agent file access contained.

## Key Configuration

The application requires a `.env` file in the project root with:
- `OPENAI_API_KEY` - Required for operation
- Optional: `MEMORY_BACKEND`, `PINECONE_API_KEY`, `REDIS_HOST`, etc.

See `.env.template` for all available options.

## Development Notes

- Python 3.10+ is required
- Code style: Black (line-length 88), isort for imports, flake8 for linting
- Pre-commit hooks run tests automatically
- The `stable` branch is recommended for production use
- New non-essential commands should be implemented as plugins, not core commands

## Testing Structure

- `tests/unit/` - Unit tests for individual components
- `tests/integration/` - Integration tests requiring external services
- `tests/mocks/` - Mock implementations for testing
- `tests/conftest.py` - Pytest fixtures and configuration
- Use `@pytest.mark.vcr` for recording HTTP interactions
