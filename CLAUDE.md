# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a RAG (Retrieval-Augmented Generation) chatbot for querying course materials. It uses ChromaDB for vector storage, Anthropic's Claude for AI responses, and sentence-transformers for embeddings.

## Commands

**Utiliza siempre `uv` y no `pip` para correr este proyecto.**



```bash
# Install dependencies
uv sync

# Run the application (from project root, requires Git Bash on Windows)
./run.sh

# Or manually start the server
cd backend && uv run uvicorn app:app --reload --port 8000
```

The application runs at `http://localhost:8000` with API docs at `/docs`.

## Architecture

### Data Flow
1. Course documents (`.txt` files in `/docs`) are parsed by `DocumentProcessor`
2. Documents are chunked and stored in ChromaDB via `VectorStore`
3. User queries come through FastAPI endpoints in `app.py`
4. `RAGSystem` orchestrates the query: it uses `AIGenerator` to call Claude with tool-use
5. Claude calls `search_course_content` tool which queries `VectorStore`
6. Results are formatted and returned to the user

### Key Components (backend/)

- **RAGSystem** (`rag_system.py`): Main orchestrator that wires together all components
- **VectorStore** (`vector_store.py`): ChromaDB wrapper with two collections:
  - `course_catalog`: Course metadata for semantic course name matching
  - `course_content`: Chunked course content for search
- **AIGenerator** (`ai_generator.py`): Claude API client with tool execution loop
- **DocumentProcessor** (`document_processor.py`): Parses course files with expected format:
  ```
  Course Title: [title]
  Course Link: [url]
  Course Instructor: [name]
  Lesson 1: [title]
  Lesson Link: [url]
  [content...]
  ```
- **CourseSearchTool** (`search_tools.py`): Tool definition for Claude's tool-use, wraps VectorStore.search()

### Frontend
Static HTML/JS/CSS served from `/frontend`. No build step required.

## Configuration

Settings in `backend/config.py`:
- `CHUNK_SIZE`: 800 characters
- `CHUNK_OVERLAP`: 100 characters
- `MAX_RESULTS`: 5 search results
- `EMBEDDING_MODEL`: all-MiniLM-L6-v2

Requires `ANTHROPIC_API_KEY` in `.env` file at project root.

## Data Models

- **Course**: title, course_link, instructor, lessons[]
- **Lesson**: lesson_number, title, lesson_link
- **CourseChunk**: content, course_title, lesson_number, chunk_index
