# Tasks: Phase 1 MVP - Foundation Implementation

**Architecture**: v2.0.0 (Agent-based with Books system)  
**Input**: Design documents from `/specs/001-phase1-mvp-tasks/`  
**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md)  
**Feature Branch**: `001-phase1-mvp-implementation`

**Phase 1 Focus**: HITL (Human-in-the-Loop) workflows with Cline + Ollama Local LLM in VS Code Dev Container. Autonomous mode (Phase 2) deferred.

**Tests**: Integration tests for constitutional compliance and complete HITL workflows. Unit tests for agent implementations (>80% coverage target).

**Organization**: Tasks are grouped by architectural component (Books, Agents, Tools, Skills) and user story to enable independent implementation.

## Format: `- [ ] [ID] [P?] [Story?] Description`

- **Checkbox**: `- [ ]` for tracking completion
- **[ID]**: Sequential task number (T001, T002, etc.)
- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (US1, US2, US3, US4)
- **File path**: Exact location in project structure

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure required for all features

- [ ] T001 Create Python package structure with reshark/ directory and pyproject.toml configuration
- [ ] T002 Initialize Poetry project with Python 3.12+ and add core dependencies (pydantic, pytest, ollama-python, tshark wrappers)
- [ ] T003 [P] Create Books workspace directory structure: books/{notebook/sessions,rulebook/grammars,rulebook/schemas,cookbook/methods,skills/}
- [ ] T004 [P] Configure .gitignore to exclude books/notebook/, books/rulebook/, artifacts/, *.pcap, *.pcapng, and __pycache__
- [ ] T005 [P] Setup pytest configuration in pyproject.toml with coverage reporting and async support
- [ ] T006 [P] Create tests/ directory structure: tests/{agents/,books/,tools/,integration/,autonomous/,performance/,fixtures/}
- [ ] T007 [P] Create Dev Container configuration in .devcontainer/ with docker-compose.yml (VS Code + tshark + Ollama + Cline)
- [ ] T008 [P] Create runtime.yaml configuration file for agent settings and Ollama endpoints
- [ ] T009 [P] Create initial Skills library structure: books/skills/{protocol/,pattern/,binary/,grammar/,README.md}

**Checkpoint**: Project structure ready - foundational phase can begin

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story implementation

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

### Books System Core

- [ ] T010 Create Books system base module at reshark/books/__init__.py with BooksAccess class providing unified interface
- [ ] T011 [P] Implement Notebook class in reshark/books/notebook/ with JSON file storage and session-based directory isolation (UUID)
- [ ] T012 [P] Implement Rulebook class in reshark/books/rulebook/ with grammar versioning (.ksy or .py) and cross-sample validation metadata
- [ ] T013 [P] Implement Cookbook class in reshark/books/cookbook/ with Markdown workflow documentation (read-only)
- [ ] T014 [P] Implement Skills class in reshark/books/skills/ with markdown methodology file loading for LLM context

### Schemas and Validation

- [ ] T015 Create schemas module at reshark/schemas/__init__.py
- [ ] T016 [P] Define Notebook JSON schema with timestamp, agent, operation, and data fields (matches books-schemas.json from plan.md)
- [ ] T017 [P] Define Pattern schema with pattern_id, claim, confidence, and evidence array (minimum 1 sample)
- [ ] T018 [P] Define Grammar schema with name, version, format (kaitai/python), content, and validation_samples array (minimum 3)

### Base Agent Framework

- [ ] T019 Create agents module at reshark/agents/__init__.py
- [ ] T020 Implement abstract BaseAgent class in reshark/agents/base.py with BooksAccess, execute() method, get_required_skills(), invoke_tool()
- [ ] T021 Add Books access methods to BaseAgent: read_notebook(), write_notebook(), read_rulebook(), write_rulebook() (GrammarValidator only)
- [ ] T022 Add skill loading method to BaseAgent: read_skill() returning markdown content for LLM context

### Utilities and Logging

- [ ] T023 Create utils module at reshark/utils/__init__.py
- [ ] T024 [P] Implement evidence trail logging in reshark/utils/logging.py with timestamp and agent attribution
- [ ] T025 [P] Implement input validation utilities in reshark/utils/validation.py for artifact file checks (PCAP, binary)
- [ ] T026 [P] Implement session management utilities in reshark/utils/session.py for UUID generation and cleanup

