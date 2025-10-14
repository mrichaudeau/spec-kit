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
   - **Documentation-First Approach**: MANDATORY web search before implementation
   - **Follow TDD approach**: Execute test tasks before their corresponding implementation tasks
   - **File-based coordination**: Tasks affecting the same files must run sequentially
   - **Validation checkpoints**: Verify each phase completion before proceeding

   **NEW - MANDATORY Documentation Search Workflow**:

   For EVERY implementation task (before writing ANY code):

   1. **Extract key technologies and concepts from the task**:
      - Programming language, framework, libraries mentioned
      - Design patterns or architectural concepts
      - APIs or services being integrated
      - Data structures or algorithms needed

   2. **Perform web search for documentation**:
      ```
      Use WebSearch tool with query like:
      "[technology] [concept] official documentation best practices 2024"

      Examples:
      - "React hooks useState useEffect official documentation 2024"
      - "Python FastAPI async endpoints error handling best practices"
      - "PostgreSQL jsonb indexes performance optimization documentation"
      - "Node.js Express middleware authentication JWT implementation"
      ```

   3. **Document findings before implementation**:
      - Record which documentation was consulted
      - Note key implementation guidelines discovered
      - Identify any warnings or deprecated approaches
      - List best practices from official sources

   4. **Only THEN proceed with implementation**:
      - Apply the patterns from documentation
      - Follow the official guidelines discovered
      - Avoid deprecated or discouraged approaches

   **NEW - Task Execution Modes**:

   **A) Delegated Mode** (when task has `[agent.md]` assignment):
   ```
   For task with agent assignment:
   1. MANDATORY: Search for documentation
      - Extract technologies from task description
      - Use WebSearch: "[tech] [feature] official documentation 2024"
      - Document key findings and best practices
   2. Read agent file from `.claude/agents/{agent_filename}.md`
   3. Extract agent's full context (frontmatter + all sections)
   4. Execute task using agent's specialized context:
      "You are acting as the agent defined in .claude/agents/{agent_filename}.md.

       Agent context:
       {full_agent_file_content}

       Documentation consulted:
       {documentation_findings}

       Now execute this task following the documented best practices:
       Task ID: {task_id}
       Description: {task_description}
       File paths: {file_paths}

       Generate the code and indicate which files were created/modified."
   5. Collect generated code from agent's response
   6. Write code to specified file paths
   7. Mark task as [X] completed in tasks.md
   ```

   **B) Direct Mode** (when task has NO agent assignment):
   ```
   Execute task directly with documentation search:
   1. MANDATORY: Search for documentation
      - Extract technologies from task description
      - Use WebSearch: "[tech] [feature] official documentation 2024"
      - Document key findings and best practices
   2. Analyze task requirements with documentation context
   3. Generate implementation following documented patterns
   4. Write to file paths
   5. Mark task as [X] completed
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

   **NEW - Documentation Verification & Progress Reporting**:
   ```
   Executing Phase 2: Core Implementation
     📚 T010: Searching documentation for "Python SQLAlchemy User model best practices"...
        Found: Official SQLAlchemy ORM documentation
        Found: Best practices for user authentication models
        Key insight: Use declarative_base() for model definitions
     ⏩ T010: Create User model [database_architect.md] - IN PROGRESS
        [database_architect.md] Implementing with documented patterns...
     ✓ T010: Create User model [database_architect.md] - COMPLETED
        Documentation consulted: SQLAlchemy 2.0 docs, security best practices
        Files created: src/models/user.py
   ```

   **Documentation Search Verification**:
   - MUST show documentation search step for EVERY task
   - MUST list what documentation was found
   - MUST show key insights or patterns discovered
   - If no relevant documentation found, MUST still search and report "No specific documentation found, using general [language] patterns"

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
   - **Verify documentation was consulted for each task**:
     ```
     Documentation Compliance Report:
     ✓ T001: Consulted 2 documentation sources
     ✓ T002: Consulted 3 documentation sources
     ✗ T003: No documentation search performed (VIOLATION)

     Compliance Rate: 66% (2/3 tasks followed documentation-first approach)
     ```
   - Check that implemented features match the original specification
   - Validate that tests pass and coverage meets requirements
   - Confirm the implementation follows the technical plan
   - **Confirm implementations follow documented best practices**
   - Report final status with summary of completed work and documentation sources used

Note: This command assumes a complete task breakdown exists in tasks.md. If tasks are incomplete or missing, suggest running `/tasks` first to regenerate the task list.
