# Implementation Plan: Phase 1 MVP - Foundation Implementation

**Branch**: `001-phase1-mvp-tasks` | **Date**: 2026-01-11 | **Spec**: [spec.md](spec.md)  
**Input**: Feature specification from `/specs/001-phase1-mvp-tasks/spec.md`

## Summary

Implement the foundational reShark system with manual orchestration, establishing core scopes (Observer, Theorist, Validator, Archivist), memory architecture (Notebook, Rulebook, Cookbook), and end-to-end protocol analysis workflow. This MVP proves the constitutional epistemic framework through evidence-based protocol reconstruction from PCAP files, with strict memory boundary enforcement and validation requirements.

## Technical Context

**Language/Version**: Python 3.11+  
**Primary Dependencies**: numpy, scipy, pandas (statistical analysis), scapy (PCAP manipulation), pyyaml (configuration)  
**Storage**: File system (JSON for Notebook, Python/YAML for Rulebook, Markdown for Cookbook)  
**Testing**: pytest with coverage reporting  
**Target Platform**: Cross-platform via containerization (Docker/Podman)  
**Project Type**: Single CLI/library project with modular scope architecture  
**Performance Goals**: Process 1GB PCAP in <10 minutes, handle 10K messages in memory  
**Constraints**: Strict memory boundaries (constitutional), minimum 3 PCAPs for validation, evidence traceability required  
**Scale/Scope**: Single protocol analysis per session, manual workflow orchestration, ~5K LOC for Phase 1

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### ✅ Epistemic Law Compliance
- **Requirement**: All protocol claims must be evidence-based, testable, and reproducible
- **Implementation**: Notebook records all observations with timestamps; validation requires 3+ PCAPs; no claims without evidence
- **Status**: PASS - spec enforces evidence trails (FR-010), validation requirements (FR-019)

### ✅ Memory Constitution Compliance
- **Requirement**: Strict boundaries between Notebook (ephemeral), Rulebook (validated), Cookbook (procedural)
- **Implementation**: Base Scope class enforces read/write permissions; only Archivist can promote to Rulebook
- **Status**: PASS - spec mandates memory boundary enforcement (FR-009, User Story 2)

### ✅ Validation and Falsification
- **Requirement**: Minimum 3 PCAPs for cross-validation, negative testing, failure logging
- **Implementation**: Validator scope requires 3 PCAPs (FR-019), generates test suites (FR-020), records outcomes (FR-021)
- **Status**: PASS - spec enforces validation requirements explicitly

### ✅ Scope Authority Boundaries
- **Requirement**: Each scope has declared authority; cannot exceed domain
- **Implementation**: Base Scope class with authority_scope property; operations validate against authority
- **Status**: PASS - spec mandates authority enforcement (FR-005)

### ⚠️ Phased Implementation
- **Requirement**: Phase 1 must prove concepts before Phase 2 automation
- **Implementation**: Manual orchestration only; LangGraph/Patternbook deferred
- **Status**: PASS - explicitly scoped to Phase 1, Phase 2 features marked out of scope

**Overall**: ✅ ALL GATES PASS - Constitution-compliant by design

## Project Structure

### Documentation (this feature)

```text
specs/001-phase1-mvp-tasks/
├── spec.md              # Feature specification (completed)
├── plan.md              # This file (implementation plan)
├── research.md          # Phase 0: Technical decisions and unknowns
├── data-model.md        # Phase 1: Entity definitions and schemas
├── quickstart.md        # Phase 1: Getting started guide
├── contracts/           # Phase 1: API contracts and schemas
│   ├── notebook-schema.json
│   ├── scope-interface.md
│   └── validation-contract.md
└── checklists/
    └── requirements.md  # Specification quality checklist (completed)
```

### Source Code (repository root)