### Tool Integration Layer

- [ ] T027 Create tools module at reshark/tools/__init__.py
- [ ] T028 Implement ToolsRegistry class in reshark/tools/registry.py with unified tool invocation interface
- [ ] T029 [P] Implement tshark_wrapper.py with command execution, output parsing, and error handling
- [ ] T030 [P] Implement binary_tools.py with wrappers for hexdump, strings, and file commands
- [ ] T031 [P] Implement entropy.py with Shannon entropy calculation utilities

**Checkpoint**: Foundation ready - research and design phase can begin

---

## Phase 2.5: Research & Design (Phase 0-1 Outputs)

**Purpose**: Technical research and architectural design before implementation

**⚠️ NOTE**: These tasks generate artifacts required by plan.md Phase 0-1. Can be done in parallel with or after Foundational phase.

### Research Phase (Phase 0)

- [ ] T032 [P] Research tool wrapper strategies (subprocess vs python libraries) and document ToolsRegistry design in specs/001-phase1-mvp-tasks/research.md
- [ ] T033 [P] Research Skills library organization (taxonomy, file naming, versioning) and document design patterns in research.md
- [ ] T034 [P] Research agent coordination patterns (function calls vs message passing) and document communication architecture in research.md
- [ ] T035 [P] Research session isolation mechanisms (UUID vs timestamp directories) and document design in research.md
- [ ] T036 [P] Research grammar validation frameworks (Kaitai Struct vs Python parsers) and document cross-sample validation strategy in research.md
- [ ] T037 [P] Research Ollama integration with Cline (HTTP API vs native) and document HITL workflow patterns in research.md
- [ ] T038 [P] Research OpenHands integration for Phase 2 autonomous mode and document adapter architecture in research.md
- [ ] T039 Document known unknowns resolution in research.md (tshark filter expressions, entropy thresholds, skills taxonomy, Patternbook embeddings)

### Design Phase (Phase 1)

- [ ] T040 [P] Create data-model.md in specs/001-phase1-mvp-tasks/ documenting core entities (Agent, Session, Observation, Pattern, ValidationResult, Grammar, Skill, ToolResult)
- [ ] T041 [P] Document entity relationships in data-model.md (Agent → Books, Session → Observation → Pattern → Grammar flows, Agent → Skills → LLM)
- [ ] T042 [P] Document state transitions in data-model.md (Session → Observation → Pattern → Grammar promotion lifecycle)
- [ ] T043 [P] Create contracts/ directory in specs/001-phase1-mvp-tasks/ with agent-interface.md defining BaseAgent class contract
- [ ] T044 [P] Create books-schemas.json in contracts/ defining Notebook, Pattern, and Grammar JSON schemas (matches plan.md)
- [ ] T045 [P] Create validation-contract.md in contracts/ defining GrammarValidator requirements (hardcoded 3-sample minimum, cross-sample metrics)
- [ ] T046 Create quickstart.md in specs/001-phase1-mvp-tasks/ with Dev Container setup, Cline configuration, and first HITL analysis walkthrough

**Checkpoint**: Research and design artifacts complete - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Skills-Guided Protocol Analysis Workflow (Priority: P1) 🎯 MVP

**Goal**: Enable end-to-end HITL protocol analysis using Cline + Ollama with skills-guided agent workflows

**Independent Test**: Provide artifact (PCAP/binary), run HITL workflow with Cline (DataInterpreter → PatternInterpreter → GrammarValidator), verify grammar in Rulebook

### Skills Library Foundation (US1)

- [ ] T047 [P] [US1] Create protocol analysis skills: books/skills/protocol/data-extraction.md with tshark and hexdump methodology
- [ ] T048 [P] [US1] Create pattern detection skills: books/skills/pattern/entropy-analysis.md and sequence-detection.md
- [ ] T049 [P] [US1] Create binary analysis skills: books/skills/binary/boundary-detection.md and type-inference.md
- [ ] T050 [P] [US1] Create grammar validation skills: books/skills/grammar/grammar-validation.md with cross-sample testing methodology
- [ ] T051 [P] [US1] Update books/skills/README.md with taxonomy, usage examples, and LLM integration guidelines

### DataInterpreter Agent Implementation (US1)

