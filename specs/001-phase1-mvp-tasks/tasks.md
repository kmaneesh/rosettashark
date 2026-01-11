# Tasks: Phase 1 MVP - Foundation Implementation

**Input**: Design documents from `/specs/001-phase1-mvp-tasks/`  
**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md)  
**Feature Branch**: `001-phase1-mvp-tasks`

**Tests**: Not included per spec - focus is on core implementation with manual validation. Integration tests added for constitutional compliance verification only.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

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
- [ ] T002 Initialize Poetry project with Python 3.11+ and add core dependencies (numpy, scipy, pandas, scapy, pyyaml)
- [ ] T003 [P] Create workspace directory structure: workspace/{notebook,rulebook/grammars,rulebook/tests,cookbook,pcaps}
- [ ] T004 [P] Configure .gitignore to exclude workspace/ directory and __pycache__
- [ ] T005 [P] Setup pytest configuration in pyproject.toml with coverage reporting
- [ ] T006 [P] Create tests/ directory structure: tests/{unit/test_scopes,unit/test_memory,unit/test_tools,integration,fixtures}
- [ ] T007 [P] Create Docker execution environment with Dockerfile and docker-compose.yml in docker/ (addresses FR-024: sandboxed tool execution)
- [ ] T008 [P] Copy constitution to docs/constitution-v2.md for reference

**Checkpoint**: Project structure ready - foundational phase can begin

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story implementation

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

### Memory System Core

- [ ] T009 Create memory system base module at reshark/memory/__init__.py with MemoryBoundaryViolation exception
- [ ] T010 [P] Implement Notebook class in reshark/memory/notebook.py with JSON file storage and session isolation
- [ ] T011 [P] Implement Rulebook class in reshark/memory/rulebook.py with grammar versioning and test suite storage
- [ ] T012 [P] Implement Cookbook class in reshark/memory/cookbook.py with Markdown procedure storage (read-only)

### Schemas and Validation

- [ ] T013 Create schemas module at reshark/schemas/__init__.py
- [ ] T014 [P] Define Notebook JSON schema in reshark/schemas/notebook_schema.py with timestamp, scope, and data fields
- [ ] T015 [P] Define Hypothesis schema in reshark/schemas/hypothesis_schema.py with claim, test method, parameters, confidence
- [ ] T016 [P] Define Grammar schema in reshark/schemas/grammar_schema.py with structure, parsing rules, version

### Base Scope Framework

- [ ] T017 Create scopes module at reshark/scopes/__init__.py
- [ ] T018 Implement abstract Base Scope class in reshark/scopes/base.py with authority_scope property, execute method, and memory access methods
- [ ] T019 Add memory boundary enforcement to Base Scope class preventing non-Archivist writes to Rulebook

### Utilities and Logging

- [ ] T020 Create utils module at reshark/utils/__init__.py
- [ ] T021 [P] Implement evidence trail logging in reshark/utils/logging.py with timestamp and scope attribution
- [ ] T022 [P] Implement input validation utilities in reshark/utils/validation.py for PCAP file checks

### Tool Integration Layer

- [ ] T023 Create tools module at reshark/tools/__init__.py
- [ ] T024 Implement PCAP reader abstraction in reshark/tools/pcap_reader.py using scapy with error handling for corrupted files

**Checkpoint**: Foundation ready - research and design phase can begin

---

## Phase 2.5: Research & Design (Phase 0-1 Outputs)

**Purpose**: Technical research and architectural design before implementation

**⚠️ NOTE**: These tasks generate artifacts required by plan.md Phase 0-1. Can be done in parallel with or after Foundational phase.

### Research Phase (Phase 0)

- [ ] T123 [P] Research PCAP parsing libraries (scapy vs dpkt vs pyshark) and document decision in specs/001-phase1-mvp-tasks/research.md
- [ ] T124 [P] Research statistical analysis methods (Shannon entropy, byte frequency, correlation) and document algorithms in research.md
- [ ] T125 [P] Research test suite generation approaches (string templates vs AST manipulation) and document strategy in research.md
- [ ] T126 [P] Research session isolation mechanisms (UUID vs timestamp directories) and document design in research.md
- [ ] T127 [P] Research grammar format options (Python classes vs YAML vs DSL) and document specification in research.md
- [ ] T128 Document known unknowns resolution in research.md (chunk sizes, entropy thresholds, confidence scoring, synthetic protocol generation)

