# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GPT Researcher is an autonomous deep research agent that conducts web and local document research on any topic, generating comprehensive reports with citations. It uses a multi-agent architecture with planner, execution, and publisher agents to produce factual, unbiased research.

**Repository**: https://github.com/assafelovic/gpt-researcher
**Documentation**: https://docs.gptr.dev/

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Research Pipeline                         │
├─────────────────────────────────────────────────────────────┤
│  Planner Agent → Execution Agents → Publisher Agent          │
│  (questions)      (gather info)      (aggregate report)      │
└─────────────────────────────────────────────────────────────┘
```

**Core Components:**
- `/gpt_researcher/agent.py` - Main `GPTResearcher` class orchestrating research workflow
- `/gpt_researcher/skills/` - Specialized capabilities (researcher, writer, browser, curator, deep_research)
- `/gpt_researcher/retrievers/` - 16 search backends (tavily, duckduckgo, arxiv, bing, google, semantic_scholar, pubmed_central, etc.)
- `/gpt_researcher/scraper/` - 9 web scraping strategies (beautifulsoup, firecrawl, browser, pymupdf, etc.)
- `/backend/server/app.py` - FastAPI REST API with WebSocket support for real-time updates
- `/frontend/nextjs/` - Production Next.js frontend
- `/multi_agents/` - LangGraph-based multi-agent system (Browser, Editor, Researcher, Reviewer, Revisor, Writer, Publisher)

## Common Commands

### Backend Development
```bash
# Install dependencies
pip install -r requirements.txt

# Start development server with auto-reload
python -m uvicorn main:app --reload

# Start production server
python main.py
# or
python -m uvicorn main:app --host=0.0.0.0 --port=8000

# Run CLI research
python cli.py "Your research query" --report_type research_report --tone objective
```

### Frontend Development
```bash
cd frontend/nextjs
npm install
npm run dev      # Development server on port 3000
npm run build    # Production build
npm run lint     # ESLint check
```

### Docker
```bash
docker compose up -d                           # Start backend (8001) + frontend (3001)
docker compose up -d gpt-researcher            # Backend only
docker compose run gpt-researcher-tests        # Run test suite
docker logs gpt-researcher -f                  # View logs
```

### Testing
```bash
# Run all tests
python -m pytest tests/

# Run specific test files
python -m pytest tests/research_test.py -v
python -m pytest tests/report-types.py
python -m pytest tests/vector-store.py

# Test your configuration
python tests/test-your-llm.py
python tests/test-your-retriever.py
python tests/test-your-embeddings.py
```

## Configuration

### Key Environment Variables (.env)
```bash
# LLM Providers
OPENAI_API_KEY=           # OpenAI
GOOGLE_API_KEY=           # Gemini
OPENAI_BASE_URL=          # Custom OpenAI-compatible API

# LLM Model Selection
FAST_LLM=gpt-4o-mini      # Quick operations
SMART_LLM=gpt-4o          # Complex reasoning
STRATEGIC_LLM=o1-preview  # Deep research planning

# Search/Retrieval
TAVILY_API_KEY=           # Tavily search (recommended)
RETRIEVER=tavily          # Options: tavily, duckduckgo, bing, google, arxiv, etc.
RETRIEVER=tavily,mcp      # Hybrid: web + MCP sources

# Embeddings
EMBEDDING=openai:text-embedding-3-small

# Local Documents
DOC_PATH=./my-docs        # Directory for local document research

# Performance
MAX_SCRAPER_WORKERS=10    # Concurrent scraper limit
SCRAPER_RATE_LIMIT_DELAY=0.5  # Rate limiting delay (seconds)
```

### Report Types
- `research_report` - Quick summary (~2 min)
- `detailed_report` - In-depth analysis (~5 min)
- `deep_research` - Recursive exploration with branching (~5 min)
- `resource_report`, `outline_report`, `subtopic_report` - Specialized formats

## Python Package API

```python
from gpt_researcher import GPTResearcher

researcher = GPTResearcher(
    query="Why is Nvidia stock going up?",
    report_type="research_report",  # or detailed_report, deep_research
    report_format="markdown",       # or pdf, docx
    tone="objective"                # or formal, analytical, etc.
)
research_result = await researcher.conduct_research()
report = await researcher.write_report()
```

## Key Patterns

- **Async-First**: Uses asyncio throughout for concurrent operations
- **WebSocket Streaming**: Real-time progress updates via `WebSocketManager`
- **Retriever Pluggability**: Add new search backends via the retriever interface in `/gpt_researcher/retrievers/`
- **LLM Provider Abstraction**: Supports OpenAI, LiteLLM, Ollama, Google, Anthropic via `/gpt_researcher/llm_provider/`
- **MCP Integration**: Model Context Protocol support for specialized data sources (GitHub, databases, custom APIs)

## Code Style

- TypeScript: Strict mode enabled for frontend
- ESLint + Prettier configured for frontend
- Tailwind CSS for styling
- Self-documenting code preferred over comments