```text
reshark/                         # Main package
├── scopes/                      # Core scope implementations
│   ├── __init__.py
│   ├── base.py                 # Base Scope class with memory boundaries
│   ├── observer.py             # Statistical analysis scope
│   ├── theorist.py             # Hypothesis generation scope
│   ├── validator.py            # Cross-PCAP validation scope
│   └── archivist.py            # Memory management scope
│
├── memory/                      # Memory system abstractions
│   ├── __init__.py
│   ├── notebook.py             # Ephemeral working memory
│   ├── rulebook.py             # Validated grammars
│   └── cookbook.py             # Procedural knowledge
│
├── tools/                       # Tool integration layer
│   ├── __init__.py
│   ├── pcap_reader.py          # PCAP file handling
│   ├── statistics.py           # Statistical analysis utilities
│   └── test_generator.py      # Validation test suite generation
│
├── orchestration/               # Phase 1 manual workflows
│   ├── __init__.py
│   ├── analyze_protocol.py     # Main analysis workflow
│   ├── validate_grammar.py     # Grammar validation workflow
│   └── promote_to_rulebook.py  # Promotion workflow
│
├── schemas/                     # Data schemas and validation
│   ├── __init__.py
│   ├── notebook_schema.py      # Notebook JSON schemas
│   ├── hypothesis_schema.py    # Hypothesis structure
│   └── grammar_schema.py       # Grammar format
│
└── utils/                       # Shared utilities
    ├── __init__.py
    ├── logging.py              # Evidence trail logging
    └── validation.py           # Input validation

tests/                           # Test suite
├── unit/                        # Unit tests per module
│   ├── test_scopes/
│   ├── test_memory/
│   └── test_tools/
├── integration/                 # Integration tests
│   ├── test_workflow.py        # End-to-end workflow
│   └── test_constitution.py    # Constitutional compliance
└── fixtures/                    # Test data
    ├── synthetic_protocol.pcap
    ├── validation_pcaps/
    └── expected_outputs/

workspace/                       # Analysis workspace (gitignored)
├── notebook/                    # Session working memory
├── rulebook/                    # Validated grammars
│   ├── grammars/
│   └── tests/
├── cookbook/                    # Procedures and methods
└── pcaps/                       # Input PCAP storage

docker/                          # Execution environment
├── Dockerfile
├── docker-compose.yml
└── tools.txt

docs/                            # Project documentation
├── constitution-v2.md
├── architecture.md
└── getting-started.md

pyproject.toml                   # Python package configuration
README.md                        # Project overview
.gitignore                       # Ignore workspace/
```

**Structure Decision**: Single Python package with modular scope architecture. Scopes are the primary abstraction, memory system provides storage boundaries, orchestration provides manual workflow scripts. Workspace directory (gitignored) contains all analysis sessions and outputs. Docker provides sandboxed execution environment.

## Complexity Tracking

> **No violations - this section intentionally blank**

All constitutional requirements are met by design. No technical debt or complexity justifications needed.

---

## Phase 0: Research & Technical Decisions

**Goal**: Resolve all NEEDS CLARIFICATION items and establish technical foundations

### Research Topics

1. **PCAP Parsing Strategy**
   - **Question**: Best approach for extracting byte streams from PCAPs?
   - **Options**: scapy.rdpcap() vs dpkt vs pyshark
   - **Decision Criteria**: Memory efficiency for large PCAPs, ease of stream extraction
   - **Output**: Selected library with code examples in research.md

2. **Statistical Analysis Methods**
   - **Question**: Which entropy calculation and invariant detection algorithms?
   - **Options**: Shannon entropy, byte frequency analysis, correlation matrices
   - **Decision Criteria**: Proven effectiveness for protocol analysis, computational efficiency
   - **Output**: Algorithm descriptions and references in research.md

3. **Test Suite Generation**
   - **Question**: How to programmatically generate pytest test code from hypotheses?
   - **Options**: String templates, AST manipulation, pytest plugin
   - **Decision Criteria**: Maintainability, debuggability, pytest compatibility
   - **Output**: Test generation strategy in research.md

4. **Session Isolation**
   - **Question**: How to ensure session isolation in file system?
   - **Options**: UUID-based directories, timestamp-based, user-specified
   - **Decision Criteria**: Collision prevention, traceability, cleanup
   - **Output**: Session management design in research.md

5. **Grammar Format**
   - **Question**: What format for protocol grammars in Rulebook?
   - **Options**: Python classes, YAML specifications, DSL
   - **Decision Criteria**: Executability, readability, version control friendliness
   - **Output**: Grammar format specification in research.md

### Known Unknowns (to be resolved in Phase 0)

