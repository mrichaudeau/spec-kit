# Quickstart: Claude Team Command Integration

**Feature**: 001-objective-integrate-a
**Purpose**: Demonstrate the complete workflow from task generation to agent-delegated implementation
**Audience**: Developers using spec-kit with Claude Code

---

## Prerequisites

Before starting, ensure you have:
- ✅ spec-kit installed and initialized in your project
- ✅ Completed `/speckit.specify` (feature specification created)
- ✅ Completed `/speckit.clarify` (ambiguities resolved)
- ✅ Completed `/speckit.plan` (technical plan ready)
- ✅ Completed `/speckit.tasks` (task breakdown generated)

**Current State**: You should have a `tasks.md` file in your feature directory (`specs/###-feature-name/tasks.md`)

---

## Step 1: Generate Task Breakdown

If you haven't already run `/speckit.tasks`, do so now:

```bash
/speckit.tasks
```

**Expected Output**: `tasks.md` file with structured task list

**Example tasks.md** (before agent assignment):
```markdown
## Phase 2: Core Implementation

- [ ] T010 [P] [US1] Create User model in src/models/user.py
- [ ] T011 [P] [US1] Create Post model in src/models/post.py
- [ ] T012 [US1] Create UserService in src/services/user.py
- [ ] T015 [US2] Create authentication endpoint in src/api/auth.py
- [ ] T020 [P] [US3] Build user registration form in src/components/RegisterForm.tsx
```

---

## Step 2: Assign Agents to Tasks

Run the new `/provide-claude-team` command:

```bash
/provide-claude-team
```

**What Happens**:
1. Command reads your `tasks.md`
2. Analyzes each task's description and file paths
3. Checks if any existing agents in `.claude/agents/` match task requirements
4. Creates new specialized agents or reuses existing ones
5. Annotates `tasks.md` with agent assignments
6. Displays a summary of assignments

**Expected Console Output**:
```
Analyzing tasks from tasks.md...
Reading plan.md for tech stack context...
Tech stack detected: Python 3.11, FastAPI, React, PostgreSQL

Scanning .claude/agents/ for existing agents...
No existing agents found.

Performing agent matching for 15 tasks...

Creating new agents:
  ✓ database_architect.md (3 tasks)
  ✓ api_developer.md (5 tasks)
  ✓ frontend_specialist.md (4 tasks)
  ✓ test_specialist.md (3 tasks)

Updating tasks.md with agent assignments...

Agent Assignment Summary
========================

Database Tasks (3 tasks → database_architect.md):
  - T010: Create User model in src/models/user.py
  - T011: Create Post model in src/models/post.py
  - T014: Create database migrations

API Development (5 tasks → api_developer.md):
  - T012: Create UserService in src/services/user.py
  - T015: Create authentication endpoint in src/api/auth.py
  - T016: Implement password hashing
  - T017: Add JWT token generation
  - T018: Create user registration endpoint

Frontend (4 tasks → frontend_specialist.md):
  - T020: Build user registration form in src/components/RegisterForm.tsx
  - T021: Create login form component
  - T022: Implement form validation
  - T023: Add error handling UI

Testing (3 tasks → test_specialist.md):
  - T030: Write user model tests
  - T031: Write API endpoint tests
  - T032: Write component integration tests

New Agents Created: 4
Agents Updated: 0
Total: 15 tasks assigned to 4 agents

✓ Complete! Ready for /speckit.implement
```

---

## Step 3: Inspect Agent Assignments

Open `tasks.md` to see the updated annotations:

**Example tasks.md** (after agent assignment):
```markdown
## Phase 2: Core Implementation

- [ ] T010 [P] [US1] Create User model in src/models/user.py [database_architect.md]
- [ ] T011 [P] [US1] Create Post model in src/models/post.py [database_architect.md]
- [ ] T012 [US1] Create UserService in src/services/user.py [api_developer.md]
- [ ] T015 [US2] Create authentication endpoint in src/api/auth.py [api_developer.md]
- [ ] T020 [P] [US3] Build user registration form in src/components/RegisterForm.tsx [frontend_specialist.md]
```

**Notice**: Each task now has `[agent_name.md]` at the end.

---

## Step 4: Verify Agent Files Created

Check that agent files were created in `.claude/agents/`:

```bash
ls .claude/agents/
```

**Expected Output**:
```
database_architect.md
api_developer.md
frontend_specialist.md
test_specialist.md
```

**Inspect an agent file** (e.g., `database_architect.md`):