- [ ] T052 [P] [US1] Implement DataInterpreter agent class in reshark/agents/interpreters/data_interpreter.py
- [ ] T053 [US1] Add artifact extraction method using ToolsRegistry (tshark for PCAPs, hexdump for binaries)
- [ ] T054 [US1] Add stream parsing method extracting protocol conversations and sessions
- [ ] T055 [US1] Add observation recording method writing structured data to Notebook
- [ ] T056 [US1] Implement DataInterpreter.execute() reading protocol/data-extraction.md skill and orchestrating tool invocations
- [ ] T057 [US1] Add DataInterpreter.get_required_skills() returning ['protocol/data-extraction.md']

### PatternInterpreter Agent Implementation (US1)

- [ ] T058 [P] [US1] Implement PatternInterpreter agent class in reshark/agents/interpreters/pattern_interpreter.py
- [ ] T059 [US1] Add entropy analysis method using ToolsRegistry entropy calculator and reading pattern/entropy-analysis.md skill
- [ ] T060 [US1] Add sequence detection method identifying repeating patterns and reading pattern/sequence-detection.md skill
- [ ] T061 [US1] Add boundary detection method finding message delimiters and reading binary/boundary-detection.md skill
- [ ] T062 [US1] Add pattern hypothesis generation method creating testable claims with confidence scores
- [ ] T063 [US1] Implement PatternInterpreter.execute() reading Notebook observations and generating pattern hypotheses
- [ ] T064 [US1] Add PatternInterpreter.get_required_skills() returning ['pattern/entropy-analysis.md', 'pattern/sequence-detection.md', 'binary/boundary-detection.md']

### GrammarValidator Agent Implementation (US1)

- [ ] T065 [P] [US1] Implement GrammarValidator agent class in reshark/agents/validation/grammar_validator.py
- [ ] T066 [US1] Add constitutional 3-sample enforcement (hardcoded check) raising ConstitutionalViolation if <3 samples
- [ ] T067 [US1] Add pytest test generation method creating test code from pattern hypotheses
- [ ] T068 [US1] Add cross-sample validation method executing tests against all provided samples (PCAP/binary files)
- [ ] T069 [US1] Add validation results recording method writing pass/fail evidence to Notebook
- [ ] T070 [US1] Add grammar promotion method writing validated grammar to Rulebook with validation metadata
- [ ] T071 [US1] Implement GrammarValidator.execute() reading grammar/grammar-validation.md skill and orchestrating validation
- [ ] T072 [US1] Add GrammarValidator.get_required_skills() returning ['grammar/grammar-validation.md']

### SessionManager and TaskOrchestrator (US1)

- [ ] T073 [P] [US1] Implement SessionManager agent in reshark/agents/orchestration/session_manager.py
- [ ] T074 [US1] Add session creation method generating UUID-based Notebook directories
- [ ] T075 [US1] Add session lifecycle management (start, pause, resume, archive)
- [ ] T076 [US1] Add session cleanup method removing old Notebook sessions
- [ ] T077 [P] [US1] Implement TaskOrchestrator agent in reshark/agents/orchestration/task_orchestrator.py
- [ ] T078 [US1] Add agent workflow coordination (DataInterpreter → PatternInterpreter → GrammarValidator sequence)
- [ ] T079 [US1] Add HITL checkpoint integration allowing human review between agent steps
- [ ] T080 [US1] Add error handling and agent failure recovery

### CLI Integration for HITL Workflow (US1)

- [ ] T081 [US1] Create CLI module at reshark/cli.py with Click framework
- [ ] T082 [US1] Add 'analyze' command invoking TaskOrchestrator with artifact path and session ID
- [ ] T083 [US1] Add 'session' commands for listing, viewing, and cleaning sessions
- [ ] T084 [US1] Add verbose logging option for debugging agent execution

### Test Data for US1

- [ ] T085 [P] [US1] Create synthetic protocol generator in tests/fixtures/protocol_generator.py
- [ ] T086 [P] [US1] Generate 3 synthetic protocol samples in tests/fixtures/samples/
- [ ] T087 [P] [US1] Create binary test artifacts (various formats) in tests/fixtures/samples/

**Checkpoint**: User Story 1 complete - skills-guided HITL workflow functional with Cline + Ollama

---

## Phase 4: User Story 2 - Constitutional Compliance & Books Boundaries (Priority: P1)

