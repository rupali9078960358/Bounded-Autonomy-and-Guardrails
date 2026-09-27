# Claude Multi-Agent Code Review System

A production-ready, multi-agent code review system powered by the Claude Agent SDK that provides comprehensive automated code reviews for GitHub pull requests.

## Overview

This system uses three specialized AI agents to analyze pull requests from multiple angles:

- **Code Quality Analyzer**: Identifies security vulnerabilities, performance issues, and maintainability concerns
- **Test Coverage Analyzer**: Evaluates test completeness and suggests specific test cases
- **Refactoring Suggester**: Identifies opportunities to improve code structure and modernize patterns

The orchestrator coordinates these agents and aggregates their findings into professional HTML, Markdown, and JSON reports.

## Features

- ✨ **Multi-Agent Architecture**: Three specialized subagents analyze code from different perspectives
- 🔒 **Security-First**: Identifies SQL injection, XSS, hardcoded secrets, and OWASP Top 10 vulnerabilities
- 🧪 **Smart Test Suggestions**: Analyzes coverage gaps and suggests specific, actionable test cases
- 🔄 **Refactoring Guidance**: Provides before/after examples for modernization opportunities
- 📊 **Multiple Output Formats**: Generates HTML, Markdown, and JSON reports
- 🛡️ **Production-Ready**: Includes error handling, rate limiting, and retry logic
- 🎯 **Structured Outputs**: Uses Zod schemas for runtime validation

## Prerequisites

- Node.js >= 18
- npm
- Anthropic API key or AWS Bedrock credentials
- GitHub Personal Access Token (recommended for higher rate limits)

## Installation

### 1. Install Dependencies

```bash
# From the repository root
cd /voc/work/cd14715-claude-code-classroom
npm install

# Navigate to project directory
cd project/starter
```

### 2. Configure Environment Variables

```bash
# Copy the example environment file
cp .env.example .env
```

Edit `.env` and configure:

```bash
# Model Configuration (REQUIRED)
ANTHROPIC_MODEL=claude-sonnet-4-5-20250929

# Project Root (REQUIRED)
PROJECT_ROOT=/voc/work/cd14715-claude-code-classroom/project/starter

# GitHub Token (RECOMMENDED for higher rate limits)
GITHUB_TOKEN=ghp_your-token-here

# Logging Level (optional)
LOG_LEVEL=info
```

### 3. Authentication

**Option 1: Anthropic API (Vocareum Workspace)**

In the Vocareum workspace, authentication is pre-configured:

```bash
# Already set in environment:
# ANTHROPIC_API_KEY
# ANTHROPIC_BASE_URL
```

**Option 2: AWS Bedrock**

Configure AWS credentials:

```bash
export AWS_ACCESS_KEY_ID=your-access-key
export AWS_SECRET_ACCESS_KEY=your-secret-key
export AWS_REGION=us-east-1
export ANTHROPIC_MODEL=us.anthropic.claude-sonnet-4-5-20250929-v1:0
```

## Usage

### Basic Usage

```bash
npm run dev <owner> <repo> <pr-number>
```

**Examples:**

```bash
# Review PR #1 from airaamane/simple-todo-app
npm run dev airaamane simple-todo-app 1

# Review PR #2
npm run dev airaamane simple-todo-app 2

# Review PR #3
npm run dev airaamane simple-todo-app 3
```

### Build for Production

```bash
npm run build
npm start airaamane simple-todo-app 1
```

### Run Tests

```bash
npm test
```

### Output Location

Reports are saved to the `reports/` directory:

```
reports/
├── airaamane-simple-todo-app-pr1.json
├── airaamane-simple-todo-app-pr1.md
├── airaamane-simple-todo-app-pr1.html
├── airaamane-simple-todo-app-pr2.json
├── airaamane-simple-todo-app-pr2.md
└── ...
```

## Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    MAIN ORCHESTRATOR                            │
│         (Coordinates analysis, aggregates results)              │
└───────────────────────────┬─────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│ Code Quality  │   │ Test Coverage │   │  Refactoring  │
│   Analyzer    │   │   Analyzer    │   │   Suggester   │
└───────────────┘   └───────────────┘   └───────────────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            ▼
              ┌─────────────────────────┐
              │   ReviewReport (JSON)   │
              └─────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
   [report.md]        [report.html]       [report.json]