```bash
cat .claude/agents/database_architect.md
```

**Expected Structure**:
```markdown
---
name: database-architect
description: Design database schemas, optimize queries, and manage data models with PostgreSQL
model: sonnet
---

## Purpose
Expert database architect specializing in PostgreSQL schema design, migrations, and query optimization...

## Capabilities

### Database Design
- Relational schema design and normalization
- PostgreSQL-specific features (JSONB, arrays, CTEs)
- Index optimization for query performance
...

## Behavioral Traits
- Prioritizes data integrity and consistency
- Uses foreign keys and constraints liberally
...

## Response Approach
1. **Analyze data requirements** from spec
2. **Design normalized schema** with appropriate relationships
3. **Create migrations** with rollback support
...
```

---

## Step 5: Run Implementation with Agent Delegation

Execute tasks using the assigned specialist agents:

```bash
/speckit.implement
```

**What Happens**:
1. Command reads `tasks.md` with agent assignments
2. Validates all agent files exist
3. For each task:
   - Loads the assigned agent's context from `.claude/agents/[agent].md`
   - Delegates task execution to that specialist agent
   - Generates code using agent's expertise
   - Writes code to specified file paths
4. Reports progress and completion

**Expected Console Output**:
```
Checking prerequisites...
✓ tasks.md found (15 tasks)
✓ plan.md found
✓ Agent assignments detected (4 unique agents)

Validating agent files...
✓ database_architect.md
✓ api_developer.md
✓ frontend_specialist.md
✓ test_specialist.md

Loading implementation context...
✓ tasks.md (15 tasks)
✓ plan.md
✓ data-model.md
✓ research.md

Executing Phase 1: Setup
  ✓ T001: Initialize project structure - COMPLETED

Executing Phase 2: Core Implementation
  ⏩ T010: Create User model [database_architect.md] - IN PROGRESS
     [database_architect.md] Analyzing data model requirements...
     [database_architect.md] Designing schema with PostgreSQL best practices...
     [database_architect.md] Creating User model with fields: id, email, password_hash, created_at
  ✓ T010: Create User model [database_architect.md] - COMPLETED
     Files created: src/models/user.py

  ⏩ T011: Create Post model [database_architect.md] - IN PROGRESS
  ✓ T011: Create Post model [database_architect.md] - COMPLETED
     Files created: src/models/post.py

  ⏩ T012: Create UserService [api_developer.md] - IN PROGRESS
     [api_developer.md] Implementing service layer with FastAPI patterns...
     [api_developer.md] Adding error handling and validation...
  ✓ T012: Create UserService [api_developer.md] - COMPLETED
     Files created: src/services/user.py

  ⏩ T015: Create authentication endpoint [api_developer.md] - IN PROGRESS
  ✓ T015: Create authentication endpoint [api_developer.md] - COMPLETED
     Files created: src/api/auth.py

  ⏩ T020: Build user registration form [frontend_specialist.md] - IN PROGRESS
     [frontend_specialist.md] Creating React component with TypeScript...
     [frontend_specialist.md] Adding form validation and accessibility...
  ✓ T020: Build user registration form [frontend_specialist.md] - COMPLETED
     Files created: src/components/RegisterForm.tsx

... (continues for remaining tasks)

Execution Summary:
  ✓ Completed: 15/15 tasks
  ✗ Failed: 0 tasks
  ⏭ Skipped: 0 tasks

Agents Used:
  - database_architect.md (3 tasks)
  - api_developer.md (5 tasks)
  - frontend_specialist.md (4 tasks)
  - test_specialist.md (3 tasks)

All tasks completed successfully! 🎉
```

---

## Step 6: Verify Generated Code

Check that code was generated according to task specifications:

```bash
# Check database models
cat src/models/user.py

# Check API endpoints
cat src/api/auth.py

# Check frontend components
cat src/components/RegisterForm.tsx
```

**Expected Characteristics**:
- **database_architect.md generated code**: Proper PostgreSQL models, foreign keys, indexes
- **api_developer.md generated code**: FastAPI endpoints with Pydantic validation, error handling
- **frontend_specialist.md generated code**: React components with TypeScript, accessibility attributes

---

## Step 7: Subsequent Features (Agent Reuse)

When working on a second feature, agents are automatically reused and evolved:

```bash
# On feature branch 002-add-comments
/speckit.specify
# ... (create spec for comment feature)

/speckit.clarify
/speckit.plan
/speckit.tasks

/provide-claude-team
```