**Goal**: Ensure constitutional Books boundaries are enforced, preventing unauthorized state changes and maintaining 3-sample validation requirement

**Independent Test**: Attempt to write to Rulebook from non-GrammarValidator agents and verify violations are blocked, attempt promotion without 3 samples and verify rejection

### Books Boundary Validation (US2)

- [ ] T088 [US2] Add write_rulebook() permission check in BaseAgent class raising BooksBoundaryViolation for non-GrammarValidator agents
- [ ] T089 [US2] Add validation gate in GrammarValidator.promote() checking pattern validation status before allowing promotion
- [ ] T090 [US2] Add 3-sample requirement check in GrammarValidator.promote() verifying constitutional minimum (hardcoded)
- [ ] T091 [US2] Add agent attribution to all Notebook writes ensuring traceability
- [ ] T092 [US2] Add ConsistencyChecker agent in reshark/agents/validation/consistency_checker.py verifying evidence chain completeness

### Integration Tests for Constitutional Compliance (US2)

- [ ] T093 [P] [US2] Create constitutional compliance test suite in tests/integration/test_constitutional_compliance.py
- [ ] T094 [US2] Add test verifying DataInterpreter cannot write to Rulebook (expects BooksBoundaryViolation)
- [ ] T095 [US2] Add test verifying PatternInterpreter cannot write to Rulebook (expects BooksBoundaryViolation)
- [ ] T096 [US2] Add test verifying SessionManager cannot write to Rulebook (expects BooksBoundaryViolation)
- [ ] T097 [US2] Add test verifying unvalidated patterns cannot be promoted (expects ValidationError)
- [ ] T098 [US2] Add test verifying promotion with <3 samples fails (expects ConstitutionalViolation)
- [ ] T099 [US2] Add test verifying all Notebook entries have timestamp and agent attribution
- [ ] T100 [US2] Add test verifying evidence trails are complete for full HITL workflow
- [ ] T101 [US2] Add test verifying Skills library files are immutable during agent execution

**Checkpoint**: Books boundary enforcement verified - constitutional compliance guaranteed through automated tests

---

## Phase 5: User Story 3 - Skills-Guided Analysis Patterns (Priority: P2)

**Goal**: Expand Skills library with methodology documentation guiding LLM behavior for consistent analysis

**Independent Test**: Run agents with different skill files, verify methodology is followed, test skills-guided prompts with Ollama

### Skills Library Expansion (US3)

- [ ] T102 [P] [US3] Create advanced protocol skills: books/skills/protocol/session-reconstruction.md and state-machine-detection.md
- [ ] T103 [P] [US3] Create advanced pattern skills: books/skills/pattern/correlation-analysis.md and anomaly-detection.md
- [ ] T104 [P] [US3] Create advanced binary skills: books/skills/binary/structure-inference.md and endianness-detection.md
- [ ] T105 [P] [US3] Create advanced grammar skills: books/skills/grammar/grammar-generation.md and kaitai-struct-writing.md
- [ ] T106 [US3] Add skill versioning mechanism tracking methodology changes over time
- [ ] T107 [US3] Add skill validation tests ensuring markdown files are LLM-readable

### Skills Integration with Ollama (US3)

- [ ] T108 [US3] Implement skills context injection in BaseAgent loading markdown content before LLM calls
- [ ] T109 [US3] Add skills-guided prompt templates in reshark/utils/prompts.py
- [ ] T110 [US3] Add Ollama client wrapper in reshark/llm/ollama_client.py with error handling and retry logic
- [ ] T111 [US3] Add skills library tests verifying methodology guidance improves analysis quality

### Cookbook Workflow Documentation (US3)

- [ ] T112 [P] [US3] Document HITL workflow in books/cookbook/methods/hitl-protocol-analysis.md
- [ ] T113 [P] [US3] Document multi-sample validation workflow in books/cookbook/methods/cross-sample-validation.md
- [ ] T114 [P] [US3] Document grammar promotion workflow in books/cookbook/methods/grammar-promotion.md
- [ ] T115 [P] [US3] Update books/cookbook/README.md with workflow index and usage guidelines

**Checkpoint**: Skills library expanded - LLM-guided analysis patterns established with 12+ methodology files

---

## Phase 6: User Story 4 - Evidence-Based Validation & Traceability (Priority: P2)

