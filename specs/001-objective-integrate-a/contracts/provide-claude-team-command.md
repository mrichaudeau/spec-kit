# Command Contract: /provide-claude-team

**Command Name**: `speckit.provide-claude-team`
**Location**: `.claude/commands/speckit.provide-claude-team.md`
**Purpose**: Analyze tasks and assign specialized Claude Code agents, creating or updating agent files as needed

## Input Contract

### Required Inputs
- **tasks.md file**: Must exist in current feature's specs directory
  - **Path**: Resolved via `.specify/scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks`
  - **Format**: Standard spec-kit tasks.md with task entries
  - **State**: Must be generated (i.e., `/speckit.tasks` must have run successfully)

### Optional Inputs
- **User arguments** (`$ARGUMENTS` from slash command)
  - **Purpose**: Override matching behavior or force specific assignments
  - **Format**: Free-text (e.g., "prefer frontend_specialist for all UI tasks")
  - **Default**: Empty (use automatic matching)

### Required Context Files
- **plan.md**: Must exist to extract tech stack information
  - **Location**: Same directory as tasks.md
  - **Used for**: Tech stack context factor in matching algorithm

- **.claude/agents/ directory**: May or may not exist
  - **If exists**: Load existing agents for reuse matching
  - **If not exists**: Create directory, all tasks get new agents

### Environment Assumptions
- Current working directory: Repository root
- Python 3.11+ available
- File system write permissions for .claude/agents/ and tasks.md

---

## Output Contract

### Primary Output: Updated tasks.md
**Format**: tasks.md with agent assignment annotations appended to each task line

**Example Transformation**:
```markdown
Before:
- [ ] T001 [P] [US1] Create User model in src/models/user.py

After:
- [ ] T001 [P] [US1] Create User model in src/models/user.py [database_architect.md]
```

**Annotation Format**: `[<agent_filename>.md]` at end of task line
**Preservation**: All existing task content, ordering, and markers preserved

### Secondary Output: Agent Files
**Location**: `.claude/agents/*.md`
**Actions**:
- **Create**: New agent files when no match found (confidence < 0.3)
- **Update**: Existing agent files when reused with new capabilities (confidence ≥ 0.3)
- **Format**: YAML frontmatter + Markdown body (see agent-file-contract.md)

### Console Output: Task-Agent Mapping Summary
**Format**: Human-readable summary table

**Example**:
```
Agent Assignment Summary
========================

Database Tasks (3 tasks → database_architect.md):
  - T001: Create User model in src/models/user.py
  - T002: Create Post model in src/models/post.py
  - T003: Create database migrations

API Development (2 tasks → api_developer.md):
  - T010: Create UserService in src/services/user.py
  - T011: Implement authentication endpoint

Frontend (1 task → frontend_specialist.md):
  - T020: Build user registration form

New Agents Created:
  - database_architect.md
  - api_developer.md
  - frontend_specialist.md

Agents Updated:
  (none)

Total: 6 tasks assigned to 3 agents (3 new, 0 updated)
```

---

## Side Effects

