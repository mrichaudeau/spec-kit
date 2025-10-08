# Data Model: Claude Team Command Integration

**Feature**: 001-objective-integrate-a
**Date**: 2025-10-08
**Purpose**: Define entities, relationships, and data structures for agent assignment and task delegation system

## Entity Definitions

### 1. Task

Represents an individual work item parsed from tasks.md.

**Attributes**:
- `task_id: str` - Unique identifier (e.g., "T001", "T002")
  - **Format**: `T` + zero-padded 3-digit number
  - **Validation**: Must match pattern `^T\d{3}$`

- `description: str` - Human-readable task description
  - **Example**: "Create User model in src/models/user.py"
  - **Constraints**: Non-empty, typically 10-100 characters

- `file_paths: List[str]` - File paths mentioned in or affected by the task
  - **Extraction**: Parse from description (e.g., "in `src/models/user.py`")
  - **Format**: Relative paths from repository root
  - **Example**: `["src/models/user.py", "tests/test_user.py"]`

- `is_parallel: bool` - Whether task can execute in parallel with others
  - **Indicator**: Presence of `[P]` marker in tasks.md
  - **Default**: `False` (sequential execution)

- `user_story_tag: Optional[str]` - Associated user story identifier
  - **Format**: `US` + number (e.g., "US1", "US2")
  - **Extraction**: Parse `[US1]` marker from tasks.md
  - **Purpose**: Group tasks by user story for independent testing

- `phase: str` - Phase this task belongs to
  - **Values**: "Setup", "Foundational", "User Story 1", "User Story 2", etc., "Polish"
  - **Extraction**: Based on section heading in tasks.md

- `dependencies: List[str]` - Task IDs this task depends on
  - **Computed**: Based on file path overlap + sequential ordering + phase dependencies
  - **Example**: `["T001", "T002"]` if T003 modifies same file as T001-T002

- `agent_assignment: Optional[str]` - Assigned agent filename
  - **Format**: `[agent_filename.md]` in tasks.md
  - **Example**: "frontend_specialist.md"
  - **State**: `None` before `/provide-claude-team` runs, populated after

- `completed: bool` - Execution status
  - **Indicator**: Checkbox state in tasks.md (`- [X]` vs `- [ ]`)
  - **Updated by**: `/implement` command after task execution

**Relationships**:
- **One Task → Zero-or-One Agent** (via `agent_assignment`)
- **One Task → Many Dependencies** (via `dependencies` list)
- **Many Tasks → One UserStory** (via `user_story_tag`)
- **Many Tasks → One Phase** (via `phase` field)

**Example**:
```python
Task(
    task_id="T010",
    description="Create User model in src/models/user.py",
    file_paths=["src/models/user.py"],
    is_parallel=True,
    user_story_tag="US1",
    phase="User Story 1",
    dependencies=[],
    agent_assignment="database_architect.md",
    completed=False
)
```

---

### 2. Agent

Represents a specialized Claude Code agent file stored in `.claude/agents/`.

**Attributes**:
- `filename: str` - Agent file name (unique identifier)
  - **Format**: `<specialty>.md` (e.g., "frontend_specialist.md")
  - **Constraint**: No feature-specific IDs (enables cross-feature reuse)
  - **Uniqueness**: Filename is the primary key

- `name: str` - Agent name from frontmatter
  - **Example**: "frontend-developer"
  - **Format**: Typically kebab-case or snake_case
  - **Source**: YAML frontmatter `name` field

- `description: str` - One-line agent purpose from frontmatter
  - **Example**: "Build React components, implement responsive layouts, and handle client-side state management"
  - **Length**: Typically 50-150 characters
  - **Source**: YAML frontmatter `description` field

- `model: str` - Claude model identifier
  - **Values**: "sonnet", "opus", "claude-sonnet-4", etc.
  - **Source**: YAML frontmatter `model` field

- `capabilities: List[str]` - Technical skills and expertise areas
  - **Extraction**: Parse `## Capabilities` section (all bullet points + H3 headings)
  - **Example**: `["REST API design", "FastAPI framework", "Authentication patterns"]`
  - **Update behavior**: Additive only (never remove existing)

- `specialty_domain: str` - High-level specialty category
  - **Derived from**: Filename (e.g., "frontend_specialist.md" → "frontend")
  - **Purpose**: Quick filtering for matching algorithm
  - **Examples**: "frontend", "api", "database", "test", "deploy", "general"

- `content: str` - Full Markdown file content (frontmatter + body)
  - **Purpose**: Preserve complete agent definition for updates
  - **Sections**: Purpose, Capabilities, Behavioral Traits, Response Approach, etc.