### Design Phase (Phase 1)

- [ ] T129 [P] Create data-model.md in specs/001-phase1-mvp-tasks/ documenting core entities (Scope, Session, Observation, Hypothesis, ValidationResult, Grammar)
- [ ] T130 [P] Document entity relationships and state transitions in data-model.md (Session→Observation→Hypothesis→ValidationResult→Grammar flows)
- [ ] T131 [P] Create contracts/ directory in specs/001-phase1-mvp-tasks/ with scope-interface.md defining Base Scope class contract
- [ ] T132 [P] Create notebook-schema.json and validation-contract.md in contracts/ directory defining memory schemas and validation requirements
- [ ] T133 Create quickstart.md in specs/001-phase1-mvp-tasks/ with setup instructions, first analysis walkthrough, and result verification steps

**Checkpoint**: Research and design artifacts complete - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Basic Protocol Analysis Workflow (Priority: P1) 🎯 MVP

**Goal**: Enable end-to-end protocol analysis from PCAP to validated grammar, demonstrating core value proposition

**Independent Test**: Provide synthetic binary protocol PCAP, run full workflow (Observer → Theorist → Validator → Archivist), verify grammar in Rulebook

### Observer Scope Implementation (US1)

- [ ] T025 [P] [US1] Implement Observer scope class in reshark/scopes/observer.py with authority_scope='statistical_analysis'
- [ ] T026 [US1] Add entropy calculation method to Observer scope using Shannon entropy from byte frequencies
- [ ] T027 [US1] Add byte frequency analysis method to Observer scope generating distribution maps
- [ ] T028 [US1] Add invariant detection method to Observer scope identifying constant byte positions
- [ ] T029 [US1] Implement Observer.execute() method orchestrating analysis and writing to Notebook
- [ ] T030 [P] [US1] Create statistical analysis utilities in reshark/tools/statistics.py for entropy and frequency calculations

### Theorist Scope Implementation (US1)

- [ ] T031 [P] [US1] Implement Theorist scope class in reshark/scopes/theorist.py with authority_scope='hypothesis_generation'
- [ ] T032 [US1] Add hypothesis generation method reading Observer results from Notebook
- [ ] T033 [US1] Add boundary detection logic identifying message delimiters from entropy patterns
- [ ] T034 [US1] Add field structure inference logic proposing fixed vs variable fields
- [ ] T035 [US1] Add confidence scoring method for generated hypotheses
- [ ] T036 [US1] Implement Theorist.execute() method generating testable hypotheses and writing to Notebook

### Validator Scope Implementation (US1)

- [ ] T037 [P] [US1] Implement Validator scope class in reshark/scopes/validator.py with authority_scope='cross_validation'
- [ ] T038 [US1] Add minimum 3 PCAP enforcement check in Validator.execute() raising error if insufficient PCAPs
- [ ] T039 [US1] Add test suite generation method creating pytest code from hypotheses
- [ ] T040 [P] [US1] Create test generator utilities in reshark/tools/test_generator.py for pytest code generation
- [ ] T041 [US1] Add cross-PCAP validation method executing tests against all provided PCAPs
- [ ] T042 [US1] Add negative testing method with corrupted PCAP detection
- [ ] T043 [US1] Implement results recording method writing pass/fail outcomes to Notebook with evidence trails

### Archivist Scope Implementation (US1)

- [ ] T044 [P] [US1] Implement Archivist scope class in reshark/scopes/archivist.py with authority_scope='memory_management'
- [ ] T045 [US1] Add grammar generation method converting validated hypotheses to protocol specifications
- [ ] T046 [US1] Add Rulebook promotion method with validation gate checking 3 PCAP requirement
- [ ] T047 [US1] Add test suite export method writing generated pytest files to workspace/rulebook/tests/
- [ ] T048 [US1] Add session archival method organizing Notebook entries for completed analyses
- [ ] T049 [US1] Implement Archivist.execute() method handling promotion operations

### Orchestration for US1

- [ ] T050 Create orchestration module at reshark/orchestration/__init__.py
- [ ] T051 [US1] Implement analyze_protocol.py script orchestrating Observer → Theorist → Validator → Archivist workflow
- [ ] T052 [US1] Add human review checkpoints in orchestration script before Rulebook promotion
- [ ] T053 [US1] Add error handling and progress reporting to orchestration script

### Test Data for US1

