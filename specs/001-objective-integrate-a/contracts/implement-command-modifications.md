# Command Modifications: /speckit.implement

**Command Name**: `speckit.implement`
**Location**: `.claude/commands/speckit.implement.md`
**Modification Type**: Enhancement (backward compatible)
**Purpose**: Add agent delegation capability while preserving existing direct implementation mode

## Overview of Changes

The `/implement` command is modified to:
1. Detect agent assignments in tasks.md (`[agent_filename.md]` annotations)
2. Load assigned agent context from `.claude/agents/` before task execution
3. Delegate task execution using Claude Code native agent switching
4. Handle failures per clarified strategy (halt dependent chain, continue independent tasks)
5. Fall back to direct implementation if no agent assignments present (backward compatibility)

---

## Modified Input Contract

### Existing Inputs (Preserved)
- tasks.md file (required)
- plan.md, data-model.md, contracts/, research.md (optional context)
- Checklist validation (preserved)

### New Inputs (Added)
- **Agent assignment annotations** in tasks.md
  - **Format**: `[agent_filename.md]` at end of task line
  - **Optional**: If absent, use direct implementation (original behavior)
  - **Example**: `- [ ] T001 Create model [database_architect.md]`

- **Agent files** in `.claude/agents/`
  - **Required if**: Agent assignments present in tasks.md
  - **Validation**: FR-023 (validate agent files exist before delegation)

---

## Modified Execution Logic

### Phase 1: Initialization (New Steps Added)

**Step 1a**: Parse tasks.md to extract agent assignments
```python
for task in tasks:
    # Extract agent assignment (new)
    match = re.search(r'\[(.+?\.md)\]$', task.line)
    if match:
        task.agent_assignment = match.group(1)
        task.delegation_mode = True
    else:
        task.agent_assignment = None
        task.delegation_mode = False
```

**Step 1b**: Validate agent files exist (FR-023)
```python
assigned_agents = {task.agent_assignment for task in tasks if task.agent_assignment}
missing_agents = []

for agent_filename in assigned_agents:
    agent_path = f".claude/agents/{agent_filename}"
    if not os.path.exists(agent_path):
        missing_agents.append(agent_filename)

if missing_agents:
    raise Error(f"Missing agent files: {missing_agents}. Run /provide-claude-team to regenerate.")
```

### Phase 2: Task Execution (Modified Logic)

**For each task**:

**Option A: Delegated Execution (New)**
```python
if task.delegation_mode:
    # Load agent file context
    agent_path = f".claude/agents/{task.agent_assignment}"
    agent_context = load_agent_file(agent_path)

    # Delegate to Claude Code agent switching mechanism
    result = execute_with_agent(
        agent_context=agent_context,
        task_description=task.description,
        file_paths=task.file_paths
    )

    # Handle result
    if result.success:
        write_generated_code(task.file_paths, result.code)
        mark_task_completed(task)
    else:
        handle_task_failure(task, result.error)
```

**Option B: Direct Execution (Original, Preserved)**
```python
else:
    # Original implementation logic (no agent)
    result = execute_task_directly(task)
    # ... existing logic unchanged
```

### Phase 3: Failure Handling (Enhanced)

**New failure handling per Clarification Q3**:

```python
def handle_task_failure(failed_task, error):
    # Report failure with full context (FR-020)
    report_failure(
        task_id=failed_task.task_id,
        agent=failed_task.agent_assignment,
        error_message=error,
        phase=failed_task.phase
    )

    # Identify dependent tasks in same chain
    dependent_tasks = get_dependency_chain(failed_task)

    # Halt dependent tasks
    for dep_task in dependent_tasks:
        mark_skipped(dep_task, reason=f"Depends on failed {failed_task.task_id}")

    # Identify independent parallel tasks
    independent_tasks = get_parallel_independent_tasks(failed_task)

    # Continue with independent tasks
    for ind_task in independent_tasks:
        # These tasks proceed normally
        continue_execution(ind_task)

    # Summary report
    print(f"Failed: {failed_task.task_id}")
    print(f"Halted (dependent): {[t.task_id for t in dependent_tasks]}")
    print(f"Continuing (independent): {[t.task_id for t in independent_tasks]}")
```

---

## New Output Contract Elements

### Console Output Enhancements

**Task execution progress now includes agent information**:
```
Executing Phase 2: Core Implementation
  ✓ T001: Create User model [database_architect.md] - COMPLETED
  ✓ T002: Create Post model [database_architect.md] - COMPLETED
  ⏩ T003: Create UserService [api_developer.md] - IN PROGRESS
```

**Failure reporting includes agent context**:
```
✗ FAILED: T005 - Create payment endpoint
  Agent: api_developer.md
  Error: ImportError: No module named 'stripe'
  Halted (dependent): T006, T007
  Continuing (independent): T010, T011, T012
```

### tasks.md Updates

**Completed tasks marked with [X]** (existing behavior, preserved):
```markdown
- [X] T001 [P] [US1] Create User model in src/models/user.py [database_architect.md]
```

**Skipped tasks marked with annotation** (new):
```markdown
- [ ] T003 [US1] Create UserService (SKIPPED: depends on failed T002)
```

---