- Optimal chunk size for processing large PCAPs
- Entropy threshold values for control byte detection
- Hypothesis confidence scoring methodology
- Test coverage requirements for grammar promotion
- Synthetic protocol generator approach for testing

### Dependencies

- PCAP analysis libraries available in execution environment
- Statistical libraries (numpy/scipy) version compatibility
- pytest plugin ecosystem for test generation
- File system permissions for workspace isolation

**Deliverable**: `research.md` with all technical decisions documented and unknowns resolved

---

## Phase 1: Design & Contracts

**Goal**: Define data models, API contracts, and system architecture

### Data Model Design

Create `data-model.md` with:

1. **Core Entities**
   - Scope (base class, authority model, memory access)
   - Session (ID, workspace paths, metadata)
   - Observation (entropy map, frequency distribution, invariants)
   - Hypothesis (claim, test method, parameters, confidence)
   - ValidationResult (pass/fail, evidence, test code)
   - Grammar (structure, parsing rules, version)

2. **Relationships**
   - Session 1:N Observations (one session generates many observations)
   - Observation 1:N Hypotheses (observations inform hypotheses)
   - Hypothesis 1:N ValidationResults (tested against multiple PCAPs)
   - ValidationResult 1:1 Grammar (validated hypothesis produces grammar)

3. **State Transitions**
   - Session: created → active → archived
   - Hypothesis: proposed → validated/rejected → promoted/discarded
   - Grammar: drafted → tested → promoted

### API Contracts

Create `contracts/` with:

1. **scope-interface.md**: Base Scope class contract
   ```python
   class Scope(ABC):
       @abstractmethod
       def execute(self, inputs: Dict) -> Dict:
           """Main execution method"""
       
       @abstractmethod
       def authority_scope(self) -> str:
           """Declared authority domain"""
       
       def read_notebook(self, key: str) -> Optional[Any]
       def write_notebook(self, key: str, data: Any) -> None
       def read_rulebook(self, name: str) -> Optional[str]
       def write_rulebook(self, name: str, content: str) -> None  # Archivist only
   ```

2. **notebook-schema.json**: JSON schema for Notebook entries
   ```json
   {
     "type": "object",
     "required": ["timestamp", "scope", "data"],
     "properties": {
       "timestamp": {"type": "string", "format": "date-time"},
       "scope": {"type": "string", "enum": ["Observer", "Theorist", "Validator", "Archivist"]},
       "data": {"type": "object"}
     }
   }
   ```

3. **validation-contract.md**: Validator scope requirements
   - Minimum 3 PCAPs
   - Test suite generation format
   - Pass/fail criteria
   - Evidence requirements

### Quickstart Guide

Create `quickstart.md` with:

1. **Setup**
   - Install dependencies: `poetry install`
   - Prepare workspace: `mkdir -p workspace/{notebook,rulebook,cookbook,pcaps}`
   - Verify environment: `poetry run pytest tests/unit/test_scopes/test_base.py`

2. **First Analysis**
   ```bash
   # Run Observer on PCAP
   poetry run python -m reshark.scopes.observer \
     --pcap tests/fixtures/synthetic_protocol.pcap \
     --session test-001
   
   # Generate hypotheses
   poetry run python -m reshark.scopes.theorist \
     --session test-001
   
   # Validate hypotheses
   poetry run python -m reshark.scopes.validator \
     --session test-001 \
     --pcaps tests/fixtures/validation_pcaps/*.pcap
   
   # Promote to Rulebook
   poetry run python -m reshark.scopes.archivist \
     --session test-001 \
     --operation promote \
     --hypothesis-id 0
   ```

3. **Verify Results**
   - Check Notebook: `cat workspace/notebook/test-001/*.json`
   - Check Rulebook: `ls workspace/rulebook/grammars/`
   - Run grammar tests: `poetry run pytest workspace/rulebook/tests/`

**Deliverables**: 
- `data-model.md`
- `contracts/` directory with interface specs
- `quickstart.md`

---

## Phase 2: Implementation Tasks

**Note**: Detailed task breakdown will be generated by `/speckit.tasks` command after Phase 0 and Phase 1 are complete. This section provides a high-level overview.

### Implementation Order (by dependency)

