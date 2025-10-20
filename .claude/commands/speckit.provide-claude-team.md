---
description: Analyze tasks and assign specialized Claude Code agents, creating or updating agent files as needed
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

Goal: Automatically assign specialized agents to each task in tasks.md by analyzing task requirements and either reusing existing agents or creating new ones.

Execution steps:

1. **Setup and Prerequisites**:
   - Run `.specify/scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks` from repo root
   - Parse JSON output to get FEATURE_DIR path
   - Validate that tasks.md exists at `{FEATURE_DIR}/tasks.md`
   - Validate that plan.md exists at `{FEATURE_DIR}/plan.md`
   - Create `.claude/agents/` directory if it doesn't exist

2. **Load Context**:
   - **Read tasks.md**: Parse all task entries
     - Extract task ID (T001, T002, etc.)
     - Extract task description
     - Extract file paths mentioned in backticks (e.g., `src/models/user.py`)
     - Extract [P] parallel markers
     - Extract [US1], [US2] user story tags
     - Note: Use regex pattern: `^- \[([ Xx])\] (T\d+) (\[P\])? (\[US\d+\])? (.+?)$`

   - **Read plan.md**: Extract tech stack from Technical Context section
     - Language/Version
     - Primary Dependencies
     - Framework choices
     - This provides context for matching (factor 3 of hybrid algorithm)

   - **Scan .claude/agents/**: Load all existing agent files
     - For each .md file in .claude/agents/:
       - Parse YAML frontmatter (name, description, model)
       - Parse ## Capabilities section (extract all bullet points)
       - Store as candidate for reuse matching

3. **For Each Task - Perform Agent Matching**:

   Use the **3-factor hybrid matching algorithm** (from research.md):

   **Factor 1: Keyword Extraction (40% weight)**
   - Extract technical keywords from task description:
     - "API", "endpoint", "REST" → api_developer
     - "frontend", "component", "UI", "React" → frontend_specialist
     - "database", "model", "schema", "SQL" → database_architect
     - "test", "testing", "pytest" → test_specialist
     - "deploy", "CI/CD", "pipeline" → deployment_engineer
   - Score: `keyword_matches / total_technical_keywords`

   **Factor 2: File Path Pattern Analysis (40% weight)**
   - Analyze file paths from task description:
     - `src/api/*` or `*/api/*` → api_developer
     - `src/components/*` or `*/frontend/*` → frontend_specialist
     - `src/models/*` or `*/database/*` → database_architect
     - `tests/*` or `*test*.py` → test_specialist
     - `.github/workflows/*` or `deploy/*` → deployment_engineer
   - Score: `matching_path_patterns / total_file_paths`

   **Factor 3: Tech Stack Alignment (20% weight)**
   - Check if task requirements align with tech stack from plan.md
     - If task involves API and plan.md mentions "FastAPI" → api_developer bonus
     - If task involves frontend and plan.md mentions "React" → frontend_specialist bonus
   - Score: `1.0` if aligned, `0.0` if not

   **Combined Score**: `0.4 * keyword_score + 0.4 * path_score + 0.2 * tech_score`

   **Matching Decision** (per clarification Q5):
   - If score > 0.3: **REUSE** closest matching existing agent + UPDATE with new capabilities
   - If score ≤ 0.3: **CREATE** new agent for this specialty domain
   - Always prefer reuse even at low match scores (<50%)

4. **Agent Creation (when no match found)**:

   When creating a new agent file:

   a) **Determine specialty name** from task analysis:
      - API tasks → `api_developer.md`
      - Frontend tasks → `frontend_specialist.md`
      - Database tasks → `database_architect.md`
      - Testing tasks → `test_specialist.md`
      - Deployment tasks → `deployment_engineer.md`
      - General/unclear → `general_purpose.md`

   b) **Generate agent file** at `.claude/agents/{specialty}.md`:

   ```markdown
   ---
   name: {specialty-name}
   description: {One-line purpose based on task requirements and specialty}
   model: sonnet
   ---

   ## Purpose
   {2-3 sentence description of agent's role and expertise}

   ## Capabilities

   ### {Category 1}
   - {Capability extracted from task requirements}
   - {Related capability}

   ### {Category 2}
   - {More capabilities}

   ## Behavioral Traits
   - {How this agent approaches work, inferred from specialty}
   - {Quality standards and priorities}
   - {Best practices adherence}

   ## Response Approach
   1. **{Step 1}**
   2. **{Step 2}**
   3. **{Step 3}**
   ```

   c) **Reference templates/claude_code_agents/** for structure examples but customize based on task needs

5. **Agent Update (when reusing existing agent)**:

   When updating an existing agent with new capabilities:

   a) **Read existing agent file** from `.claude/agents/{agent}.md`

   b) **Extract new capabilities** needed from current task that aren't already in agent:
      - Parse task description for technical requirements
      - Check if capability already exists in agent's ## Capabilities section
      - Use semantic similarity to avoid duplicates

   c) **Merge capabilities** into ## Capabilities section:
      - If agent uses H3 subsections, add to appropriate subsection or create new one
      - If flat bullet list, append to end
      - Preserve existing capabilities (never remove - per FR-026)
      - Maintain formatting and structure

   d) **Write updated agent file** back to `.claude/agents/{agent}.md`

6. **Annotate tasks.md**:

   For each task, append agent assignment to the task line:

   **Original format**:
   ```
   - [ ] T001 [P] [US1] Create User model in src/models/user.py
   ```

   **Annotated format**:
   ```
   - [ ] T001 [P] [US1] Create User model in src/models/user.py [database_architect.md]
   ```

   **Rules**:
   - Append `[{agent_filename}.md]` to end of task line
   - Preserve all existing content (checkbox, markers, description)
   - Maintain line ordering
   - Do NOT modify completed tasks (those with [X])

7. **Generate Summary Report**:

   Output console summary:

   ```
   Agent Assignment Summary
   ========================

   {Specialty} ({count} tasks → {agent_filename}.md):
     - {task_id}: {task_description}
     - {task_id}: {task_description}

   {Another Specialty} ({count} tasks → {agent_filename}.md):
     - ...

   New Agents Created: {count}
     - {agent_filename}.md

   Agents Updated: {count}
     - {agent_filename}.md (added: {new_capabilities})

   Total: {total_tasks} tasks assigned to {total_agents} agents
   ```

8. **Validation**:
   - Verify all tasks have agent assignments
   - Verify all agent files are valid (frontmatter + required sections)
   - Verify tasks.md annotations are parseable

9. **Error Handling**:

   - **ERR-001**: If tasks.md not found → "tasks.md not found. Please run /speckit.tasks first."
   - **ERR-003**: If plan.md not found → "plan.md not found. Please run /speckit.plan first."
   - **ERR-010**: If templates/claude_code_agents/ not found → "Agent templates not found. Check spec-kit installation."
   - **ERR-012**: If cannot write tasks.md → "Failed to update tasks.md: [error]. Check file permissions."

## Implementation Notes

- Use built-in regex for parsing (no external Markdown libraries needed)
- Agent files follow YAML frontmatter + Markdown structure
- Matching threshold of 0.3 ensures aggressive reuse (per clarification Q5)
- Default to general_purpose.md if task is completely ambiguous
- Performance target: <10 minutes for typical feature (variable scale, graceful degradation)

## Success Criteria

After execution:
- ✅ Every task in tasks.md has `[agent.md]` annotation
- ✅ All referenced agent files exist in `.claude/agents/`
- ✅ Agent files have valid structure (frontmatter + required sections)
- ✅ Console summary clearly shows task-to-agent mappings
- ✅ Ready for `/speckit.implement` to read mappings and delegate