**Relationships**:
- **One Agent → Many Tasks** (across multiple features)
- **One Agent → One Specialty Domain** (derived field)

**Lifecycle**:
1. **Creation**: `/provide-claude-team` creates agent when no match found
2. **Update**: `/provide-claude-team` adds capabilities when reused with new requirements
3. **Reuse**: Multiple features reference same agent file

**Example**:
```python
Agent(
    filename="api_developer.md",
    name="api-developer",
    description="Design and implement RESTful APIs with authentication and error handling",
    model="sonnet",
    capabilities=[
        "REST API design patterns",
        "FastAPI framework",
        "Authentication and authorization",
        "Error handling and validation",
        "API documentation with OpenAPI"
    ],
    specialty_domain="api",
    content="---\nname: api-developer\n..."
)
```

---

### 3. AgentAssignment

Represents the matching and assignment logic outcome for a task-agent pair.

**Attributes**:
- `task: Task` - The task being assigned
  - **Reference**: Task entity

- `assigned_agent: Agent` - The agent assigned to this task
  - **Reference**: Agent entity
  - **Constraint**: Must exist or be created before assignment

- `match_confidence: float` - Overall matching score (0.0 - 1.0)
  - **Calculation**: Weighted combination of match factors
  - **Formula**: `0.4 * keyword_score + 0.4 * path_score + 0.2 * tech_score`
  - **Threshold**: > 0.3 triggers reuse, < 0.3 triggers new agent creation

- `match_factors: Dict[str, float]` - Individual factor scores
  - **Keys**: "keyword_score", "path_score", "tech_stack_score"
  - **Values**: 0.0 - 1.0 for each factor
  - **Purpose**: Debugging and confidence reporting

- `capabilities_added: List[str]` - New capabilities added to agent during assignment
  - **Purpose**: Track agent evolution over time
  - **Empty list**: If agent fully covers task requirements (no update needed)
  - **Non-empty**: If agent required capability additions

- `created_new_agent: bool` - Whether a new agent was created for this assignment
  - **True**: No existing agent had meaningful match (score < 0.3)
  - **False**: Reused existing agent (with or without updates)

**Relationships**:
- **One AgentAssignment → One Task**
- **One AgentAssignment → One Agent**
- **Ephemeral**: Not persisted after assignment completes (only annotation in tasks.md persists)

**Example**:
```python
AgentAssignment(
    task=Task(task_id="T015", description="Create user login API endpoint", ...),
    assigned_agent=Agent(filename="api_developer.md", ...),
    match_confidence=0.87,
    match_factors={
        "keyword_score": 0.67,  # "API", "endpoint" matched
        "path_score": 1.0,      # src/api/ path matched perfectly
        "tech_stack_score": 1.0 # FastAPI in plan.md
    },
    capabilities_added=["JWT token generation"],  # Added during this assignment
    created_new_agent=False  # Reused existing api_developer.md
)
```

---

### 4. MatchingContext

Represents the global context used during agent matching for all tasks in a feature.

**Attributes**:
- `task_keywords: Set[str]` - Extracted technical keywords from all tasks
  - **Examples**: {"API", "frontend", "database", "authentication", "test"}
  - **Purpose**: Vocabulary for keyword-based matching
  - **Extraction**: From task descriptions across entire tasks.md

- `file_path_patterns: Set[str]` - File path patterns observed in tasks
  - **Examples**: {"src/api/*", "src/models/*", "src/components/*", "tests/*"}
  - **Purpose**: Pattern matching for specialty inference

- `tech_stack: Dict[str, str]` - Technology choices from plan.md
  - **Format**: `{"category": "technology"}`
  - **Example**: `{"language": "Python 3.11", "framework": "FastAPI", "frontend": "React"}`
  - **Source**: Parsed from plan.md Technical Context section

- `existing_agents: List[Agent]` - All agents currently in .claude/agents/
  - **Purpose**: Pool of candidates for reuse
  - **Loaded once**: At start of `/provide-claude-team` execution

- `feature_dir: str` - Path to current feature's specs directory
  - **Example**: `specs/001-objective-integrate-a/`
  - **Purpose**: Locate plan.md and tasks.md

**Relationships**:
- **One MatchingContext → Many AgentAssignments** (provides context for all matches)
- **Singleton**: One per `/provide-claude-team` execution

**Lifecycle**:
1. **Initialization**: Load plan.md, scan .claude/agents/, extract keywords/patterns
2. **Usage**: Referenced by each AgentAssignment during matching
3. **Disposal**: Discarded after `/provide-claude-team` completes