**Goal**: Ensure all pattern hypotheses are validated against 3+ samples with complete evidence trails, building trust in generated grammars

**Independent Test**: Provide patterns and samples, run validation, verify evidence includes sample paths, test code, and results for each

### Enhanced Validation Evidence (US4)

- [ ] T116 [US4] Add evidence collection method to GrammarValidator recording sample file paths for each validation run
- [ ] T117 [US4] Add test code preservation in GrammarValidator results storing generated pytest code in Notebook
- [ ] T118 [US4] Add detailed pass/fail recording in GrammarValidator capturing assertion failures and exceptions
- [ ] T119 [US4] Add validation summary method generating human-readable validation reports

### Cross-Sample Consistency Metrics (US4)

- [ ] T120 [US4] Add ConsistencyChecker methods calculating cross-sample pattern match rates
- [ ] T121 [US4] Add consistency scoring method measuring how well patterns generalize across samples
- [ ] T122 [US4] Add outlier detection method identifying samples that don't match the pattern
- [ ] T123 [US4] Add consistency report generation creating consistency_report.md in Notebook

### Evidence Trail Completeness (US4)

- [ ] T124 [US4] Add evidence verification method in GrammarValidator checking all validation results have complete evidence before promotion
- [ ] T125 [US4] Add evidence export method in GrammarValidator generating evidence.json alongside grammars in Rulebook
- [ ] T126 [US4] Enhance logging in reshark/utils/logging.py to capture detailed agent execution traces with skill usage
- [ ] T127 [US4] Add evidence audit method listing all evidence for a given grammar in Rulebook

### Validation Reporting (US4)

- [ ] T128 [US4] Create validation report generator in reshark/utils/reports.py
- [ ] T129 [US4] Add report generation to GrammarValidator.execute() creating validation_report.md in Notebook
- [ ] T130 [US4] Add grammar confidence scoring based on validation success rates across samples
- [ ] T131 [US4] Add validation history tracking showing all validation attempts for patterns

**Checkpoint**: Evidence-based validation complete - all grammars have comprehensive validation evidence and traceability

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Improvements affecting multiple user stories, documentation, and final validation

### Documentation

- [ ] T132 [P] Update README.md with v2.0.0 architecture overview, HITL workflows, and quickstart example
- [ ] T133 [P] Create docs/architecture.md explaining agent model, Books system, and skills-guided workflows
- [ ] T134 [P] Create docs/setup.md with Dev Container setup, Ollama configuration, and Cline integration
- [ ] T135 [P] Update project root QUICKSTART.md (already created) with additional troubleshooting and examples

### Error Handling and Edge Cases

- [ ] T136 Add comprehensive error handling for corrupted artifacts (PCAP/binary) with clear user messages
- [ ] T137 Add error handling for large artifacts (>1GB) with memory-efficient streaming
- [ ] T138 Add error handling for empty or invalid artifacts
- [ ] T139 Add error handling for missing session data with helpful recovery suggestions
- [ ] T140 Add error handling for insufficient disk space in books/ directories
- [ ] T141 Add error handling for Ollama connection failures with retry logic

### Performance and Monitoring

- [ ] T142 Add progress reporting for DataInterpreter when processing large artifacts
- [ ] T143 Add performance logging capturing analysis duration for each agent execution
- [ ] T144 Add memory usage monitoring warning when approaching limits
- [ ] T145 Optimize artifact processing for 1GB+ files using chunk processing
- [ ] T146 Add agent execution profiling for performance optimization

### Integration Testing

- [ ] T147 Create end-to-end HITL workflow test in tests/integration/test_complete_workflow.py
- [ ] T148 Add test for complete workflow with synthetic protocol verifying grammar generation with 3+ samples
- [ ] T149 Add test for workflow with validation failures verifying rejection behavior
- [ ] T150 Add test for multi-session isolation verifying no cross-contamination in Notebook
- [ ] T151 Add test for large artifact handling (1GB synthetic PCAP)
- [ ] T152 Add test for skills-guided analysis verifying methodology is followed
- [ ] T153 Add test for Ollama integration verifying LLM responses incorporate skills context

### Performance Benchmarks

