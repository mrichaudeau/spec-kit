# Research: Claude Team Command Integration

**Feature**: 001-objective-integrate-a
**Date**: 2025-10-08
**Purpose**: Technical research to resolve implementation approaches for `/provide-claude-team` command and agent delegation system

## Research Area 1: Markdown Parsing for Task Extraction

### Decision
Use Python's built-in `re` (regular expressions) module combined with simple line-by-line parsing, without introducing additional heavyweight Markdown libraries.

### Rationale
- **Simplicity**: tasks.md has a predictable, line-based format (task per line with standard markers)
- **Zero new dependencies**: No need for markdown-it-py, mistune, or similar libraries
- **Performance**: Line-by-line regex parsing is fast and sufficient for the task format
- **Maintainability**: Simple regex patterns are easy to understand and modify
- **Existing precedent**: Spec-kit already uses simple file I/O for similar tasks

### Alternatives Considered
1. **markdown-it-py**: Full-featured Markdown parser
   - **Rejected**: Overkill for structured line parsing; adds unnecessary dependency weight
2. **mistune**: Fast Markdown parser
   - **Rejected**: Still more complex than needed; tasks.md isn't rich Markdown
3. **Custom AST parser**: Build Markdown AST
   - **Rejected**: Over-engineered for simple `- [ ] T001 [P] [US1] Description [agent.md]` format

### Parsing Strategy
```
Pattern: ^- \[([ Xx])\] (T\d+) (\[P\])? (\[US\d+\])? (.+?) (?:\[(.+?\.md)\])?$

Groups:
1. Checkbox state (completed/incomplete)
2. Task ID (T001, T002, etc.)
3. Parallel marker ([P] or None)
4. User story tag ([US1], [US2], or None)
5. Task description + file paths
6. Agent assignment ([agent_filename.md] or None)
```

### Traceability
- FR-002: Read and parse tasks.md
- FR-011: Extract agent assignments
- FR-015: Parse task structure

---

## Research Area 2: Agent File Format & Structure

### Decision
Agent files follow YAML frontmatter + Markdown body structure, with strict section requirements:

**Frontmatter (YAML)**:
```yaml
---
name: agent_specialty_name
description: One-line description of agent purpose and when to use proactively
model: sonnet | claude-sonnet-4 | opus
---
```

**Required Sections** (Markdown H2):
1. `## Purpose` - Detailed agent role and expertise summary
2. `## Capabilities` - Bulleted list of technical skills (may contain H3 subsections)
3. `## Behavioral Traits` - How the agent approaches tasks
4. `## Response Approach` - Step-by-step methodology

**Optional Sections**:
- `## Knowledge Base` - Domain-specific knowledge areas
- `## Example Interactions` - Sample prompts the agent handles well