```

### Project Structure

```
src/
├── agents/                   # Subagent definitions
│   ├── code-quality-analyzer.ts
│   ├── test-coverage-analyzer.ts
│   ├── refactoring-suggester.ts
│   └── index.ts
├── prompts/                  # Agent prompts
│   ├── orchestrator.prompt.ts
│   ├── code-quality-analyzer.prompt.ts
│   ├── test-coverage-analyzer.prompt.ts
│   ├── refactoring-suggester.prompt.ts
│   └── index.ts
├── config/                   # MCP server configurations
│   └── mcp.config.ts
├── utils/                    # Utility functions
│   ├── error-handler.ts     # Retry and timeout logic
│   ├── rate-limiter.ts      # Token bucket rate limiting
│   ├── logger.ts            # Winston logging
│   ├── report-generator.ts  # Format converters
│   └── index.ts
├── types/                    # Zod schemas and types
│   ├── analysis-results.ts
│   ├── report-types.ts
│   └── index.ts
├── orchestrator.ts           # Main coordination logic
└── main.ts                   # CLI entry point
```

## Implementation Details

### MCP Configuration

Configured in `src/config/mcp.config.ts`:

- **GitHub MCP Server**: Fetches PR data, file diffs, repository information using `@modelcontextprotocol/server-github`
- **ESLint MCP Server**: Provides code linting capabilities using `@eslint/mcp`

### Subagents

**Code Quality Analyzer** (`src/agents/code-quality-analyzer.ts`):
- Security: SQL injection, XSS, CSRF, hardcoded secrets
- Performance: O(n²) algorithms, memory leaks, N+1 queries
- Best practices: TypeScript types, error handling, modern syntax
- Uses Claude Skills (javascript-best-practices) for domain expertise

**Test Coverage Analyzer** (`src/agents/test-coverage-analyzer.ts`):
- Identifies untested functions, branches, edge cases
- Estimates coverage by comparing source with test files
- Suggests specific test cases with assertions
- Prioritizes critical paths

**Refactoring Suggester** (`src/agents/refactoring-suggester.ts`):
- Extract function, rename, simplify patterns
- Modernization: callbacks → async/await, var → const/let
- Design pattern improvements
- Before/after code examples

### Error Handling & Resilience

**Error Handler** (`src/utils/error-handler.ts`):
- Exponential backoff retry (max 3 retries, 1-4s delays)
- Timeout wrapper using Promise.race
- Custom error codes for different failure modes

**Rate Limiter** (`src/utils/rate-limiter.ts`):
- Token bucket algorithm with 60-second sliding window
- Limits: 50 req/min, 100k tokens/min, 5 concurrent
- Prevents API throttling and cost overruns

## Configuration

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `ANTHROPIC_API_KEY` | Yes* | Anthropic API key (pre-configured in Vocareum) |
| `ANTHROPIC_BASE_URL` | No | Custom API endpoint |
| `ANTHROPIC_MODEL` | Yes | Model name (claude-sonnet-4-5-20250929) |
| `PROJECT_ROOT` | Yes | Absolute path to project/starter directory |
| `GITHUB_TOKEN` | Recommended | GitHub PAT for higher API rate limits |
| `LOG_LEVEL` | No | Logging verbosity (error, warn, info, debug) |

*Or AWS credentials if using Bedrock

## Troubleshooting

### Common Issues

**"ANTHROPIC_MODEL environment variable is required"**
```bash
# Add to .env file:
ANTHROPIC_MODEL=claude-sonnet-4-5-20250929
```

**"No authentication configured"**
```bash
# In Vocareum, check environment:
echo $ANTHROPIC_API_KEY | head -c 10

# For local development:
export ANTHROPIC_API_KEY=sk-ant-your-key-here
```

**GitHub rate limit errors**
```bash
# Add GitHub token to .env:
GITHUB_TOKEN=ghp_your-token-here
```

**TypeScript compilation errors**
```bash
# Check for errors:
npm run build

# Common fixes:
# - Ensure all imports use .js extensions
# - Check agent definitions match SDK types
# - Verify Zod schemas are correctly defined
```

## Development

### Code Style

- ES Modules with `.js` extensions in imports
- TypeScript for type safety
- Zod for runtime validation
- Winston for structured logging
- Async/await for asynchronous operations

### Testing

```bash
# Run all tests
npm test

# Watch mode
npm run test:watch

