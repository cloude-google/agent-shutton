# CLAUDE.md - AI Assistant Guide for Agent Shutton

## Project Overview

**Agent Shutton** is a multi-agent blog writing system built with Google Agent Development Kit (ADK). It automates the creation of technical blog posts through a modular, collaborative architecture where specialized AI agents handle different aspects of content creation.

**Key Purpose**: Streamline technical blog writing by automating research, outlining, writing, editing, and social media promotion while maintaining human oversight and feedback loops.

**Tech Stack**:
- Python 3.11.3+
- Google ADK 1.18.0
- Gemini 2.5 Pro/Flash models
- pytest for testing

## Architecture Overview

This is a **multi-agent system** with hierarchical delegation:

```
interactive_blogger_agent (orchestrator)
├── robust_blog_planner (LoopAgent)
│   ├── blog_planner (content strategist)
│   └── OutlineValidationChecker (validator)
├── robust_blog_writer (LoopAgent)
│   ├── blog_writer (technical writer)
│   └── BlogPostValidationChecker (validator)
├── blog_editor (editor)
└── social_media_writer (marketer)
```

**Agent Pattern**: The root agent (`interactive_blogger_agent`) acts as a project manager, delegating specialized tasks to sub-agents and maintaining an iterative workflow with user feedback loops.

## Directory Structure

```
/home/user/agent-shutton/
├── blogger_agent/              # Main Python package
│   ├── __init__.py            # Package exports (root_agent)
│   ├── agent.py               # Main orchestrator agent definition
│   ├── config.py              # Model configuration and settings
│   ├── tools.py               # Custom tools (file saving, codebase analysis)
│   ├── agent_utils.py         # Helper functions (output suppression)
│   ├── validation_checkers.py # Quality validation agents
│   └── sub_agents/            # Specialized sub-agents
│       ├── __init__.py        # Sub-agent exports
│       ├── blog_planner.py    # Outline generation agent
│       ├── blog_writer.py     # Content writing agent
│       ├── blog_editor.py     # Revision agent
│       └── social_media_writer.py  # Promotion content agent
├── tests/
│   └── test_agent.py          # Integration test
├── requirements.txt           # Python dependencies
├── README.md                  # Project documentation
├── .gitignore                 # Git ignore patterns
└── LICENSE                    # Apache 2.0 license
```

## Key Components

### 1. Main Orchestrator Agent

**File**: `blogger_agent/agent.py`

**Agent**: `interactive_blogger_agent` (exported as `root_agent`)
- **Model**: `gemini-2.5-flash` (worker_model)
- **Role**: Coordinates the entire blog creation workflow
- **Tools**: `save_blog_post_to_file`, `analyze_codebase`
- **Sub-agents**: All four specialized agents
- **Output Key**: `blog_outline`

**Workflow**:
1. Plan → Refine outline with user feedback
2. Choose visual content strategy
3. Write → Present draft
4. Edit → Iterate based on feedback
5. Generate social media posts (optional)
6. Export to markdown file

### 2. Sub-Agents

#### Blog Planner (`blog_planner.py`)

**Base Agent**: `blog_planner`
- **Model**: `gemini-2.5-flash`
- **Role**: Content strategist - creates structured outlines
- **Tools**: `google_search`
- **Output Key**: `blog_outline`
- **Special Feature**: Reads `codebase_context` from session state

**Robust Wrapper**: `robust_blog_planner` (LoopAgent)
- **Max Iterations**: 3
- **Validation**: `OutlineValidationChecker`
- **Pattern**: Retries until valid outline is generated

#### Blog Writer (`blog_writer.py`)

**Base Agent**: `blog_writer`
- **Model**: `gemini-2.5-pro` (critic_model) - uses more powerful model
- **Role**: Expert technical writer for sophisticated audiences
- **Tools**: `google_search`
- **Output Key**: `blog_post`
- **Audience**: Similar to "Towards Data Science" and "freeCodeCamp" readers

**Robust Wrapper**: `robust_blog_writer` (LoopAgent)
- **Max Iterations**: 3
- **Validation**: `BlogPostValidationChecker`

#### Blog Editor (`blog_editor.py`)

**Agent**: `blog_editor`
- **Model**: `gemini-2.5-pro` (critic_model)
- **Role**: Professional technical editor
- **Output Key**: `blog_post`
- **Purpose**: Iterative revision based on user feedback