- [ ] T054 [P] [US1] Create synthetic binary protocol generator script for testing
- [ ] T055 [P] [US1] Generate synthetic protocol PCAP in tests/fixtures/synthetic_protocol.pcap
- [ ] T056 [P] [US1] Generate 3 validation PCAPs in tests/fixtures/validation_pcaps/ directory

**Checkpoint**: User Story 1 complete - full workflow from PCAP to grammar is functional and manually testable

---

## Phase 4: User Story 2 - Memory Boundary Enforcement (Priority: P1)

**Goal**: Ensure constitutional memory boundaries are enforced, preventing unauthorized state changes and maintaining system integrity

**Independent Test**: Attempt to write to Rulebook from non-Archivist scopes and verify violations are blocked, attempt promotion without validation and verify rejection

### Memory Boundary Validation (US2)

- [ ] T057 [US2] Add write_rulebook() permission check in Base Scope class raising MemoryBoundaryViolation for non-Archivist scopes
- [ ] T058 [US2] Add validation gate in Archivist.promote() checking hypothesis validation status before allowing promotion
- [ ] T059 [US2] Add validation requirement check in Archivist.promote() verifying 3 PCAP minimum from Validator results
- [ ] T060 [US2] Add scope attribution to all Notebook writes ensuring traceability

### Integration Tests for Constitutional Compliance (US2)

- [ ] T061 [P] [US2] Create constitutional compliance test suite in tests/integration/test_constitution.py
- [ ] T062 [US2] Add test verifying Observer cannot write to Rulebook (expects MemoryBoundaryViolation)
- [ ] T063 [US2] Add test verifying Theorist cannot write to Rulebook (expects MemoryBoundaryViolation)
- [ ] T064 [US2] Add test verifying Validator cannot write to Rulebook (expects MemoryBoundaryViolation)
- [ ] T065 [US2] Add test verifying unvalidated hypotheses cannot be promoted (expects ValidationError)
- [ ] T066 [US2] Add test verifying promotion with <3 PCAPs fails (expects ConstitutionalViolation)
- [ ] T067 [US2] Add test verifying all Notebook entries have timestamp and scope attribution
- [ ] T068 [US2] Add test verifying evidence trails are complete for full workflow

**Checkpoint**: Memory boundary enforcement verified - constitutional compliance guaranteed through automated tests

---

## Phase 5: User Story 3 - Manual Workflow Orchestration (Priority: P2)

**Goal**: Provide manual control over each analysis step, enabling human review and informed decision-making

**Independent Test**: Run each scope independently via command-line, verify outputs at each step, test session isolation with multiple protocols

### Command-Line Interfaces (US3)

- [ ] T069 [P] [US3] Add CLI entry point to Observer scope in reshark/scopes/observer.py accepting --pcap and --session arguments
- [ ] T070 [P] [US3] Add CLI entry point to Theorist scope in reshark/scopes/theorist.py accepting --session argument
- [ ] T071 [P] [US3] Add CLI entry point to Validator scope in reshark/scopes/validator.py accepting --session and --pcaps arguments
- [ ] T072 [P] [US3] Add CLI entry point to Archivist scope in reshark/scopes/archivist.py accepting --session, --operation, and --hypothesis-id arguments

### Session Management (US3)

- [ ] T073 [US3] Create session manager module at reshark/utils/session.py
- [ ] T074 [US3] Add session creation method generating unique session IDs (UUID-based)
- [ ] T075 [US3] Add session isolation enforcement ensuring separate workspace/notebook/{session_id}/ directories
- [ ] T076 [US3] Add session listing method showing all active and archived sessions
- [ ] T077 [US3] Add session cleanup method for removing old session data

### Workflow Scripts (US3)

- [ ] T078 [US3] Enhance analyze_protocol.py to support step-by-step execution with pause between stages
- [ ] T079 [US3] Add hypothesis review mode to analyze_protocol.py showing Theorist results before validation
- [ ] T080 [US3] Add validation result review in analyze_protocol.py allowing refinement before promotion
- [ ] T081 [US3] Create validate_grammar.py script in reshark/orchestration/ for re-running validation on existing grammars
- [ ] T082 [US3] Create promote_to_rulebook.py script in reshark/orchestration/ for explicit promotion operations

**Checkpoint**: Manual workflow orchestration complete - users have full control over each analysis step with session isolation

---

## Phase 6: User Story 4 - Evidence-Based Validation (Priority: P2)