## New Error Conditions

### Agent Delegation Errors

**ERR-030: Agent file not found**
- **Condition**: tasks.md references agent file that doesn't exist in .claude/agents/
- **Message**: `"Agent file not found: .claude/agents/api_developer.md. Run /provide-claude-team to regenerate."`
- **Exit Code**: 1
- **Recovery**: Run `/provide-claude-team` to create/regenerate agent files

**ERR-031: Agent file malformed**
- **Condition**: Agent file exists but missing required frontmatter or sections
- **Message**: `"Agent file .claude/agents/api_developer.md is malformed. Missing required frontmatter field: 'name'"`
- **Exit Code**: 1
- **Recovery**: Manually fix agent file or delete and re-run `/provide-claude-team`

**ERR-032: Agent context load failure**
- **Condition**: Cannot read or parse agent file
- **Message**: `"Failed to load agent context from .claude/agents/api_developer.md: [OS error]"`
- **Exit Code**: 1
- **Recovery**: Check file permissions and integrity

### Task Execution Errors (Enhanced)

**ERR-040: Task delegation failure**
- **Condition**: Agent executes task but returns error
- **Message**: `"Task T005 failed when delegated to api_developer.md: [agent error message]"`
- **Exit Code**: 0 (continues with independent tasks per FR-020)
- **Side Effect**: Dependent tasks halted, independent tasks continue

---

## Backward Compatibility

### Scenario 1: tasks.md without agent assignments
```markdown
- [ ] T001 Create User model in src/models/user.py
```
**Behavior**: Direct implementation (original mode)
**Result**: No change from existing `/implement` behavior

### Scenario 2: Mixed tasks (some with agents, some without)
```markdown
- [ ] T001 Create User model [database_architect.md]
- [ ] T002 Create config file (no agent assignment)
```
**Behavior**:
- T001: Delegated to database_architect
- T002: Direct implementation
**Result**: Hybrid execution mode

### Scenario 3: .claude/agents/ directory doesn't exist
**If tasks.md has no agent assignments**: No error (direct mode)
**If tasks.md has agent assignments**: ERR-030 (missing agent files)

---

## Performance Impact

### Additional Overhead
- **Agent file loading**: ~10-50ms per unique agent (loaded once, reused for multiple tasks)
- **Agent context switching**: Minimal (native Claude Code mechanism)
- **Validation step**: O(n) where n = unique agents (check file existence)

### Total Impact
- **Negligible for most features**: < 1 second overhead for typical 20-50 tasks
- **Scales with agent diversity**: More unique agents = more file loads (but still fast)

---

## Modified Testing Requirements

### New Unit Tests
- **Test-Impl-1**: Parse agent assignments from tasks.md
- **Test-Impl-2**: Load agent file and extract context
- **Test-Impl-3**: Execute task with agent delegation (mocked)
- **Test-Impl-4**: Handle agent file not found error
- **Test-Impl-5**: Fall back to direct mode when no assignments
- **Test-Impl-6**: Halt dependent tasks, continue independent on failure

### Modified Integration Tests
- **Test-Impl-7**: End-to-end with agent delegation (tasks.md with assignments → code generated)
- **Test-Impl-8**: Mixed mode (some tasks with agents, some without)
- **Test-Impl-9**: Failure scenario (one task fails, verify halt/continue behavior)

---

## Migration Path

### For Existing Projects
1. **No action required**: Existing features without agent assignments continue to work
2. **Opt-in per feature**: New features can use `/provide-claude-team` to get agent assignments
3. **Gradual adoption**: Can run `/provide-claude-team` on old features to retrofit agents

### Breaking Changes
**None**. This is a backward-compatible enhancement.

---

## Example Execution Flow

### With Agent Delegation
```
$ /speckit.implement

Checking prerequisites...
✓ tasks.md found
✓ plan.md found
✓ Agent assignments detected (3 unique agents)

Validating agent files...
✓ database_architect.md
✓ api_developer.md
✓ frontend_specialist.md

Loading implementation context...
✓ tasks.md (15 tasks)
✓ plan.md
✓ data-model.md
✓ research.md

Executing Phase 1: Setup
  ✓ T001: Initialize project structure - COMPLETED (direct mode)

Executing Phase 2: Core Implementation
  ⏩ T010: Create User model [database_architect.md] - IN PROGRESS
     [database_architect.md] Generating model with fields: id, email, password_hash...
  ✓ T010: Create User model [database_architect.md] - COMPLETED

  ⏩ T011: Create Post model [database_architect.md] - IN PROGRESS
  ✓ T011: Create Post model [database_architect.md] - COMPLETED

  ⏩ T015: Create UserService [api_developer.md] - IN PROGRESS
  ✗ T015: Create UserService [api_developer.md] - FAILED
     Error: Missing dependency 'user_model'
     Halted (dependent): T016, T017
     Continuing (independent): T020, T021

Execution Summary:
  Completed: 12 tasks
  Failed: 1 task (T015)
  Skipped: 2 tasks (T016, T017 - dependency failed)
  Agents used: database_architect, api_developer, frontend_specialist
```

---

This contract defines all modifications to the `/implement` command to support agent delegation while maintaining full backward compatibility.