### Rationale
- **Consistency**: Matches existing templates/claude_code_agents/*.md structure exactly
- **Machine-readable**: YAML frontmatter parses easily with Python `yaml` library
- **Human-readable**: Markdown body is editable and understandable
- **Claude Code compatible**: Frontmatter structure aligns with Claude Code agent discovery

### Examples Analyzed
- `frontend-developer.md`: Rich capabilities with H3 subsections, detailed behavioral traits
- `general-purpose.md`: Minimal structure, fewer capabilities, simpler format
- Both follow same frontmatter + required sections pattern

### Update Strategy
When adding capabilities to existing agent:
1. Parse existing file, extract frontmatter + sections
2. Locate `## Capabilities` section
3. Append new bullet points or H3 subsections (avoid duplicates)
4. Preserve all other sections unchanged
5. Write back preserving formatting

### Traceability
- FR-009: Frontmatter fields (name, description, model)
- FR-010: Structural template from templates/claude_code_agents/
- FR-026: Merge capabilities without overwriting

---

## Research Area 3: Claude Code Agent Switching Mechanism

### Decision
Claude Code uses file-based agent discovery in `.claude/agents/` directory. To delegate to a specialist agent, the system loads the agent file into Claude Code's context by referencing the file path. Agent switching happens automatically when Claude Code is directed to "act as" or "use" a specific agent file.

### Rationale
- **Native mechanism**: Clarification Q1 confirmed using Claude Code's existing agent file switching
- **No subprocess overhead**: No need to spawn separate Claude instances
- **Context isolation**: Each agent file provides specialized context/instructions
- **Simple implementation**: Just reference agent file path in task delegation prompt

### Implementation Approach
When `/implement` encounters a task with agent assignment `[frontend_specialist.md]`:

```
1. Validate .claude/agents/frontend_specialist.md exists
2. Load agent file content into memory
3. Execute task with agent context:
   "Using the agent defined in .claude/agents/frontend_specialist.md,
    perform task T001: [task description]"
4. Collect generated code output
5. Write to specified file paths
```

### Alternatives Considered
1. **Subprocess API calls**: Spawn separate Claude instances per agent
   - **Rejected**: Clarification Q1 ruled this out; not native mechanism
2. **Prompt engineering only**: Simulate agent personas with prompts
   - **Rejected**: Doesn't leverage Claude Code agent files; less effective specialization
3. **Task tool with agent parameter**: Use existing Task tool
   - **Rejected**: Clarification Q1 confirmed agent file switching is the mechanism

### Traceability
- FR-016: Load assigned agent file context before execution
- Clarification Q1: Use Claude Code agent file switching mechanism

---

## Research Area 4: Hybrid Matching Algorithm Design

### Decision
Three-factor weighted scoring system with configurable thresholds:

**Factor 1: Keyword Extraction (40% weight)**
- Extract technical keywords from task description: "API", "frontend", "database", "test", "deploy", etc.
- Match against agent specialty domains (inferred from agent name/description)
- Score: `keyword_matches / total_keywords`

**Factor 2: File Path Pattern Analysis (40% weight)**
- Analyze file paths mentioned in task description
- Pattern matching:
  - `src/api/*` or `*/api/*` → api_developer
  - `src/components/*` or `*/frontend/*` → frontend_specialist
  - `src/models/*` or `*/database/*` → database_architect
  - `tests/*` or `*test*.py` → test_specialist
- Score: `path_pattern_matches / total_paths`

**Factor 3: Tech Stack Context (20% weight)**
- Read plan.md for technology choices
- Match task requirements against declared tech stack
  - FastAPI in plan → bonus for api_developer
  - React in plan → bonus for frontend_specialist
  - PostgreSQL in plan → bonus for database_architect
- Score: `tech_stack_alignment (boolean 0 or 1)`

**Combined Score**: `0.4 * keyword_score + 0.4 * path_score + 0.2 * tech_score`

**Reuse Threshold**: Match score > 0.3 triggers reuse + update
- Score 0.3-0.6: Partial match, reuse and add capabilities
- Score > 0.6: Strong match, reuse as-is (maybe add minor capabilities)
- Score < 0.3: No meaningful overlap, create new agent

### Rationale
- **Balanced approach**: Clarification Q2 confirmed hybrid multi-factor
- **File paths most reliable**: Path patterns strongly indicate specialty (40% weight)
- **Keywords complement**: Task descriptions provide additional context (40% weight)
- **Tech stack context**: Plan.md provides global constraints (20% weight)
- **Low threshold**: Clarification Q5 confirmed always prefer reuse (even <50% = 0.3 score)

### Example Calculation
Task: "Create API endpoint for user registration in `src/api/users.py`"

- **Keywords**: "API", "endpoint" → api_developer match → score: 2/3 = 0.67
- **File path**: `src/api/users.py` matches `src/api/*` pattern → api_developer → score: 1.0
- **Tech stack**: plan.md mentions FastAPI → api_developer aligned → score: 1.0
- **Combined**: 0.4 * 0.67 + 0.4 * 1.0 + 0.2 * 1.0 = 0.87 (strong match)

Decision: Reuse api_developer.md, minimal capability additions needed

### Traceability
- FR-024: Hybrid approach (keywords + file paths + tech stack)
- Clarification Q2: Multi-factor decision
- FR-005: Always prefer reuse (low threshold 0.3)

---

## Research Area 5: Agent File Update Strategy

### Decision
Surgical capability merging with content-aware deduplication:

**Update Algorithm**:
1. **Parse existing agent** (frontmatter + sections)
2. **Identify new capabilities** required by task (from task description + file paths)
3. **Locate `## Capabilities` section** (mandatory section)
4. **Check for duplicates**: Use semantic similarity (keyword overlap) to avoid redundant bullets
5. **Append new capabilities**:
   - If capabilities organized with H3 subsections, find appropriate subsection or create new one
   - If flat bullet list, append to end
6. **Preserve formatting**: Maintain existing indentation, bullet style, section order
7. **Write updated file** (overwrite .claude/agents/[agent].md)

**Deduplication Logic**:
```python
new_cap = "REST API design patterns"
existing_caps = ["REST API implementation", "API authentication", ...]

# Check keyword overlap
if keyword_overlap(new_cap, existing_caps) > 0.7:
    skip  # Already covered
else:
    append(new_cap)
```

**Example Update**:
```markdown
## Capabilities

### API Development (existing)
- REST API implementation
- API authentication

### Database Integration (existing)
- SQL query optimization

(NEW capability needed: GraphQL support)

## Capabilities (updated)

### API Development
- REST API implementation
- API authentication
- GraphQL schema design and resolvers  <-- ADDED

### Database Integration
- SQL query optimization
```

### Rationale
- **Preserves history**: Existing capabilities never removed (FR-026)
- **Avoids duplication**: Semantic checks prevent redundant bullets
- **Maintains structure**: H3 subsections preserved when present
- **Graceful evolution**: Agents grow more capable over time, not replaced

### Alternatives Considered
1. **Simple append to end**: No deduplication
   - **Rejected**: Leads to redundancy over many features
2. **Full capability replacement**: Overwrite entire section
   - **Rejected**: Violates FR-026 (preserve existing capabilities)
3. **Manual review prompt**: Ask user to approve updates
   - **Rejected**: Defeats automation goal; should be transparent

### Traceability
- FR-026: Merge without removing/overwriting
- FR-005: Update existing agents when reused

---

## Research Area 6: Failure Handling & Dependency Tracking

### Decision
**Dependency Graph Construction**:
1. **Parse tasks.md** to identify:
   - Tasks marked `[P]` → parallel, no dependencies
   - Tasks referencing same file paths → sequential dependency
   - Tasks within same phase without `[P]` → sequential within phase
   - Tasks in later phases depend on earlier phases completing

2. **Build dependency chains**:
   ```
   Chain 1: T001 → T002 → T003 (same file, sequential)
   Chain 2: T010 [P] (independent, parallel)
   Chain 3: T011 [P] (independent, parallel)
   Chain 4: T020 → T021 (different file, sequential)
   ```

3. **Execution Model**:
   - Parallel tasks ([P]) execute concurrently (within phase)
   - Sequential tasks execute in order
   - Phase boundaries are hard synchronization points

**Failure Handling Logic** (from Clarification Q3):
```python
if task_fails(task_id):
    # Halt dependent tasks in same chain
    for dep_task in get_dependency_chain(task_id):
        mark_skipped(dep_task, reason=f"Depends on failed {task_id}")

    # Continue independent parallel tasks
    for parallel_task in get_parallel_independent_tasks():
        continue_execution(parallel_task)

    # Report failure with context
    report(task_id, agent, error_message, halted_tasks, continuing_tasks)
```

**Dependency Detection Rules**:
1. **File path overlap**: Tasks modifying same file → sequential
2. **Phase ordering**: Later phase tasks depend on earlier phase completion
3. **Explicit markers**: Absence of `[P]` within phase → sequential
4. **User story boundaries**: Tasks in same `[US1]` may have internal dependencies

### Rationale
- **Clarification Q3**: Continue independent tasks, halt only dependent chain
- **Maximizes progress**: Don't stop all work due to one failure
- **Clear reporting**: User knows which tasks succeeded, failed, skipped
- **Preserves consistency**: File-level dependencies prevent conflicts

### Example Scenario
```
Phase 2: Core Implementation
- [ ] T010 [P] [US1] Create User model in src/models/user.py
- [ ] T011 [P] [US1] Create Post model in src/models/post.py
- [ ] T012 [US1] Create UserService in src/services/user.py (depends on T010)
- [ ] T013 [US2] Create APIRouter in src/api/router.py

If T010 fails:
- T011 continues (parallel, different file, independent)
- T012 skipped (depends on T010 via User model)
- T013 continues (different user story, different file, independent)
```

### Traceability
- FR-018: Respect parallel [P] and sequential execution rules
- FR-020: Halt dependent tasks, continue independent tasks
- Clarification Q3: Failure handling behavior

---

## Summary of Research Findings

| Area | Decision | Key Traceability |
|------|----------|------------------|
| **Markdown Parsing** | Built-in regex (no deps) | FR-002, FR-011, FR-015 |
| **Agent File Format** | YAML frontmatter + required sections | FR-009, FR-010, FR-026 |
| **Agent Switching** | Claude Code native file-based | FR-016, Clarification Q1 |
| **Matching Algorithm** | 3-factor weighted (0.4/0.4/0.2) | FR-024, Clarification Q2 |
| **Agent Updates** | Surgical capability merging | FR-026, FR-005 |
| **Failure Handling** | Halt chain, continue independent | FR-020, Clarification Q3 |

**All research areas resolved. Ready for Phase 1 (Design & Contracts).**