**Goal**: Ensure all hypotheses are validated against multiple PCAPs with complete evidence trails, building trust in generated grammars

**Independent Test**: Provide hypotheses and PCAPs, run validation, verify evidence includes PCAP paths, test code, and results for each

### Enhanced Validation Evidence (US4)

- [ ] T083 [US4] Add evidence collection method to Validator scope recording PCAP file paths for each validation run
- [ ] T084 [US4] Add test code preservation in Validator results storing generated pytest code in Notebook
- [ ] T085 [US4] Add detailed pass/fail recording in Validator capturing assertion failures and exceptions
- [ ] T086 [US4] Add validation summary method generating human-readable validation reports

### Negative Testing Implementation (US4)

- [ ] T087 [US4] Add corrupted PCAP detection method in reshark/tools/pcap_reader.py
- [ ] T088 [US4] Add negative test generation in Validator creating tests that should reject invalid data
- [ ] T089 [US4] Add negative test execution in Validator verifying hypotheses correctly reject corrupted PCAPs
- [ ] T090 [US4] Add negative test evidence recording in Notebook

### Evidence Trail Completeness (US4)

- [ ] T091 [US4] Add evidence verification method in Archivist checking all validation results have complete evidence before promotion
- [ ] T092 [US4] Add evidence export method in Archivist generating evidence.json alongside grammars in Rulebook
- [ ] T093 [US4] Enhance logging in reshark/utils/logging.py to capture detailed validation execution traces
- [ ] T094 [US4] Add evidence audit method listing all evidence for a given grammar in Rulebook

### Validation Reporting (US4)

- [ ] T095 [US4] Create validation report generator in reshark/tools/report_generator.py
- [ ] T096 [US4] Add report generation to Validator.execute() creating validation_report.md in Notebook
- [ ] T097 [US4] Add grammar confidence scoring based on validation success rates across PCAPs
- [ ] T098 [US4] Add validation history tracking showing all validation attempts for hypotheses

**Checkpoint**: Evidence-based validation complete - all grammars have comprehensive validation evidence and traceability

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Improvements affecting multiple user stories, documentation, and final validation

### Documentation

- [ ] T099 [P] Create getting-started.md in docs/ with setup instructions and first analysis walkthrough
- [ ] T100 [P] Create architecture.md in docs/ explaining scope model, memory system, and workflow
- [ ] T101 [P] Document analysis procedures in workspace/cookbook/ with Markdown files for entropy analysis, hypothesis generation, validation
- [ ] T102 [P] Update README.md with Phase 1 capabilities and quickstart example

### Error Handling and Edge Cases

- [ ] T103 Add comprehensive error handling for corrupted PCAP files with clear user messages
- [ ] T104 Add error handling for large PCAP files (>10GB) with memory-efficient streaming
- [ ] T105 Add error handling for empty PCAPs or PCAPs with no valid packets
- [ ] T106 Add error handling for missing session data with helpful recovery suggestions
- [ ] T107 Add error handling for insufficient disk space in workspace directory

### Performance and Monitoring

- [ ] T108 Add progress reporting for Observer scope when processing large PCAPs
- [ ] T109 Add performance logging capturing analysis duration for each scope execution
- [ ] T110 Add memory usage monitoring warning when approaching limits
- [ ] T111 Optimize PCAP reading for 1GB+ files using chunk processing

### Integration Testing

- [ ] T112 Create end-to-end workflow test in tests/integration/test_workflow.py
- [ ] T113 Add test for complete workflow with synthetic protocol verifying grammar generation
- [ ] T114 Add test for workflow with validation failures verifying rejection behavior
- [ ] T115 Add test for multi-session isolation verifying no cross-contamination
- [ ] T116 Add test for large PCAP handling (1GB synthetic PCAP)

### Validation and Cleanup

- [ ] T117 Run pytest coverage report verifying >80% coverage (success criterion SC-009)
- [ ] T118 Execute full workflow on real protocol example (success criterion SC-007)
- [ ] T119 Validate all constitutional compliance tests pass (success criterion SC-010)
- [ ] T120 Verify quickstart.md instructions work on clean environment
- [ ] T121 Review all edge cases from spec are handled with tests
- [ ] T122 Final code review for memory boundary enforcement correctness