#### Social Media Writer (`social_media_writer.py`)

**Agent**: `social_media_writer`
- **Model**: `gemini-2.5-pro` (critic_model)
- **Role**: Social media marketing expert
- **Output Key**: `social_media_posts`
- **Platforms**: Twitter (short, engaging) and LinkedIn (professional)

### 3. Tools (`tools.py`)

**`save_blog_post_to_file(blog_post: str, filename: str) -> dict`**
- Exports final blog post to markdown file
- Returns: `{"status": "success"}`

**`analyze_codebase(directory: str) -> dict`**
- Recursively scans directory for all files
- Concatenates content into `codebase_context` string
- Handles encoding issues (UTF-8 fallback to latin-1)
- Returns: `{"codebase_context": <string>}`

### 4. Validation System (`validation_checkers.py`)

**Pattern**: Custom `BaseAgent` implementations that check session state

**`OutlineValidationChecker`**
- Checks for `blog_outline` in session state
- Escalates (exits loop) if valid, otherwise retries

**`BlogPostValidationChecker`**
- Checks for `blog_post` in session state
- Escalates if valid, otherwise retries

**Key Mechanism**: `EventActions(escalate=True)` signals LoopAgent to proceed

### 5. Configuration (`config.py`)

**`ResearchConfiguration` dataclass**:
```python
critic_model: str = "gemini-2.5-pro"      # High-quality tasks (writing, editing)
worker_model: str = "gemini-2.5-flash"    # Fast tasks (planning, orchestration)
max_search_iterations: int = 5
```

**Environment Variables**:
- `GOOGLE_CLOUD_PROJECT`: Auto-detected or set manually
- `GOOGLE_CLOUD_LOCATION`: Default "global"
- `GOOGLE_GENAI_USE_VERTEXAI`: Default "True"
- Optional: `GOOGLE_API_KEY` for AI Studio (requires `GOOGLE_GENAI_USE_VERTEXAI=FALSE`)

### 6. Utilities (`agent_utils.py`)

**`suppress_output_callback(callback_context: CallbackContext) -> Content`**
- Used in `after_agent_callback` for sub-agents
- Prevents intermediate agent outputs from showing to user
- Returns empty `Content()` object

## Development Workflows

### Running the Agent

**ADK Web Mode** (Interactive UI):
```bash
adk web
```
This starts the ADK web interface for interactive testing.

**Integration Test**:
```bash
python -m tests.test_agent
```

### Test Workflow (`tests/test_agent.py`)

The integration test simulates a complete blog creation workflow:
1. "I want to write a blog post about the new features in the latest version of the ADK."
2. "looks good, write it" (approve outline)
3. "1" (choose visual content option: placeholders)
4. "looks good, I approve" (approve draft)
5. "yes" (generate social media posts)
6. "my_new_blog_post.md" (save with filename)

**Key Components**:
- Uses `InMemorySessionService` for state management
- `Runner` class executes agent with session context
- Processes events asynchronously

### Installation

```bash
# Create virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # or `venv\Scripts\activate` on Windows

# Install dependencies
pip install -r requirements.txt
```

## Code Conventions

### 1. Agent Definition Pattern

```python
from google.adk.agents import Agent
from .config import config

my_agent = Agent(
    name="agent_name",
    model=config.worker_model,  # or config.critic_model
    description="Brief agent description",
    instruction="""
    Detailed multi-line instructions for agent behavior.
    Include role, task, output format, and special considerations.
    """,
    tools=[...],  # Optional tools
    sub_agents=[...],  # Optional sub-agents
    output_key="state_key_name",  # Where to store output in session state
    after_agent_callback=suppress_output_callback,  # Optional
)
```

### 2. LoopAgent Pattern (Robust Retry)

```python
from google.adk.agents import LoopAgent

robust_agent = LoopAgent(
    name="robust_agent_name",
    description="Description of retry behavior",
    sub_agents=[
        base_agent,
        ValidationChecker(name="validation_checker"),
    ],
    max_iterations=3,
    after_agent_callback=suppress_output_callback,
)
```

**How it works**:
1. LoopAgent runs `base_agent`
2. Runs `ValidationChecker`
3. If validator escalates → exit loop
4. If validator doesn't escalate → retry (up to max_iterations)

### 3. Validation Checker Pattern

