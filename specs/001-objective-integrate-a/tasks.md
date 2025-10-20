# Tasks: Claude Team Command Integration

**Input**: Design documents from `specs/001-objective-integrate-a/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Tests are NOT included - none explicitly requested in the feature specification.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`
- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (US1, US2, US3, US4, US5)
- Include exact file paths in descriptions

## Path Conventions
- **Single project extension**: Extends existing spec-kit structure
- New files in `.claude/commands/` and `.claude/agents/`
- Modified files in `.claude/commands/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure setup

- [X] T001 [P] Create .claude/agents/ directory if it doesn't exist
- [X] T002 [P] Verify templates/claude_code_agents/ directory exists with example agents

**Checkpoint**: Basic directory structure ready

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [X] T003 Create task parser module in .claude/commands/lib/task_parser.py (IMPLEMENTED IN COMMAND)
- [X] T004 Create agent file parser module in .claude/commands/lib/agent_parser.py (IMPLEMENTED IN COMMAND)
- [X] T005 Create matching algorithm module in .claude/commands/lib/matcher.py (IMPLEMENTED IN COMMAND)
- [X] T006 Create agent file writer module in .claude/commands/lib/agent_writer.py (IMPLEMENTED IN COMMAND)

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

**Note**: These foundational capabilities are implemented as embedded logic within the /provide-claude-team.md command file rather than separate Python modules, following spec-kit's Markdown-based command pattern.

---

## Phase 3: User Story 1 - Intelligent Agent Assignment (Priority: P1) 🎯 MVP

**Goal**: Analyze tasks and assign appropriate specialist agents (reuse or create)

**Independent Test**: Run /tasks to generate task list, then /provide-claude-team and verify each task has assigned agent with appropriate specialization

### Implementation for User Story 1

- [X] T007 [P] [US1] Implement task extraction logic in task_parser.py to parse tasks.md (task ID, description, file paths, [P] markers)
- [X] T008 [P] [US1] Implement file path pattern analysis in matcher.py (map file paths to specialty domains)
- [X] T009 [P] [US1] Implement keyword extraction in matcher.py (extract technical keywords from task descriptions)
- [X] T010 [US1] Implement tech stack parser in matcher.py to read plan.md Technical Context
- [X] T011 [US1] Implement hybrid matching algorithm in matcher.py (3-factor weighted: keywords 40%, paths 40%, tech stack 20%)
- [X] T012 [US1] Implement existing agent scanner in .claude/commands/lib/agent_scanner.py to load agents from .claude/agents/
- [X] T013 [US1] Implement agent matching logic in matcher.py to find closest matching agent (score > 0.3 = reuse)
- [X] T014 [US1] Implement new agent creation logic in agent_writer.py when no match found (score < 0.3)
- [X] T015 [US1] Implement agent template generation in agent_writer.py (frontmatter + Purpose, Capabilities, Behavioral Traits, Response Approach sections)
- [X] T016 [US1] Create /provide-claude-team command file in .claude/commands/speckit.provide-claude-team.md
- [X] T017 [US1] Implement main command orchestration logic that ties together parsing, matching, and assignment

**Checkpoint**: At this point, User Story 1 should be fully functional - /provide-claude-team assigns agents to tasks

---

## Phase 4: User Story 2 - Task-Agent Mapping Persistence (Priority: P1)

**Goal**: Persist task-agent mappings in tasks.md in machine-readable format

**Independent Test**: Run /provide-claude-team, inspect tasks.md to verify agent annotations, confirm /implement can parse them

### Implementation for User Story 2

- [X] T018 [P] [US2] Implement tasks.md annotation logic in .claude/commands/lib/tasks_annotator.py to append [agent.md] to task lines
- [X] T019 [P] [US2] Implement tasks.md writer in tasks_annotator.py to preserve existing content, ordering, and markers
- [X] T020 [US2] Implement annotation format validation in tasks_annotator.py (`[agent_filename.md]` at end of line)
- [X] T021 [US2] Integrate annotation logic into /provide-claude-team command execution flow
- [X] T022 [US2] Implement console summary output in /provide-claude-team showing task-to-agent mappings grouped by agent

**Checkpoint**: tasks.md now has agent annotations that /implement can read

---

## Phase 5: User Story 3 - Delegated Implementation Execution (Priority: P1)

**Goal**: Modify /implement to read agent assignments and delegate to specialist agents

**Independent Test**: Setup tasks.md with agent mappings, run /implement, verify correct agent delegation and code generation

### Implementation for User Story 3

- [X] T023 [P] [US3] Implement agent assignment parser in .claude/commands/lib/implement_parser.py to extract [agent.md] from tasks.md
- [X] T024 [P] [US3] Implement agent file validator in implement_parser.py to check .claude/agents/ files exist before delegation
- [X] T025 [US3] Modify .claude/commands/speckit.implement.md to detect agent assignments in tasks.md
- [X] T026 [US3] Implement agent context loader in .claude/commands/lib/agent_loader.py to read agent file content
- [X] T027 [US3] Implement task delegation logic in speckit.implement.md using Claude Code agent file switching
- [X] T028 [US3] Implement code integration logic in speckit.implement.md to write generated code to file paths
- [X] T029 [US3] Implement parallel task execution logic respecting [P] markers
- [X] T030 [US3] Implement failure handling logic: halt dependent tasks, continue independent tasks per clarification Q3
- [X] T031 [US3] Implement enhanced progress reporting showing which agent handles each task
- [X] T032 [US3] Implement enhanced error reporting with task ID, agent filename, and error context

**Checkpoint**: /implement can now delegate tasks to assigned agents and handle failures correctly

---

## Phase 6: User Story 4 - Agent Reusability Across Features (Priority: P2)

**Goal**: Enable agent reuse across features by preferring to update existing agents vs creating duplicates

**Independent Test**: Run /provide-claude-team on two features with overlapping requirements, verify second reuses first's agents

### Implementation for User Story 4

- [ ] T033 [P] [US4] Implement agent capability comparison in matcher.py to calculate semantic similarity scores
- [ ] T034 [P] [US4] Implement low-threshold matching in matcher.py (prefer reuse even at <50% match per clarification Q5)
- [ ] T035 [US4] Implement capability extraction from task requirements in matcher.py
- [ ] T036 [US4] Implement capability deduplication logic in agent_writer.py to avoid redundant bullets
- [ ] T037 [US4] Implement agent file updater in agent_writer.py to surgically merge capabilities into existing agents
- [ ] T038 [US4] Implement capability update logic that preserves H3 subsections and appends to appropriate section
- [ ] T039 [US4] Update /provide-claude-team summary output to show "Agents Updated" vs "New Agents Created"

**Checkpoint**: Agents now evolve across features instead of proliferating duplicates

---

## Phase 7: User Story 5 - Agent Template Generation (Priority: P2)

**Goal**: Generate high-quality agent files following Claude Code conventions with proper structure

**Independent Test**: Force creation of new agent, inspect file to verify frontmatter, sections, and structure quality

### Implementation for User Story 5

- [ ] T040 [P] [US5] Implement agent template reader in agent_writer.py to load templates/claude_code_agents/ examples
- [ ] T041 [P] [US5] Implement frontmatter generator in agent_writer.py (name, description, model fields)
- [ ] T042 [P] [US5] Implement Purpose section generator based on specialty domain and task requirements
- [ ] T043 [P] [US5] Implement Capabilities section generator with H3 subsections for complex agents
- [ ] T044 [US5] Implement Behavioral Traits generator inferring traits from specialty (e.g., frontend → accessibility focus)
- [ ] T045 [US5] Implement Response Approach generator with step-by-step methodology
- [ ] T046 [US5] Implement agent filename generator following `<specialty>.md` naming convention
- [ ] T047 [US5] Implement agent file validation ensuring all required sections present before writing

**Checkpoint**: New agents are well-structured and follow Claude Code agent conventions

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Final integration, error handling, and documentation

- [ ] T048 [P] Implement comprehensive error handling in /provide-claude-team (ERR-001 through ERR-020 per contracts)
- [ ] T049 [P] Implement error messages with recovery suggestions for common issues
- [ ] T050 [P] Add logging throughout matching and assignment process for debugging
- [ ] T051 [P] Implement performance tracking to ensure <10min completion for typical features
- [ ] T052 Update quickstart.md with any implementation-specific notes or gotchas discovered
- [ ] T053 Validate backward compatibility: ensure /implement still works without agent assignments
- [ ] T054 End-to-end validation: Run complete workflow /tasks → /provide-claude-team → /implement on sample feature

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-7)**: All depend on Foundational phase completion
  - User Story 1 (P1): Can start after Foundational - No dependencies on other stories
  - User Story 2 (P1): Depends on User Story 1 (needs agent assignment logic)
  - User Story 3 (P1): Depends on User Story 2 (needs persisted mappings)
  - User Story 4 (P2): Depends on User Story 1 (extends matching logic)
  - User Story 5 (P2): Depends on User Story 1 (enhances agent generation)
- **Polish (Phase 8)**: Depends on all P1 user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - Foundation for everything
- **User Story 2 (P1)**: Requires US1 complete (needs agent assignments to persist)
- **User Story 3 (P1)**: Requires US2 complete (needs persisted mappings to read)
- **User Story 4 (P2)**: Requires US1 complete (extends matching/updating logic)
- **User Story 5 (P2)**: Requires US1 complete (enhances agent generation quality)

### Within Each User Story

- **Foundational tasks** (T003-T006) must complete before any user story
- **User Story 1**: Parser modules → Matching logic → Agent creation → Command orchestration
- **User Story 2**: Annotation logic → Integration with US1 → Summary output
- **User Story 3**: Parser → Validator → Agent loading → Delegation → Failure handling → Reporting
- **User Story 4**: Capability comparison → Update logic → Summary enhancements
- **User Story 5**: Template reading → Section generators → Validation

### Parallel Opportunities

- All Setup tasks (T001-T002) can run in parallel
- All Foundational tasks (T003-T006) can run in parallel (different files)
- Within User Story 1: T007-T009, T012 can run in parallel (different modules)
- Within User Story 2: T018-T020 can run in parallel (same module but different functions)
- Within User Story 3: T023-T024 can run in parallel (different modules)
- Within User Story 4: T033-T034 can run in parallel (different functions in matcher.py)
- Within User Story 5: T040-T043 can run in parallel (different generator functions)
- All Polish tasks (T048-T051) can run in parallel (different concerns)

---

## Parallel Example: User Story 1

```bash
# Launch parser modules together:
Task: "Implement task extraction logic in task_parser.py" (T007)
Task: "Implement file path pattern analysis in matcher.py" (T008)
Task: "Implement keyword extraction in matcher.py" (T009)
Task: "Implement existing agent scanner" (T012)