**Checkpoint**: Phase 1 MVP complete - all success criteria met, ready for production use

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - start immediately
- **Foundational (Phase 2)**: Depends on Setup (Phase 1) - BLOCKS all user stories
- **Research & Design (Phase 2.5)**: Can run in parallel with Foundational or after - Produces documentation artifacts
- **User Story 1 (Phase 3)**: Depends on Foundational (Phase 2) - Core workflow
- **User Story 2 (Phase 4)**: Depends on Foundational (Phase 2) and US1 implementation - Tests boundaries
- **User Story 3 (Phase 5)**: Depends on US1 implementation - Adds CLI and session management
- **User Story 4 (Phase 6)**: Depends on US1 Validator implementation - Enhances validation
- **Polish (Phase 7)**: Depends on all user stories - Final improvements

### Critical Path (Minimum MVP)

For fastest MVP delivery, focus on:
1. Setup (T001-T008)
2. Foundational (T009-T024)
3. Research & Design (T123-T133) - Optional but recommended for architectural clarity
4. User Story 1 (T025-T056) - Core protocol analysis
5. User Story 2 (T057-T068) - Constitutional compliance
6. Select polish tasks (T099-T102 for docs, T112-T116 for integration tests)

### User Story Dependencies

- **US1 (Protocol Analysis)**: No dependencies on other stories - can complete independently after Foundational
- **US2 (Memory Boundaries)**: Requires US1 scopes to exist for testing violations
- **US3 (Manual Orchestration)**: Requires US1 scopes for CLI implementation
- **US4 (Evidence Validation)**: Requires US1 Validator implementation for evidence enhancement

### Within Each User Story

- US1: Observer → Theorist → Validator → Archivist (sequential scope implementation)
- US1: Test data generation (T054-T056) can be done anytime after Setup
- US2: Integration tests (T061-T068) can run in parallel once US1 scopes exist
- US3: CLI entry points (T069-T072) can run in parallel
- US4: Evidence enhancements build on existing Validator

### Parallel Opportunities

**Setup Phase**:
- T003, T004, T005, T006, T007, T008 can all run in parallel after T001-T002

**Foundational Phase**:
- T010, T011, T012 (memory classes) in parallel
- T014, T015, T016 (schemas) in parallel
- T021, T022 (utilities) in parallel

**Research & Design Phase**:
- T123, T124, T125, T126, T127 (all research topics) in parallel
- T128 can follow research completion
- T129, T130, T131, T132, T133 (all design docs) in parallel

**User Story 1**:
- T025, T030 (Observer + statistics utility) in parallel
- T031, T040 (Theorist + test generator utility) in parallel
- T037, T040 (Validator + test generator utility) in parallel
- T044 (Archivist) in parallel with any above
- T054, T055, T056 (test data) all in parallel

**User Story 2**:
- T062, T063, T064 (boundary violation tests) in parallel
- T065, T066, T067, T068 (compliance tests) in parallel

**User Story 3**:
- T069, T070, T071, T072 (CLI entry points) in parallel

**User Story 4**:
- T095, T096, T097, T098 (reporting features) can be developed in parallel

**Polish Phase**:
- T099, T100, T101, T102 (documentation) all in parallel
- T103, T104, T105, T106, T107 (error handling) all in parallel
- T112, T113, T114, T115, T116 (integration tests) all in parallel

---

## Parallel Example: User Story 1 Observer Implementation

```bash
# Launch in parallel:
Task T025: "Implement Observer scope class in reshark/scopes/observer.py"
Task T030: "Create statistical analysis utilities in reshark/tools/statistics.py"

# Then sequentially implement Observer methods:
Task T026: "Add entropy calculation method"
Task T027: "Add byte frequency analysis method"
Task T028: "Add invariant detection method"
Task T029: "Implement Observer.execute() method"
```

---

## Implementation Strategy

### MVP First (US1 + US2 Only)

**Week 1-2**: Foundation
1. Complete Phase 1: Setup (T001-T008)
2. Complete Phase 2: Foundational (T009-T024)
3. Complete Phase 2.5: Research & Design (T123-T133) - Can overlap with Foundational
4. **Checkpoint**: Foundation ready

**Week 3-4**: Core Workflow
1. Complete Phase 3: User Story 1 (T025-T056)
2. **Checkpoint**: End-to-end workflow functional

**Week 5**: Constitutional Compliance
1. Complete Phase 4: User Story 2 (T057-T068)
2. **Checkpoint**: Memory boundaries enforced and tested

