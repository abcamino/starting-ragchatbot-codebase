# CLAUDE.local.md

This file provides guidance to Claude Code (claude.ai/code) when working in the backend directory.

## Running the Server

```bash
# From this directory
uv run uvicorn app:app --reload --port 8000

# From project root
cd backend && uv run uvicorn app:app --reload --port 8000
```

## Module Responsibilities

| Module | Purpose |
|--------|---------|
| `app.py` | FastAPI routes: `/api/query`, `/api/courses` |
| `rag_system.py` | Wires all components, entry point for queries |
| `vector_store.py` | ChromaDB operations, two collections |
| `ai_generator.py` | Claude API calls with tool-use loop |
| `document_processor.py` | Parse `.txt` files, chunk text |
| `search_tools.py` | Tool definitions for Claude |
| `session_manager.py` | Conversation history per session |
| `config.py` | All settings, loads `.env` |
| `models.py` | Pydantic models: Course, Lesson, CourseChunk |

## Key Patterns

### Adding a New Tool
1. Create class extending `Tool` in `search_tools.py`
2. Implement `get_tool_definition()` returning Anthropic tool schema
3. Implement `execute(**kwargs)` returning string result
4. Register in `RAGSystem.__init__`: `self.tool_manager.register_tool(YourTool())`

### ChromaDB Collections
- `course_catalog`: One document per course (title as ID), metadata includes lessons JSON
- `course_content`: One document per chunk, filtered by `course_title` and `lesson_number`

### Document Format Expected
```
Course Title: [title]
Course Link: [url]
Course Instructor: [name]

Lesson 0: [title]
Lesson Link: [url]
[content...]

Lesson 1: [title]
...
```

## API Endpoints

- `POST /api/query` - Query with `{query, session_id?}`, returns `{answer, sources, session_id}`
- `GET /api/courses` - Returns `{total_courses, course_titles[]}`

## Database Location

ChromaDB persists to `./chroma_db/`. Delete this folder to reset all course data.
