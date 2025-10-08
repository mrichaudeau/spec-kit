---
description: Execute the implementation plan by processing and executing all tasks defined in tasks.md
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

1. Run `.specify/scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks` from repo root and parse FEATURE_DIR and AVAILABLE_DOCS list. All paths must be absolute.

2. **Check checklists status** (if FEATURE_DIR/checklists/ exists):
   - Scan all checklist files in the checklists/ directory
   - For each checklist, count:
     * Total items: All lines matching `- [ ]` or `- [X]` or `- [x]`
     * Completed items: Lines matching `- [X]` or `- [x]`
     * Incomplete items: Lines matching `- [ ]`
   - Create a status table:
     ```
     | Checklist | Total | Completed | Incomplete | Status |
     |-----------|-------|-----------|------------|--------|
     | ux.md     | 12    | 12        | 0          | ✓ PASS |
     | test.md   | 8     | 5         | 3          | ✗ FAIL |
     | security.md | 6   | 6         | 0          | ✓ PASS |
     ```
   - Calculate overall status:
     * **PASS**: All checklists have 0 incomplete items
     * **FAIL**: One or more checklists have incomplete items
   
   - **If any checklist is incomplete**:
     * Display the table with incomplete item counts
     * **STOP** and ask: "Some checklists are incomplete. Do you want to proceed with implementation anyway? (yes/no)"
     * Wait for user response before continuing
     * If user says "no" or "wait" or "stop", halt execution
     * If user says "yes" or "proceed" or "continue", proceed to step 3
   
   - **If all checklists are complete**:
     * Display the table showing all checklists passed
     * Automatically proceed to step 3

3. Load and analyze the implementation context:
   - **REQUIRED**: Read tasks.md for the complete task list and execution plan
   - **REQUIRED**: Read plan.md for tech stack, architecture, and file structure
   - **IF EXISTS**: Read data-model.md for entities and relationships
   - **IF EXISTS**: Read contracts/ for API specifications and test requirements
   - **IF EXISTS**: Read research.md for technical decisions and constraints
   - **IF EXISTS**: Read quickstart.md for integration scenarios

4. Parse tasks.md structure and extract:
   - **Task phases**: Setup, Tests, Core, Integration, Polish
   - **Task dependencies**: Sequential vs parallel execution rules
   - **Task details**: ID, description, file paths, parallel markers [P]
   - **Agent assignments**: Extract `[agent_filename.md]` annotations if present
   - **Execution flow**: Order and dependency requirements

   **NEW - Agent Assignment Detection**:
   - Check each task line for `[{agent}.md]` pattern at end of line
   - If present: Task uses **delegated mode** (execute with specialist agent)
   - If absent: Task uses **direct mode** (original implementation behavior)
   - Parse pattern: `\[(.+?\.md)\]$` to extract agent filename

   **NEW - Agent File Validation**:
   - If any agent assignments detected:
     - Scan `.claude/agents/` to collect all unique agent filenames referenced
     - Validate each agent file exists
     - If missing: **ERROR** "Agent file not found: .claude/agents/{agent}.md. Run /provide-claude-team to regenerate."
   - If no agent assignments: Skip validation (backward compatible mode)

5. Execute implementation following the task plan:
   - **Phase-by-phase execution**: Complete each phase before moving to the next
   - **Respect dependencies**: Run sequential tasks in order, parallel tasks [P] can run together
   - **Follow TDD approach**: Execute test tasks before their corresponding implementation tasks
   - **File-based coordination**: Tasks affecting the same files must run sequentially
   - **Validation checkpoints**: Verify each phase completion before proceeding

   **NEW - Task Execution Modes**:

   **A) Delegated Mode** (when task has `[agent.md]` assignment):
   ```
   For task with agent assignment:
   1. Read agent file from `.claude/agents/{agent_filename}.md`
   2. Extract agent's full context (frontmatter + all sections)
   3. Execute task using agent's specialized context:
      "You are acting as the agent defined in .claude/agents/{agent_filename}.md.

       Agent context:
       {full_agent_file_content}

       Now execute this task:
       Task ID: {task_id}
       Description: {task_description}
       File paths: {file_paths}

       Generate the code and indicate which files were created/modified."
   4. Collect generated code from agent's response
   5. Write code to specified file paths
   6. Mark task as [X] completed in tasks.md
   ```

   **B) Direct Mode** (when task has NO agent assignment):
   ```
   Execute task directly (original behavior):
   1. Analyze task requirements
   2. Generate implementation
   3. Write to file paths
   4. Mark task as [X] completed
   ```

   **Backward Compatibility**: If tasks.md has no agent assignments at all, entire file runs in direct mode (original /implement behavior preserved)

6. Implementation execution rules:
   - **Setup first**: Initialize project structure, dependencies, configuration
   - **Tests before code**: If you need to write tests for contracts, entities, and integration scenarios
   - **Core development**: Implement models, services, CLI commands, endpoints
   - **Integration work**: Database connections, middleware, logging, external services
   - **Polish and validation**: Unit tests, performance optimization, documentation

7. Progress tracking and error handling:
   - Report progress after each completed task
   - **IMPORTANT** For completed tasks, make sure to mark the task off as [X] in the tasks file.

   **NEW - Enhanced Progress Reporting** (when using agents):
   ```
   Executing Phase 2: Core Implementation
     ⏩ T010: Create User model [database_architect.md] - IN PROGRESS
        [database_architect.md] Analyzing schema requirements...
     ✓ T010: Create User model [database_architect.md] - COMPLETED
        Files created: src/models/user.py
   ```

   **NEW - Enhanced Failure Handling** (per clarification Q3):

   When a task fails:

   1. **Report failure with full context**:
      ```
      ✗ FAILED: T015 - Create payment endpoint
        Agent: api_developer.md (if delegated) or "direct mode"
        Error: {error_message}
        File: {file_path_attempted}
      ```

   2. **Identify dependency chains**:
      - Tasks in same phase without [P] marker → sequential dependency
      - Tasks modifying same file paths → file dependency
      - Later phase tasks → phase dependency

   3. **Halt only dependent tasks** (not everything):
      - Mark dependent tasks in same chain as SKIPPED
      - Reason: "Depends on failed {task_id}"
      - Continue executing independent parallel tasks

   4. **Report impact**:
      ```
      Impact Analysis:
        Halted (dependent): T016, T017, T018
        Continuing (independent): T020, T021, T025
      ```

   5. **For parallel tasks [P]**: Continue with successful ones, report failed

   **Backward Compatible**: Non-agent tasks continue with original error handling

8. Completion validation:
   - Verify all required tasks are completed
   - Check that implemented features match the original specification
   - Validate that tests pass and coverage meets requirements
   - Confirm the implementation follows the technical plan
   - Report final status with summary of completed work

Note: This command assumes a complete task breakdown exists in tasks.md. If tasks are incomplete or missing, suggest running `/tasks` first to regenerate the task list.