- [ ] T154 [P] Create performance benchmark suite in tests/performance/test_benchmarks.py
- [ ] T155 Add benchmark for DataInterpreter processing 100MB PCAP (<30s target)
- [ ] T156 Add benchmark for PatternInterpreter analysis (<10s per observation target)
- [ ] T157 Add benchmark for GrammarValidator with 3 samples (<60s target)
- [ ] T158 Add benchmark for complete HITL workflow (<30min target per SC-001)

### Validation and Cleanup

- [ ] T159 Run pytest coverage report verifying >80% coverage (success criterion SC-009)
- [ ] T160 Execute full HITL workflow on real protocol example (success criterion SC-007)
- [ ] T161 Validate all constitutional compliance tests pass (success criterion SC-010)
- [ ] T162 Verify QUICKSTART.md instructions work on clean Dev Container environment
- [ ] T163 Review all edge cases from spec.md are handled with tests
- [ ] T164 Final code review for Books boundary enforcement correctness
- [ ] T165 Verify skills library has 12+ methodology files (success criterion SC-004)

**Checkpoint**: Phase 1 MVP complete - all success criteria met, ready for production use

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - start immediately
- **Foundational (Phase 2)**: Depends on Setup (Phase 1) - BLOCKS all user stories
- **Research & Design (Phase 2.5)**: Can run in parallel with Foundational or after - Produces documentation artifacts
- **User Story 1 (Phase 3)**: Depends on Foundational (Phase 2) - Skills-guided HITL workflow
- **User Story 2 (Phase 4)**: Depends on Foundational (Phase 2) and US1 implementation - Tests Books boundaries
- **User Story 3 (Phase 5)**: Depends on US1 implementation - Expands skills library
- **User Story 4 (Phase 6)**: Depends on US1 GrammarValidator implementation - Enhances validation evidence
- **Polish (Phase 7)**: Depends on all user stories - Final improvements

### Critical Path (Minimum MVP)

For fastest MVP delivery, focus on:
1. Setup (T001-T009) - Project structure + Books + Dev Container + Skills foundation
2. Foundational (T010-T031) - Books system + BaseAgent + ToolsRegistry
3. Research & Design (T032-T046) - Optional but recommended for architectural clarity
4. User Story 1 (T047-T087) - Skills library + Agents (DataInterpreter, PatternInterpreter, GrammarValidator) + HITL workflow
5. User Story 2 (T088-T101) - Constitutional compliance + Books boundary tests
6. Select polish tasks (T132-T135 for docs, T147-T153 for integration tests)

### User Story Dependencies

- **US1 (Skills-Guided Analysis)**: No dependencies on other stories - can complete independently after Foundational
- **US2 (Books Boundaries)**: Requires US1 agents to exist for testing violations
- **US3 (Skills Expansion)**: Requires US1 agents for skills integration testing
- **US4 (Evidence Validation)**: Requires US1 GrammarValidator implementation for evidence enhancement

### Within Each User Story

- US1: Skills → DataInterpreter → PatternInterpreter → GrammarValidator → SessionManager/TaskOrchestrator → CLI (sequential agent implementation)
- US1: Test data generation (T085-T087) can be done anytime after Setup
- US2: Integration tests (T093-T101) can run in parallel once US1 agents exist
- US3: Skills files (T102-T105) can be created in parallel
- US4: Evidence enhancements build on existing GrammarValidator

### Parallel Opportunities

**Setup Phase**:
- T003, T004, T005, T006, T007, T008, T009 can all run in parallel after T001-T002

**Foundational Phase**:
- T011, T012, T013, T014 (Books classes) in parallel
- T016, T017, T018 (schemas) in parallel
- T024, T025, T026 (utilities) in parallel
- T029, T030, T031 (tool wrappers) in parallel

**Research & Design Phase**:
- T032, T033, T034, T035, T036, T037, T038 (all research topics) in parallel
- T039 can follow research completion
- T040, T041, T042, T043, T044, T045, T046 (all design docs) in parallel

**User Story 1**:
- T047, T048, T049, T050, T051 (skills library files) all in parallel
- T052, T057 (DataInterpreter + tool requirements) in parallel
- T058, T064 (PatternInterpreter + tool requirements) in parallel
- T065, T072 (GrammarValidator + requirements) in parallel
- T073, T077 (SessionManager + TaskOrchestrator) in parallel
- T085, T086, T087 (test data) all in parallel