1. **Foundation** (Week 1-2)
   - Project structure and build system
   - Base Scope class with memory boundaries
   - Memory system (Notebook, Rulebook, Cookbook abstractions)
   - Logging and evidence trail utilities

2. **Observer Scope** (Week 2-3)
   - PCAP reader integration
   - Entropy calculation
   - Byte frequency analysis
   - Invariant detection
   - Notebook output format

3. **Theorist Scope** (Week 3-4)
   - Hypothesis generation from observations
   - Boundary detection logic
   - Field structure inference
   - Confidence scoring

4. **Validator Scope** (Week 4-5)
   - Test suite generation
   - Cross-PCAP validation
   - Negative testing
   - Results recording

5. **Archivist Scope** (Week 5-6)
   - Grammar generation
   - Rulebook promotion
   - Session archival
   - Metadata management

6. **Orchestration** (Week 6-7)
   - Manual workflow scripts
   - Human review checkpoints
   - Error handling
   - Progress reporting

7. **Testing & Documentation** (Week 7-8)
   - Comprehensive test suite
   - Constitutional compliance tests
   - End-to-end workflow tests
   - Documentation and examples

### Risks and Mitigations

**High Risk**:
- PCAP parsing edge cases → Extensive testing with varied PCAPs, robust error handling
- Validation framework correctness → Known-good test protocols, independent validation

**Medium Risk**:
- Memory boundary enforcement → Comprehensive unit tests, code review focus
- Performance on large PCAPs → Profiling, chunk processing, progress feedback

**Low Risk**:
- Documentation completeness → Document as you go, peer review

---

## Success Criteria Review

From spec, these must be achievable with the designed architecture:

✅ **SC-001**: 30-minute end-to-end analysis  
→ Achieved via: Efficient PCAP processing, cached statistics, simple workflow script

✅ **SC-002**: 100% memory boundary enforcement  
→ Achieved via: Base Scope class validation, pytest tests for violations

✅ **SC-003**: Traceable operations  
→ Achieved via: Notebook JSON format with timestamps and scope attribution

✅ **SC-004**: Validation rejects failing hypotheses  
→ Achieved via: Validator cross-PCAP testing logic

✅ **SC-005**: Grammars include test suites  
→ Achieved via: Archivist generates both grammar and pytest files

✅ **SC-006**: 1GB PCAP handling  
→ Achieved via: Streaming PCAP processing, memory-efficient data structures

✅ **SC-007**: Real protocol example  
→ Achieved via: Manual workflow proven on at least one real-world protocol

✅ **SC-008**: Cookbook documentation  
→ Achieved via: Markdown files documenting analysis methods

✅ **SC-009**: 80% test coverage  
→ Achieved via: pytest-cov, comprehensive unit and integration tests

✅ **SC-010**: Constitutional compliance verification  
→ Achieved via: Dedicated constitutional compliance test suite

**All success criteria are achievable with the proposed architecture.**

---

## Next Steps

1. **Execute Phase 0**: Run research workflow to resolve all technical unknowns
2. **Execute Phase 1**: Define data models, contracts, and quickstart guide
3. **Review Constitution**: Re-check constitutional compliance after design is complete
4. **Generate Tasks**: Run `/speckit.tasks` to create detailed implementation task breakdown
5. **Begin Implementation**: Start with foundation and Base Scope class

---

## Appendix: Constitutional Cross-Reference

| Constitution Section | Implementation Component | Compliance Mechanism |
|---------------------|-------------------------|---------------------|
| Section 2: Epistemic Law | Notebook evidence trails | JSON with timestamps, scope attribution |
| Section 3.1: Notebook | `reshark/memory/notebook.py` | Ephemeral JSON storage, session isolation |
| Section 3.2: Rulebook | `reshark/memory/rulebook.py` | Versioned grammars, test suites, promotion gate |
| Section 3.4: Cookbook | `reshark/memory/cookbook.py` | Markdown procedures, read-only for scopes |
| Section 4: Validation | `reshark/scopes/validator.py` | 3+ PCAP requirement, test suite generation |
| Section 5: Scopes | `reshark/scopes/base.py` | Authority scope property, memory boundaries |
| Section 10.1: Phase 1 | Manual orchestration scripts | `reshark/orchestration/` directory |