```python
from google.adk.agents import BaseAgent
from google.adk.events import Event, EventActions

class MyValidationChecker(BaseAgent):
    async def _run_async_impl(self, context: InvocationContext) -> AsyncGenerator[Event, None]:
        if context.session.state.get("expected_key"):
            # Valid - escalate to exit loop
            yield Event(author=self.name, actions=EventActions(escalate=True))
        else:
            # Invalid - don't escalate, LoopAgent will retry
            yield Event(author=self.name)
```

### 4. Tool Definition Pattern

```python
def my_tool(param1: str, param2: int) -> dict:
    """Tool description for agent to understand usage."""
    # Tool implementation
    return {"key": "value"}  # Return dict for session state
```

Wrap with `FunctionTool`:
```python
from google.adk.tools import FunctionTool

tools=[FunctionTool(my_tool)]
```

### 5. Session State Usage

**Writing to state**: Use `output_key` parameter in Agent definition
```python
output_key="blog_outline"  # Agent output stored in session.state["blog_outline"]
```

**Reading from state**: Reference in instructions
```python
instruction="""
The outline is available in the `blog_outline` state key.
The codebase context is in the `codebase_context` state key.
"""
```

### 6. Copyright Headers

All Python files include Apache 2.0 copyright header:
```python
# Copyright 2025 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License")
# ...
```

## Model Selection Strategy

| Task Type | Model | Rationale |
|-----------|-------|-----------|
| Orchestration | `gemini-2.5-flash` | Fast decision-making, cost-effective |
| Planning | `gemini-2.5-flash` | Quick outline generation with retries |
| Writing | `gemini-2.5-pro` | High-quality technical content |
| Editing | `gemini-2.5-pro` | Nuanced revision and refinement |
| Social Media | `gemini-2.5-pro` | Marketing requires quality copywriting |

**Principle**: Use Flash for speed, Pro for quality.

## Extension Points

### Adding a New Sub-Agent

1. **Create agent file** in `blogger_agent/sub_agents/my_new_agent.py`:
```python
from google.adk.agents import Agent
from ..config import config

my_new_agent = Agent(
    name="my_new_agent",
    model=config.worker_model,
    description="Agent description",
    instruction="...",
    output_key="my_output",
)
```

2. **Export in** `blogger_agent/sub_agents/__init__.py`:
```python
from .my_new_agent import my_new_agent
```

3. **Import and add to main agent** in `blogger_agent/agent.py`:
```python
from .sub_agents import (..., my_new_agent)

interactive_blogger_agent = Agent(
    ...
    sub_agents=[..., my_new_agent],
)
```

4. **Update workflow instructions** to include when to delegate to new agent

### Adding a New Tool

1. **Define function** in `blogger_agent/tools.py`:
```python
def my_new_tool(param: str) -> dict:
    """Tool description."""
    # Implementation
    return {"result": "value"}
```

2. **Import and wrap in agent**:
```python
from .tools import my_new_tool
from google.adk.tools import FunctionTool

tools=[FunctionTool(my_new_tool)]
```

### Adding Google Search to New Agents

```python
from google.adk.tools import google_search

my_agent = Agent(
    ...
    tools=[google_search],
)
```

## Common AI Assistant Tasks

### When Modifying Agent Instructions

1. **Read the current agent file** to understand existing behavior
2. **Preserve the workflow structure** unless explicitly asked to change it
3. **Maintain consistency** in tone and instruction format
4. **Update related agents** if changes affect workflow dependencies
5. **Test with integration test** after changes

### When Debugging Agent Behavior

1. **Check session state keys** - ensure output_key matches references in other agents
2. **Verify LoopAgent validation** - ensure validators check correct state keys
3. **Review agent instructions** - confirm they reference available tools and state
4. **Check model selection** - ensure appropriate model for task complexity
5. **Test workflow sequence** - ensure agents are called in correct order

### When Adding Features

1. **Determine placement**:
   - New capability → new sub-agent
   - New data source → new tool
   - New validation → new checker
2. **Follow existing patterns** (see Code Conventions)
3. **Update main agent instructions** to incorporate new feature in workflow
4. **Add to integration test** if it changes the workflow
5. **Update README.md** with user-facing documentation

### When Refactoring

1. **Preserve public interface** - `root_agent` export must remain
2. **Maintain session state keys** - other agents depend on these
3. **Keep agent names consistent** - ADK uses names for routing
4. **Test thoroughly** - multi-agent systems have complex dependencies
5. **Update this CLAUDE.md** to reflect changes