**User Story 2**:
- T094, T095, T096 (boundary violation tests) in parallel
- T097, T098, T099, T100, T101 (compliance tests) in parallel

**User Story 3**:
- T102, T103, T104, T105 (skills files) all in parallel
- T112, T113, T114, T115 (cookbook docs) all in parallel

**User Story 4**:
- T120, T121, T122, T123 (consistency metrics) can be developed in parallel
- T128, T129, T130, T131 (reporting features) can be developed in parallel

**Polish Phase**:
- T132, T133, T134, T135 (documentation) all in parallel
- T136, T137, T138, T139, T140, T141 (error handling) all in parallel
- T147, T148, T149, T150, T151, T152, T153 (integration tests) all in parallel
- T154, T155, T156, T157, T158 (performance benchmarks) all in parallel

---

## Parallel Example: User Story 1 DataInterpreter Implementation

```bash
# Launch in parallel:
Task T047: "Create protocol/data-extraction.md skill"
Task T052: "Implement DataInterpreter agent class"
Task T029: "Implement tshark_wrapper.py"

# Then sequentially implement DataInterpreter methods:
Task T053: "Add artifact extraction method"
Task T054: "Add stream parsing method"
Task T055: "Add observation recording method"
Task T056: "Implement DataInterpreter.execute() with skill loading"
```

---

## Implementation Strategy

### MVP First (US1 + US2 Only)

**Week 1-2**: Foundation
1. Complete Phase 1: Setup (T001-T009)
2. Complete Phase 2: Foundational (T010-T031)
3. Complete Phase 2.5: Research & Design (T032-T046) - Can overlap with Foundational
4. **Checkpoint**: Foundation ready

**Week 3-5**: Skills-Guided HITL Workflow
1. Complete Phase 3: User Story 1 (T047-T087)
   - Week 3: Skills library + DataInterpreter + PatternInterpreter
   - Week 4: GrammarValidator + constitutional 3-sample enforcement
   - Week 5: SessionManager + TaskOrchestrator + CLI + Test data
2. **Checkpoint**: End-to-end HITL workflow functional with Cline + Ollama

**Week 6**: Constitutional Compliance
1. Complete Phase 4: User Story 2 (T088-T101)
2. **Checkpoint**: Books boundaries enforced and tested

**Week 7**: Polish
1. Complete documentation (T132-T135)
2. Complete integration tests (T147-T153)
3. **VALIDATE MVP**: Run full HITL workflow, verify success criteria

### Full Phase 1 Delivery

Continue after MVP with:

**Week 8**: Skills Expansion
1. Complete Phase 5: User Story 3 (T102-T115)
2. **Checkpoint**: 12+ skills files, Ollama integration tested, Cookbook workflows documented

**Week 9**: Evidence Enhancement
1. Complete Phase 6: User Story 4 (T116-T131)
2. **Checkpoint**: Validation evidence complete with cross-sample consistency metrics

**Week 10**: Final Polish
1. Complete remaining polish tasks (T136-T165)
2. **FINAL VALIDATION**: All success criteria verified, performance benchmarks passed

### Incremental Delivery

Each phase delivers value:
1. **After Foundation**: Core abstractions ready (Books, BaseAgent, ToolsRegistry)
2. **After US1**: Skills-guided HITL protocol analysis works end-to-end (MVP!)
3. **After US2**: Constitutional guarantees verified (3-sample minimum hardcoded)
4. **After US3**: Skills library expanded with 12+ methodology files
5. **After US4**: Evidence trails complete with cross-sample metrics
6. **After Polish**: Production ready with >80% test coverage

### Parallel Team Strategy

With 3 developers:

**Week 1-2 (Together)**: Complete Setup + Foundational + Research & Design

**Week 3-5 (Parallel)**:
- Dev A: Skills library + DataInterpreter (T047-T057)
- Dev B: PatternInterpreter (T058-T064)
- Dev C: GrammarValidator (T065-T072)
- All: SessionManager + TaskOrchestrator + CLI (T073-T084)
- All: Test data generation (T085-T087)

**Week 6 (Parallel)**:
- Dev A: Books boundary enforcement (T088-T092)
- Dev B: Constitutional tests (T093-T101)
- Dev C: Documentation start (T132-T135)

**Week 7-10 (Parallel)**:
- Dev A: Skills expansion (US3)
- Dev B: Evidence enhancement (US4)
- Dev C: Integration tests and polish