# TypeScript linting
npm run lint
```

## Deliverables

This project generates the following deliverables:

1. **Implementation Files**:
   - 3 subagent definitions with prompts
   - MCP configuration for GitHub and ESLint
   - Main orchestrator with structured outputs
   - Error handling and rate limiting utilities
   - Complete CLI application

2. **PR Analysis Reports** (in `reports/`):
   - 9 total reports (3 PRs × 3 formats each)
   - JSON, Markdown, and HTML formats
   - For PRs #1, #2, #3 from airaamane/simple-todo-app

## License

ISC

## Acknowledgments

- Built with [Claude Agent SDK](https://github.com/anthropics/anthropic-sdk-typescript)
- Uses [Model Context Protocol](https://modelcontextprotocol.io) for tool integration
- Powered by Claude Sonnet 4.5 from Anthropic
- Course project for Udacity Claude Code Program

6. **Error Handler** (Recommended) (`src/utils/error-handler.ts`)
   - Custom `ReviewError` class
   - Retry logic with exponential backoff
   - Timeout wrapper

7. **Rate Limiter** (Optional) (`src/utils/rate-limiter.ts`)
   - Token bucket algorithm with sliding window
   - Request and token tracking
   - Concurrent request management

## Getting Started

### Prerequisites

- Node.js 18+
- Anthropic API access (provided in Vocareum workspace) or [your own API key](https://console.anthropic.com/)
- [GitHub Personal Access Token](https://github.com/settings/tokens) (recommended - scopes: `repo`, `read:org`)

### Installation

**In Vocareum Workspace (Recommended):**

Your workspace comes pre-configured with Anthropic API credentials.

```bash
# Install dependencies from repository root (uses npm workspaces)
cd /voc/work/cd14715-claude-code-classroom
npm install

# Navigate to project and configure
cd project/starter
cp .env.example .env
```

**Local Setup:**

```bash
# Clone the repository
git clone https://github.com/udacity/cd14715-claude-code-classroom.git
cd cd14715-claude-code-classroom/project/starter

# Install dependencies
npm install

# Configure environment
cp .env.example .env
```

### Configuration

Edit `.env` with your settings:

**In Vocareum Workspace:**
```bash
# API credentials are already in your environment - don't add them here

# Model Configuration (REQUIRED)
ANTHROPIC_MODEL=claude-sonnet-4-5-20250929

# Project root (REQUIRED)
PROJECT_ROOT=/voc/work/cd14715-claude-code-classroom/project/starter

# GitHub Token (RECOMMENDED for higher rate limits)
# GITHUB_TOKEN=ghp_your-token-here

# Logging level (optional)
LOG_LEVEL=info
```

**Local Setup with Your Own API Key:**
```bash
# Your Anthropic API key
ANTHROPIC_API_KEY=sk-ant-your-key-here

# Model Configuration (REQUIRED)
ANTHROPIC_MODEL=claude-sonnet-4-5-20250929

# Project root (REQUIRED - update to your path)
PROJECT_ROOT=/absolute/path/to/project/starter

# GitHub Token (RECOMMENDED)
# GITHUB_TOKEN=ghp_your-token-here

# Logging level (optional)
LOG_LEVEL=info
```

### Running

```bash
# Development mode
npm run dev -- <owner> <repo> <pr-number>

# Production build
npm run build
npm start <owner> <repo> <pr-number>

# Example
npm run dev -- facebook react 12345
```

### Testing

```bash
# Run all tests
npm test

# Run specific test
npm test -- orchestrator.test.ts

# Watch mode
npm test -- --watch
```

## Key Technologies

- **Claude Agent SDK** - Multi-agent orchestration framework
- **Model Context Protocol (MCP)** - External data integration
- **Zod** - Schema validation and type safety
- **TypeScript** - Type-safe development
- **Vitest** - Testing framework
- **Winston** - Structured logging

## Success Criteria

Your implementation is complete when:

- [ ] TypeScript compiles without errors: `npm run build`
- [ ] All tests pass: `npm test`
- [ ] Can review a real PR: `npm start owner repo pr-number`
- [ ] Generates reports in at least one format (MD, HTML, JSON)
- [ ] Rate limiting prevents API throttling (Optional)
- [ ] Errors are handled gracefully (Recommended)

## Resources

- [Claude Agent SDK](https://github.com/anthropics/claude-agent-sdk)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [Anthropic API Docs](https://docs.anthropic.com/)
- [Zod Documentation](https://zod.dev/)

Good luck!