**Example**:
```python
MatchingContext(
    task_keywords={"API", "endpoint", "authentication", "database", "model"},
    file_path_patterns={"src/api/*", "src/models/*", "tests/*"},
    tech_stack={
        "language": "Python 3.11",
        "framework": "FastAPI",
        "database": "PostgreSQL",
        "testing": "pytest"
    },
    existing_agents=[
        Agent(filename="api_developer.md", ...),
        Agent(filename="database_architect.md", ...),
        Agent(filename="general-purpose.md", ...)
    ],
    feature_dir="C:\\...\\specs\\001-objective-integrate-a"
)
```

---

## Entity Relationship Diagram

```
┌──────────────────┐
│ MatchingContext  │ (Singleton per execution)
│  - task_keywords │
│  - file_patterns │
│  - tech_stack    │
│  - existing_agents│
└────────┬─────────┘
         │ provides context for
         ▼
┌──────────────────┐         ┌──────────────┐
│ AgentAssignment  │────────>│    Agent     │
│  - task          │  assigns │  - filename  │
│  - assigned_agent│          │  - name      │
│  - confidence    │          │  - capabilities│
│  - match_factors │          │  - specialty │
└────────┬─────────┘          └──────┬───────┘
         │                           │
         │                           │ reused by many
         │                           ▼
         │                    ┌─────────────┐
         └───────────────────>│    Task     │
              for one task    │  - task_id  │
                              │  - description│
                              │  - file_paths│
                              │  - is_parallel│
                              │  - agent_assignment│
                              └─────────────┘
```

## Data Flow

### /provide-claude-team Execution Flow

1. **Load MatchingContext**:
   - Parse plan.md → extract tech_stack
   - Scan .claude/agents/ → load existing_agents
   - Parse tasks.md → extract task_keywords and file_path_patterns

2. **For each Task**:
   - Create AgentAssignment instance
   - Calculate match_confidence using MatchingContext
   - If match_confidence > 0.3: Reuse existing agent + update capabilities
   - If match_confidence ≤ 0.3: Create new Agent
   - Annotate tasks.md with agent_assignment

3. **Persist**:
   - Write/update Agent files in .claude/agents/
   - Update tasks.md with `[agent_filename.md]` annotations
   - Output summary report (task-to-agent mappings)

### /implement Execution Flow

1. **Parse tasks.md**:
   - Load all Task entities
   - Extract agent_assignment for each task
   - Build dependency graph (dependencies field)

2. **For each Task (respecting dependencies)**:
   - Validate Agent file exists (.claude/agents/[agent_assignment])
   - Load Agent content
   - Delegate task execution using Claude Code agent switching
   - Collect generated code
   - Write to file_paths
   - Mark completed = True in tasks.md

3. **Failure Handling**:
   - If task fails: Halt dependent tasks in chain, continue independent tasks
   - Report failed task, skipped tasks, continuing tasks

---

## Validation Rules

### Task Validation
- `task_id` must be unique within tasks.md
- `file_paths` must be valid relative paths
- `dependencies` must reference existing task_ids
- `agent_assignment` must reference existing file in .claude/agents/ (when set)

### Agent Validation
- `filename` must match `<specialty>.md` pattern (no feature IDs)
- Frontmatter must contain `name`, `description`, `model` fields
- File must have `## Capabilities` section
- `capabilities` list must not be empty

### AgentAssignment Validation
- `match_confidence` must be in range [0.0, 1.0]
- `assigned_agent` must be a valid Agent instance
- `capabilities_added` items must not duplicate existing agent capabilities

---

## State Transitions

### Task States
```
[Pending] → [Assigned] → [Executing] → [Completed]
                ↓
             [Skipped] (if dependency fails)
                ↓
             [Failed] (if execution errors)
```

### Agent States
```
[Not Exists] → [Created] → [Reused] → [Updated] → [Reused] → ...
                                                      ↑____________|
                                                   (evolving capabilities)
```

---

## Storage Format

### tasks.md (After agent assignment)
```markdown
- [ ] T001 [P] [US1] Create User model in src/models/user.py [database_architect.md]
- [ ] T002 [P] [US1] Create Post model in src/models/post.py [database_architect.md]
- [ ] T003 [US1] Create UserService in src/services/user.py [api_developer.md]
```

### .claude/agents/database_architect.md
```yaml
---
name: database-architect
description: Design database schemas, optimize queries, and manage data models
model: sonnet
---

## Purpose
...

## Capabilities
- Database schema design
- SQL query optimization
- ...
```

---

This data model provides the foundation for implementing the `/provide-claude-team` command and modified `/implement` logic. All entities are designed for simplicity, traceability, and alignment with functional requirements.