**Week 6**: Polish
1. Complete documentation (T099-T102)
2. Complete integration tests (T112-T116)
3. **VALIDATE MVP**: Run full workflow, verify success criteria

### Full Phase 1 Delivery

Continue after MVP with:

**Week 7**: Manual Control
1. Complete Phase 5: User Story 3 (T069-T082)
2. **Checkpoint**: CLI and session management working

**Week 8**: Evidence Enhancement
1. Complete Phase 6: User Story 4 (T083-T098)
2. **Checkpoint**: Validation evidence complete

**Week 9**: Final Polish
1. Complete remaining polish tasks (T103-T122)
2. **FINAL VALIDATION**: All success criteria verified

### Incremental Delivery

Each phase delivers value:
1. **After Foundation**: Core abstractions ready
2. **After US1**: Protocol analysis works end-to-end (MVP!)
3. **After US2**: Constitutional guarantees verified
4. **After US3**: Manual workflow control available
5. **After US4**: Evidence trails complete
6. **After Polish**: Production ready

### Parallel Team Strategy

With 3 developers:

**Week 1-2 (Together)**: Complete Setup + Foundational + Research & Design

**Week 3-4 (Parallel)**:
- Dev A: Observer + Theorist (T025-T036)
- Dev B: Validator (T037-T043)
- Dev C: Archivist + Orchestration (T044-T053)
- All: Test data generation (T054-T056)

**Week 5 (Parallel)**:
- Dev A: Memory boundary enforcement (T057-T060)
- Dev B: Constitutional tests (T061-T068)
- Dev C: Documentation start (T099-T102)

**Week 6-8 (Parallel)**:
- Dev A: CLI implementation (US3)
- Dev B: Evidence enhancement (US4)
- Dev C: Integration tests and polish

---

## Success Criteria Mapping

| Success Criterion | Verified By Tasks | How to Test |
|-------------------|------------------|-------------|
| SC-001: 30-min end-to-end | T112, T118 | Run analyze_protocol.py with synthetic PCAP, time to completion |
| SC-002: 100% boundary enforcement | T057-T068 | Run constitutional compliance test suite |
| SC-003: Traceable operations | T060, T067 | Inspect Notebook entries for timestamp and scope fields |
| SC-004: Validation rejects failures | T041, T113, T114 | Run validation with failing hypotheses, verify rejection |
| SC-005: Grammars include tests | T047, T113 | Check workspace/rulebook/tests/ for generated pytest files |
| SC-006: 1GB PCAP handling | T116 | Process 1GB synthetic PCAP, verify completion |
| SC-007: Real protocol example | T118 | Complete analysis of real-world protocol |
| SC-008: Cookbook documentation | T101 | Verify workspace/cookbook/ contains procedure documentation |
| SC-009: 80% test coverage | T117 | Run pytest-cov, verify coverage report |
| SC-010: Constitutional compliance | T061-T068, T119 | Run tests/integration/test_constitution.py |

---

## Notes

- **No TDD**: Tests are for constitutional compliance only, not TDD workflow. Implementation comes first.
- **[P] tasks**: Different files, no dependencies within phase - can parallelize
- **[Story] labels**: Maps to user stories US1-US4 from spec.md for traceability
- **File paths**: All paths are absolute from repository root
- **Session isolation**: Each protocol analysis gets unique session ID
- **Evidence trails**: All operations logged to Notebook with timestamps
- **Manual orchestration**: Phase 1 focuses on scripts, not autonomous agents
- **Constitutional gates**: Memory boundaries and validation requirements enforced by design
- **Incremental value**: Each user story delivers independent value
- **MVP = US1 + US2**: Core workflow + constitutional compliance = minimum viable product

---

## Risk Mitigation Notes

From plan.md risk assessment:

**High Risk - PCAP parsing edge cases**:
- Addressed by: T013 (PCAP validation), T103-T105 (error handling), T116 (large file test)

**High Risk - Validation framework correctness**:
- Addressed by: T061-T068 (constitutional tests), T113-T114 (validation failure tests)

**Medium Risk - Memory boundary enforcement**:
- Addressed by: T057-T060 (boundary implementation), T061-T068 (boundary tests)

**Medium Risk - Performance on large PCAPs**:
- Addressed by: T108 (progress reporting), T111 (chunk processing), T116 (1GB test)

**Low Risk - Documentation completeness**:
- Addressed by: T099-T102 (core docs), T101 (cookbook procedures)