---

## Success Criteria Mapping

| Success Criterion | Verified By Tasks | How to Test |
|-------------------|------------------|-------------|
| SC-001: <30min end-to-end | T147-T148, T158, T160 | Run HITL workflow with Cline, time to completion |
| SC-002: 100% constitutional (3-sample) | T088-T101 | Run constitutional compliance test suite |
| SC-003: Agent responsibility isolation | T088, T099 | Verify Books access boundaries, test violations |
| SC-004: Skills-guided consistency | T051, T107, T152, T165 | Verify 12+ skills files, test LLM methodology adherence |
| SC-005: <8K LOC Phase 1 | T159 (coverage + LOC) | Count lines in reshark/ excluding Phase 2 autonomous/ |
| SC-006: Ollama performance (<10s) | T156, T153 | Benchmark agent calls with Ollama, verify <10s |
| SC-007: ToolsRegistry completeness | T027-T031 | Verify tshark, hexdump, strings, file wrappers exist |
| SC-008: Cross-sample validation (3+) | T066, T098, T148 | Test GrammarValidator enforces 3-sample minimum |
| SC-009: Notebook session isolation | T026, T150 | Test UUID-based directories, verify no cross-contamination |
| SC-010: Dev Container HITL workflow | T007, T162 | Test QUICKSTART.md in clean Dev Container |

---

## Notes

- **HITL Focus**: Phase 1 prioritizes human-in-the-loop workflows with Cline + Ollama. Autonomous mode (Phase 2) deferred.
- **Skills-Guided**: LLM reads markdown methodology files before agent execution for consistent analysis patterns.
- **[P] tasks**: Different files, no dependencies within phase - can parallelize
- **[Story] labels**: Maps to user stories US1-US4 from spec.md for traceability
- **File paths**: All paths are absolute from repository root
- **Session isolation**: Each analysis gets unique UUID-based session in books/notebook/sessions/
- **Evidence trails**: All operations logged to Notebook with timestamps and agent attribution
- **Books boundaries**: Only GrammarValidator can write to Rulebook, enforced by BaseAgent class
- **Constitutional gates**: 3-sample minimum hardcoded in GrammarValidator, cannot be overridden
- **Incremental value**: Each user story delivers independent value
- **MVP = US1 + US2**: Skills-guided HITL workflow + constitutional compliance = minimum viable product

---

## Risk Mitigation Notes

From plan.md risk assessment:

**High Risk - Tool wrapper errors (tshark timeouts, malformed output)**:
- Addressed by: T029-T031 (tool wrappers with error handling), T136-T141 (comprehensive error handling), T155 (benchmark tshark performance)

**High Risk - LLM hallucination in pattern detection**:
- Addressed by: T047-T051 (skills-guided prompts), T066 (3-sample constitutional requirement), T116-T123 (evidence-based validation with consistency metrics)

**High Risk - Constitutional compliance enforcement (3-sample minimum)**:
- Addressed by: T066 (hardcoded check in GrammarValidator), T088-T101 (constitutional compliance tests), T098 (test <3 samples rejection)

**Medium Risk - Skills library maintainability (markdown sprawl)**:
- Addressed by: T009 (taxonomic organization), T051 (README with guidelines), T106 (versioning mechanism), T107 (validation tests)

**Medium Risk - Agent coordination complexity**:
- Addressed by: T057-T060 (boundary implementation), T061-T068 (boundary tests)

**Medium Risk - Performance on large PCAPs**:
- Addressed by: T108 (progress reporting), T111 (chunk processing), T116 (1GB test)

**Low Risk - Documentation completeness**:
- Addressed by: T099-T102 (core docs), T101 (cookbook procedures)
**Medium Risk - Agent coordination complexity**:
- Addressed by: T078-T080 (TaskOrchestrator with clear workflow), T016-T018 (well-defined Notebook schemas), T147-T153 (integration tests)

**Medium Risk - Performance on large artifacts (>1GB)**:
- Addressed by: T142 (progress reporting), T145 (chunk processing), T151 (1GB test), T155-T158 (performance benchmarks)

**Low Risk - Dev Container setup complexity**:
- Addressed by: T007 (docker-compose with pre-configured services), T134 (documented setup in docs/setup.md), T162 (QUICKSTART verification)