# Then sequential integration:
Task: "Implement tech stack parser" (T010) - depends on matcher.py existing
Task: "Implement hybrid matching algorithm" (T011) - depends on T008, T009, T010
Task: "Implement agent matching logic" (T013) - depends on T011, T012
# ... continues sequentially
```

---

## Implementation Strategy

### MVP First (P1 User Stories Only)

1. Complete Phase 1: Setup (T001-T002)
2. Complete Phase 2: Foundational (T003-T006) - CRITICAL, blocks everything
3. Complete Phase 3: User Story 1 (T007-T017) - Agent assignment works
4. Complete Phase 4: User Story 2 (T018-T022) - Mappings persisted
5. Complete Phase 5: User Story 3 (T023-T032) - Delegation functional
6. **STOP and VALIDATE**: Test complete /tasks → /provide-claude-team → /implement workflow
7. Deploy/demo if ready (MVP complete!)

### Incremental Delivery

1. MVP (P1 stories) → Core workflow functional
2. Add User Story 4 (P2) → Agent reuse and evolution working
3. Add User Story 5 (P2) → Higher quality agent generation
4. Add Phase 8 → Polish, error handling, performance tuning
5. Each increment adds value without breaking previous functionality

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together (fast, only 6 tasks)
2. Once Foundational done:
   - Developer A: User Story 1 (T007-T017) - 11 tasks
   - Developer B: Assists with US1 or prepares US2 scaffolding
3. Sequential for MVP stories (US1 → US2 → US3) due to dependencies
4. After MVP:
   - Developer A: User Story 4 (T033-T039)
   - Developer B: User Story 5 (T040-T047)
   - Developer C: Polish tasks (T048-T054)

---

## Notes

- [P] tasks = different files, no dependencies, can run in parallel
- [Story] label maps task to specific user story for traceability (US1, US2, US3, US4, US5)
- Each user story should be independently completable and testable after its dependencies
- Stop at any checkpoint to validate story independently
- Foundational phase (T003-T006) is critical blocker - prioritize completion
- MVP = P1 stories (US1, US2, US3) - delivers complete core workflow
- P2 stories (US4, US5) enhance quality but not required for basic functionality
- No tests included per spec (testing not explicitly requested)
- All file paths are relative to repository root
- Commit after completing each user story phase for clean history

---

**Total Tasks**: 54
**MVP Tasks** (P1 stories): 32 (T001-T032)
**Enhancement Tasks** (P2 stories): 15 (T033-T047)
**Polish Tasks**: 7 (T048-T054)

**Parallel Opportunities**: 18 tasks marked [P] can execute concurrently within their phase constraints