## Session State Reference

| Key | Set By | Used By | Type |
|-----|--------|---------|------|
| `codebase_context` | `analyze_codebase` tool | `blog_planner`, `blog_writer` | str |
| `blog_outline` | `blog_planner` | `blog_writer`, user review | str (markdown) |
| `blog_post` | `blog_writer`, `blog_editor` | User review, export | str (markdown) |
| `social_media_posts` | `social_media_writer` | User review | str (markdown) |

## Testing Strategy

### Integration Test Pattern

```python
async def main():
    # 1. Create session service
    session_service = InMemorySessionService()
    await session_service.create_session(...)

    # 2. Create runner
    runner = Runner(agent=root_agent, ...)

    # 3. Simulate conversation
    queries = ["query1", "query2", ...]
    for query in queries:
        async for event in runner.run_async(...):
            if event.is_final_response():
                # Process response
```

### What to Test

- **Happy path**: Complete workflow from topic to exported file
- **User feedback**: Revision cycles work correctly
- **Edge cases**: Empty codebase, invalid directories, encoding errors
- **State persistence**: Session state maintains across turns

## Dependencies

### Core Dependencies
- `google-adk==1.18.0` - Agent Development Kit
- `pytest==8.4.2` - Testing framework
- `pytest-asyncio==1.2.0` - Async test support

### Optional Dependencies
- `deprecated` - Deprecation warnings
- `pandas` - Data manipulation (for future eval framework)
- `tabulate` - Table formatting
- `tqdm` - Progress bars
- `scikit-learn` - ML utilities (for future eval framework)

## Git Workflow

### Branch Naming
- Use descriptive branch names: `feature/new-agent`, `fix/validation-bug`

### Commit Messages
- Follow conventional commits: `feat:`, `fix:`, `docs:`, `refactor:`
- Reference issue numbers when applicable

### Ignored Files (`.gitignore`)
- Python cache files (`__pycache__`, `*.pyc`)
- Virtual environments (`.venv`, `venv/`)
- Environment files (`.env`)
- Distribution files (`dist/`, `build/`)
- IDE configs (`.idea/`, `.vscode/*`)
- Generated blog posts (`*.md` outputs)

## Troubleshooting

### Common Issues

**Agent not found**: Check `sub_agents/__init__.py` exports
**State key missing**: Verify `output_key` matches references
**Validation loop infinite**: Ensure validator uses `EventActions(escalate=True)`
**Encoding errors**: `analyze_codebase` handles UTF-8 and latin-1 fallback
**Model errors**: Check `GOOGLE_CLOUD_PROJECT` and authentication

### Environment Setup Issues

1. **Vertex AI (default)**:
   - Ensure `gcloud auth application-default login`
   - Project ID auto-detected from credentials

2. **AI Studio**:
   - Create `.env` file with `GOOGLE_API_KEY=...`
   - Set `GOOGLE_GENAI_USE_VERTEXAI=FALSE`

## Future Enhancement Ideas

Based on README.md value statement:
- **Trending topic scanner**: Agent to research trending topics across sites
- **MCP server integration**: Connect to Model Context Protocol servers
- **SEO optimization agent**: Analyze and optimize for search engines
- **Multi-format export**: Support for HTML, PDF, Medium format
- **Content calendar**: Scheduling and planning agent
- **Performance analytics**: Track engagement metrics

## Quick Reference

### File Locations
- Main agent: `blogger_agent/agent.py:30` (`interactive_blogger_agent`)
- Root export: `blogger_agent/__init__.py:1` (`root_agent`)
- Configuration: `blogger_agent/config.py:46` (`config`)
- Tools: `blogger_agent/tools.py`
- Tests: `tests/test_agent.py`

### Running Commands
```bash
# Interactive mode
adk web

# Integration test
python -m tests.test_agent

# Install deps
pip install -r requirements.txt
```

### Key Design Patterns
1. **LoopAgent + Validator**: Robust retry with quality checks
2. **Session State**: Agents communicate via state keys
3. **Hierarchical Delegation**: Main agent delegates to specialists
4. **User-in-the-Loop**: Iterative feedback at each stage
5. **Tool Abstraction**: Tools handle external operations

---

**Last Updated**: 2025-11-15
**ADK Version**: 1.18.0
**Python Version**: 3.11.3+

For questions or contributions, refer to the [README.md](README.md) and [LICENSE](LICENSE).