### File System Changes
1. **tasks.md**: Modified in-place with agent annotations
2. **.claude/agents/**: Directory created if not exists
3. **.claude/agents/*.md**: New agent files created or existing files updated
4. **No other files modified**: Spec, plan, research remain unchanged

### State Changes
- Tasks transition from "unassigned" to "assigned" state
- Agent library grows (new agents) or evolves (updated capabilities)
- Feature ready for `/speckit.implement` execution

---

## Error Conditions

### Input Validation Errors

**ERR-001: tasks.md not found**
- **Condition**: tasks.md does not exist in feature directory
- **Message**: `"tasks.md not found. Please run /speckit.tasks first to generate task breakdown."`
- **Exit Code**: 1
- **Recovery**: User must run `/speckit.tasks`

**ERR-002: tasks.md malformed**
- **Condition**: tasks.md exists but cannot be parsed (invalid format)
- **Message**: `"tasks.md is malformed. Expected format: - [ ] T001 [P] [US1] Description"`
- **Example Bad Line**: `* T001 Some task` (wrong bullet, no checkbox)
- **Exit Code**: 1
- **Recovery**: User must fix tasks.md or regenerate with `/speckit.tasks`

**ERR-003: plan.md not found**
- **Condition**: plan.md does not exist (tech stack context unavailable)
- **Message**: `"plan.md not found. Please run /speckit.plan first."`
- **Exit Code**: 1
- **Recovery**: User must run `/speckit.plan`

**ERR-004: No tasks found**
- **Condition**: tasks.md exists but contains no task entries
- **Message**: `"No tasks found in tasks.md. Nothing to assign."`
- **Exit Code**: 0 (success, no-op)
- **Recovery**: Not an error, just no work to do

### Execution Errors

**ERR-010: Agent template missing**
- **Condition**: templates/claude_code_agents/*.md examples not found (needed for structure reference)
- **Message**: `"Agent templates not found in templates/claude_code_agents/. Cannot generate agents."`
- **Exit Code**: 1
- **Recovery**: Check spec-kit installation integrity

**ERR-011: Agent file write failure**
- **Condition**: Cannot write to .claude/agents/ (permissions or disk space)
- **Message**: `"Failed to write agent file .claude/agents/api_developer.md: [OS error]"`
- **Exit Code**: 1
- **Recovery**: Check file system permissions and disk space

**ERR-012: tasks.md write failure**
- **Condition**: Cannot write updated tasks.md (file locked or permissions)
- **Message**: `"Failed to update tasks.md with agent assignments: [OS error]"`
- **Exit Code**: 1
- **Recovery**: Close any programs with tasks.md open, check permissions
- **Rollback**: Agent files created successfully remain, but tasks.md unchanged

### Matching Errors

**ERR-020: All tasks default to general-purpose**
- **Condition**: Matching algorithm assigns all tasks to general-purpose agent (possible misconfiguration)
- **Message**: `"Warning: All tasks assigned to general-purpose agent. Review plan.md tech stack or task descriptions."`
- **Exit Code**: 0 (warning, not fatal)
- **Recovery**: User reviews plan.md and task descriptions for clarity

---

## Performance Contract

### Time Constraints
- **Target**: Complete within 10 minutes for typical feature sizes (per SC-001)
- **Scaling**: Graceful degradation with increased task count
  - 1-20 tasks: < 1 minute
  - 21-50 tasks: 1-5 minutes
  - 51-100 tasks: 5-10 minutes
  - 100+ tasks: > 10 minutes (acceptable)

### Resource Usage
- **Memory**: O(n) where n = number of tasks (all tasks loaded into memory)
- **Disk I/O**: O(m) where m = number of agents created/updated (file writes)
- **CPU**: Minimal (regex parsing, string matching)

---

## Idempotency

**Re-execution Behavior**:
- Running `/provide-claude-team` multiple times on same tasks.md:
  - **First run**: Assigns agents, annotates tasks.md
  - **Subsequent runs**: Re-evaluates assignments, may update agents with new capabilities if task descriptions changed
  - **Safe**: Existing agent files preserved (additive updates only)

**Manual Edits**:
- If user manually edits agent assignments in tasks.md: Preserved unless explicit re-assignment requested
- If user deletes agent files: Re-running `/provide-claude-team` recreates them

---

## Integration Points

### Upstream Commands (Prerequisites)
1. `/speckit.specify` → Generates spec.md
2. `/speckit.clarify` → Refines spec.md
3. `/speckit.plan` → Generates plan.md
4. `/speckit.tasks` → Generates tasks.md (**required direct predecessor**)

### Downstream Commands (Consumers)
1. `/speckit.implement` → Reads agent assignments from tasks.md, delegates to agents

### Workflow Position
```
/specify → /clarify → /plan → /tasks → /provide-claude-team → /implement
                                              ↑ YOU ARE HERE
```

---

## Testing Contract

### Unit Test Requirements
- **Test 1**: Parse tasks.md with various formats (parallel markers, user story tags)
- **Test 2**: Match tasks to existing agents with known characteristics
- **Test 3**: Create new agent files with correct frontmatter and structure
- **Test 4**: Update existing agent capabilities without overwriting
- **Test 5**: Handle empty tasks.md gracefully
- **Test 6**: Handle missing plan.md with appropriate error

### Integration Test Requirements
- **Test 7**: End-to-end: tasks.md → agent assignment → updated tasks.md + agent files
- **Test 8**: Verify agent reuse across multiple features (run on feature 1, then feature 2)
- **Test 9**: Verify tasks.md annotations are machine-readable by `/implement`

### Acceptance Criteria (from spec.md)
- **AC-001**: Running command after `/tasks` produces agent assignments for all tasks (FR-002, FR-005, FR-006)
- **AC-002**: tasks.md updated with `[agent_filename.md]` annotations (FR-011)
- **AC-003**: Agent files created in .claude/agents/ with valid structure (FR-009, FR-010)
- **AC-004**: Existing agents reused when capabilities match (FR-005)
- **AC-005**: Console summary clearly shows task-to-agent mappings (FR-013)

---

## Example Usage

### Scenario 1: First Feature (No Existing Agents)
```bash
# After running /speckit.tasks
/provide-claude-team

# Output:
# Analyzing 15 tasks from tasks.md...
# No existing agents found in .claude/agents/
# Creating 4 new agents: database_architect, api_developer, frontend_specialist, test_specialist
# Updated tasks.md with agent assignments
#
# Agent Assignment Summary
# ========================
# Database Tasks (3 tasks → database_architect.md)
# API Development (5 tasks → api_developer.md)
# Frontend (4 tasks → frontend_specialist.md)
# Testing (3 tasks → test_specialist.md)
#
# New Agents Created: 4
# Total: 15 tasks assigned to 4 agents
```

### Scenario 2: Second Feature (Reusing Agents)
```bash
# On a new feature branch with different tasks
/provide-claude-team

# Output:
# Analyzing 10 tasks from tasks.md...
# Found 4 existing agents in .claude/agents/
# Reusing api_developer (added GraphQL capabilities)
# Reusing frontend_specialist (added accessibility features)
# Creating 1 new agent: deployment_engineer
#
# Agent Assignment Summary
# ========================
# API Development (4 tasks → api_developer.md) [UPDATED]
# Frontend (3 tasks → frontend_specialist.md) [UPDATED]
# Deployment (3 tasks → deployment_engineer.md) [NEW]
#
# New Agents Created: 1
# Agents Updated: 2
# Total: 10 tasks assigned to 3 agents
```

---

This contract defines all interfaces, behaviors, and guarantees for the `/provide-claude-team` command.