**Expected Output** (second feature):
```
Analyzing tasks from tasks.md...
Scanning .claude/agents/ for existing agents...
Found 4 existing agents.

Performing agent matching for 8 tasks...

Reusing existing agents:
  ✓ database_architect.md (2 tasks) - Added capability: "Comment model design"
  ✓ api_developer.md (3 tasks) - Added capability: "Comment CRUD endpoints"
  ✓ frontend_specialist.md (2 tasks) - Added capability: "Comment UI components"

Creating new agents:
  ✓ moderation_specialist.md (1 task)

Agent Assignment Summary
========================

Database Tasks (2 tasks → database_architect.md) [UPDATED]:
  - T005: Create Comment model
  - T006: Add comment-user foreign key

API Development (3 tasks → api_developer.md) [UPDATED]:
  - T010: Create comment endpoints
  - T011: Add comment validation
  - T012: Implement comment pagination

Frontend (2 tasks → frontend_specialist.md) [UPDATED]:
  - T020: Build comment list component
  - T021: Create comment form

Moderation (1 task → moderation_specialist.md) [NEW]:
  - T030: Implement content moderation filter

New Agents Created: 1
Agents Updated: 3
Total: 8 tasks assigned to 4 agents
```

**Notice**:
- Existing agents were **reused** (not duplicated)
- Capabilities were **added** (agents evolved)
- New specialty required = new agent created (`moderation_specialist.md`)

---

## Troubleshooting

### Issue 1: "Agent file not found" error during /implement

**Symptom**:
```
✗ ERROR: Agent file not found: .claude/agents/api_developer.md
```

**Cause**: Agent file was deleted or moved after assignment

**Solution**:
```bash
# Re-run agent assignment to regenerate
/provide-claude-team
```

---

### Issue 2: All tasks assigned to general-purpose agent

**Symptom**:
```
Agent Assignment Summary
========================
General Tasks (15 tasks → general-purpose.md)
```

**Cause**: Task descriptions or plan.md lack sufficient technical detail for specialty matching

**Solution**:
1. Review task descriptions - add technical keywords (e.g., "API", "database", "frontend")
2. Ensure plan.md has detailed tech stack in Technical Context section
3. Re-run `/provide-claude-team` after clarifications

---

### Issue 3: Agent file malformed error

**Symptom**:
```
✗ ERROR: Agent file .claude/agents/api_developer.md is malformed.
Missing required frontmatter field: 'name'
```

**Cause**: Manual edit corrupted agent file structure

**Solution**:
```bash
# Delete corrupted agent file
rm .claude/agents/api_developer.md

# Re-run agent assignment to regenerate
/provide-claude-team
```

---

## Advanced Usage

### Manual Agent Assignment Override

You can manually edit `tasks.md` to assign a different agent:

```markdown
# Change assignment
- [ ] T010 Create model [database_architect.md]

# To:
- [ ] T010 Create model [api_developer.md]
```

Then run `/implement` (no need to re-run `/provide-claude-team`)

### Creating Custom Agents

You can manually create agent files in `.claude/agents/` following the contract:

1. Copy an existing agent as a template
2. Modify frontmatter (name, description, model)
3. Update Purpose, Capabilities, Behavioral Traits, Response Approach sections
4. Save as `<specialty>.md`
5. Manually assign in `tasks.md` or let `/provide-claude-team` discover it

---

## Summary Workflow

```
┌────────────────┐
│ /speckit.tasks │
└───────┬────────┘
        │
        ▼
┌─────────────────────┐
│ /provide-claude-team│ ← NEW COMMAND
└───────┬─────────────┘
        │
        ├─→ Creates/updates .claude/agents/*.md
        └─→ Annotates tasks.md with [agent.md]
        │
        ▼
┌──────────────────┐
│ /speckit.implement│ ← MODIFIED COMMAND
└───────┬──────────┘
        │
        ├─→ Loads agent context
        ├─→ Delegates to specialist agents
        └─→ Generates code with agent expertise
        │
        ▼
    ✅ Feature Complete
```

---

## Next Steps

After completing this quickstart, you can:

1. **Inspect agent files** to understand their capabilities
2. **Customize agents** by adding domain-specific knowledge
3. **Build agent library** over multiple features (agents evolve)
4. **Share agents** across teams (`.claude/agents/` can be committed to Git)

**Congratulations!** You've successfully used the Claude Team command integration to automate agent assignment and delegation. 🎉
