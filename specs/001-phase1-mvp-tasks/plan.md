# Implementation Plan: Phase 1 MVP - Foundation Implementation

**Branch**: `001-phase1-mvp-implementation` | **Date**: 2026-01-11 | **Updated**: 2026-01-15 | **Spec**: [spec.md](spec.md)  
**Architecture**: v2.0.0 | **Input**: Feature specification from `/specs/001-phase1-mvp-tasks/spec.md`

## Summary

Implement the foundational reShark v2.0.0 system using VS Code + Dev Container + Cline + Ollama for HITL (Human-in-the-Loop) mode. Establish Books system (Notebook, Rulebook, Cookbook, Skills), agent architecture (Interpreters, Orchestration, Validation), tools registry, and skills-guided analysis workflows. This MVP proves the dual-mode general reverse engineering framework through evidence-based artifact analysis (PCAP, binaries), with Constitutional 3-sample validation and markdown skills directing LLM behavior. Phase 1 focuses on HITL foundation before Phase 2 autonomous mode with OpenHands and Patternbook.

## Technical Context

**Language/Version**: Python 3.12+  
**Development Environment**: VS Code + Dev Container + Cline + Ollama  
**LLM**: Ollama (deepseek-coder or similar) for local inference  
**Primary Dependencies**: numpy, scipy (statistical analysis), qdrant-client, sentence-transformers (Phase 2)  
**Tools**: tshark (PCAP), hexdump, strings, file (binary analysis), Poetry (dependency management)  
**Storage**: File system (JSON for Notebook, .ksy/.py for Rulebook grammars, Markdown for Skills/Cookbook)  
**Testing**: pytest with coverage >80% target  
**Containerization**: Dev Container (Phase 1), Docker Compose for Phase 2 (OpenHands + Qdrant)  
**Project Type**: Agent-based framework with Books system, skills library, tools registry  
**Performance Goals**: Process 1GB artifacts in <5 minutes, complete HITL workflow in <30 minutes  
**Constraints**: Constitutional 3-sample validation, evidence traceability, skills-guided workflows  
**Scale/Scope**: Single artifact per session, HITL orchestration via Cline, ~8K LOC for Phase 1  
**Key Benefit**: AI-assisted analysis via skills, transparent agent operations, rapid HITL iteration

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### ✅ Epistemic Law Compliance
- **Requirement**: All claims must be evidence-based, testable, and reproducible
- **Implementation**: Agents write evidence to Notebook with timestamps; validation requires 3+ samples per Constitutional requirement
- **Status**: PASS - spec enforces evidence logging (FR-036), validation requirements (FR-032)

### ✅ Books System Compliance
- **Requirement**: Knowledge properly organized (Notebook ephemeral, Rulebook validated, Cookbook methods, Skills guidance)
- **Implementation**: Agent base class provides Books access; only validated content promoted to Rulebook
- **Status**: PASS - spec mandates Books structure (FR-008 through FR-013)

### ✅ Validation and Falsification
- **Requirement**: Minimum 3 samples for cross-validation, negative testing, failure logging
- **Implementation**: GrammarValidator enforces 3-sample minimum (FR-032), records evidence (FR-033), supports negative tests (FR-034)
- **Status**: PASS - spec enforces Constitutional requirement with AgentError on violation

### ✅ Skills-Guided Analysis
- **Requirement**: Methodologies must be documented and applied consistently
- **Implementation**: 12+ markdown skills guide LLM behavior during Cline sessions; Skills library expanded incrementally
- **Status**: PASS - spec mandates skills system (FR-014 through FR-017)

### ⚠️ Phased Implementation
- **Requirement**: Phase 1 HITL foundation before Phase 2 autonomous mode
- **Implementation**: HITL with Cline only; OpenHands/Patternbook deferred to Phase 2
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
├── agents/                      # Agent implementations (replaces scopes/)
│   ├── __init__.py
│   ├── base.py                 # Base Agent class with Books access
│   ├── interpreters/           # Data analysis agents
│   │   ├── __init__.py
│   │   ├── data_interpreter.py         # Artifact extraction (PCAP streams, binary data)
│   │   ├── pattern_interpreter.py      # Statistical analysis (entropy, sequences)
│   │   └── autonomous_pattern_interpreter.py  # Phase 2: Patternbook integration
│   ├── orchestration/          # Workflow coordination
│   │   ├── __init__.py
│   │   ├── session_manager.py          # Session lifecycle management
│   │   └── task_orchestrator.py        # Multi-agent workflow coordination
│   ├── validation/             # Quality assurance
│   │   ├── __init__.py
│   │   ├── grammar_validator.py        # Cross-sample validation (3+ samples)
│   │   └── consistency_checker.py      # Result verification
│   └── analysis/               # Advanced analysis (Week 13)
│       ├── __init__.py
│       ├── fsm_analyzer.py             # State machine inference
│       └── structure_recovery.py       # Data structure inference
│
├── books/                       # Books system (replaces memory/)
│   ├── __init__.py
│   ├── notebook/               # Ephemeral observations
│   │   └── sessions/           # Session-based directories
│   ├── rulebook/               # Validated grammars
│   │   ├── grammars/           # .ksy or .py grammar files
│   │   └── schemas/            # JSON schemas
│   ├── cookbook/               # Analysis methods
│   │   ├── README.md
│   │   └── methods/            # Workflow documentation
│   ├── patternbook/            # Phase 2: Vector DB integration
│   │   └── pattern_store.py
│   └── skills/                 # Markdown methodology docs (read by LLM)
│       ├── README.md
│       ├── protocol/           # Protocol analysis skills
│       │   └── data-extraction.md
│       ├── pattern/            # Pattern detection skills
│       │   ├── entropy-analysis.md
│       │   └── sequence-detection.md
│       ├── binary/             # Binary analysis skills
│       │   ├── boundary-detection.md
│       │   └── type-inference.md
│       └── grammar/            # Grammar operations skills
│           └── grammar-validation.md
│
├── tools/                       # Tool wrappers
│   ├── __init__.py
│   ├── registry.py             # ToolsRegistry for unified invocation
│   ├── tshark_wrapper.py       # PCAP analysis wrapper
│   ├── binary_tools.py         # hexdump, strings, file wrappers
│   └── entropy.py              # Entropy calculation utilities
│
├── autonomous/                  # Phase 2: Autonomous mode
│   ├── __init__.py
│   ├── openhands_adapter.py    # OpenHands integration
│   ├── tasks.py                # Task definitions
│   └── workflows/              # Autonomous workflow implementations
│
├── cli.py                       # Command-line interface
│
└── utils/                       # Shared utilities
    ├── __init__.py
    ├── logging.py              # Evidence trail logging
    └── validation.py           # Input validation

tests/                           # Test suite
├── agents/                      # Agent tests (replaces test_scopes/)
│   ├── interpreters/
│   ├── orchestration/
│   └── validation/
├── books/                       # Books system tests
│   ├── test_notebook.py
│   └── test_skills.py
├── tools/                       # Tool wrapper tests
│   └── test_tshark_wrapper.py
├── integration/                 # Integration tests
│   ├── test_complete_workflow.py  # End-to-end workflow
│   └── test_constitutional_compliance.py  # Constitutional compliance
├── autonomous/                  # Phase 2: Autonomous mode tests
│   └── test_openhands_integration.py
├── performance/                 # Performance benchmarks
│   └── test_benchmarks.py
└── fixtures/                    # Test data
    ├── protocol_generator.py   # Synthetic test data
    └── samples/                # Test artifacts

books/                           # Books workspace (gitignored for local work)
├── notebook/
│   └── sessions/               # Session directories
├── rulebook/
│   ├── grammars/
│   └── schemas/
├── cookbook/
│   └── methods/
└── skills/                     # Symlink to reshark/books/skills/

docker/                          # Execution environment
├── Dockerfile.pcap
├── Dockerfile.openhands        # Phase 2
├── docker-compose.yml          # Phase 2: OpenHands + Ollama + Qdrant
└── README.md

docs/                            # Project documentation
├── setup.md
└── architecture.md

pyproject.toml                   # Python package configuration
README.md                        # Project overview
runtime.yaml                     # Runtime configuration
.gitignore                       # Ignore books/ workspace
```

**Structure Decision**: Single Python package with modular agent architecture. Agents are the primary abstraction with specialized roles (DataInterpreter, PatternInterpreter, TaskOrchestrator, SessionManager, GrammarValidator, ConsistencyChecker). Books system provides knowledge boundaries (Notebook, Rulebook, Cookbook, Skills). Skills library (markdown files) guides LLM behavior for consistent analysis patterns. Phase 1 uses Cline + Ollama for HITL workflows. Phase 2 adds OpenHands for autonomous execution with Patternbook (Qdrant vector database). Docker provides Dev Container (Phase 1) and autonomous environment (Phase 2) with tshark + Ollama + Qdrant.

## Complexity Tracking

> **No violations - this section intentionally blank**

All constitutional requirements are met by design. No technical debt or complexity justifications needed.

---

## Phase 0: Research & Technical Decisions

**Goal**: Resolve all NEEDS CLARIFICATION items and establish technical foundations

### Research Topics

1. **Tool Integration Strategy**
   - **Question**: Best approach for wrapping tshark/hexdump/strings commands?
   - **Options**: subprocess.run() with parsing vs python-pcap libraries vs scapy
   - **Decision Criteria**: Performance, error handling, output parsing reliability
   - **Output**: ToolsRegistry design with wrapper implementations in research.md

2. **Skills Library Design**
   - **Question**: How to structure markdown methodology files for LLM consumption?
   - **Options**: Single files per skill vs hierarchical categories vs template system
   - **Decision Criteria**: Ease of maintenance, LLM context window efficiency, versioning
   - **Output**: Skills organization pattern and example files in research.md

3. **Agent Communication Patterns**
   - **Question**: How do agents coordinate through Books system?
   - **Options**: Direct function calls vs message passing vs event-driven
   - **Decision Criteria**: Testability, auditability, phase transition compatibility
   - **Output**: Agent coordination architecture in research.md

4. **Session Isolation**
   - **Question**: How to ensure session isolation in Notebook filesystem?
   - **Options**: UUID-based directories vs timestamp-based vs user-specified
   - **Decision Criteria**: Collision prevention, traceability, cleanup
   - **Output**: Session management design in research.md

5. **Grammar Validation Framework**
   - **Question**: How to validate grammars across 3+ samples constitutionally?
   - **Options**: Kaitai Struct (.ksy) vs Python parsers vs hybrid approach
   - **Decision Criteria**: Cross-sample testability, grammar reusability, error diagnostics
   - **Output**: Grammar format specification and validation workflow in research.md

6. **LLM Integration (Phase 1)**
   - **Question**: How to integrate Ollama Local LLM with Cline in Dev Container?
   - **Options**: HTTP API vs direct Python library vs Cline native integration
   - **Decision Criteria**: Latency, error handling, context management
   - **Output**: Ollama integration pattern for HITL workflows in research.md

7. **Autonomous Mode Architecture (Phase 2)**
   - **Question**: How to integrate OpenHands for autonomous task execution?
   - **Options**: Docker-in-Docker vs shared volumes vs REST API
   - **Decision Criteria**: Security isolation, state persistence, performance
   - **Output**: OpenHands adapter design and Patternbook integration in research.md

### Known Unknowns (to be resolved in Phase 0)

- Optimal tshark filter expressions for protocol feature extraction
- Entropy threshold values for control byte detection (agent-specific)
- Skills library taxonomy and file naming conventions
- ToolsRegistry error handling and timeout strategies
- Notebook JSON schema versioning approach
- Patternbook embedding model selection (Phase 2)
- OpenHands task definition format (Phase 2)

### Dependencies

- tshark, hexdump, strings, file available in Dev Container
- Ollama Local LLM running in Dev Container or host
- Python 3.12+ with asyncio for agent coordination
- pytest for grammar validation test generation
- File system permissions for books/ workspace isolation
- Docker Compose for Phase 2 (OpenHands + Qdrant)

**Deliverable**: `research.md` with all technical decisions documented and unknowns resolved

---

## Phase 1: Design & Contracts

**Goal**: Define data models, API contracts, and agent architecture

### Data Model Design

Create `data-model.md` with:

1. **Core Entities**
   - Agent (base class, Books access methods, tool invocation)
   - Session (ID, Notebook paths, metadata, timestamp)
   - Observation (entropy map, byte frequency, invariants, tool outputs)
   - Pattern (statistical claim, confidence score, evidence samples)
   - ValidationResult (pass/fail status, cross-sample evidence, test outputs)
   - Grammar (structure definition, parsing rules, Kaitai Struct or Python)
   - Skill (markdown file path, category, content)
   - ToolResult (command, stdout, stderr, exit code, timestamp)

2. **Relationships**
   - Session 1:N Observations (DataInterpreter creates observations per session)
   - Observation 1:N Patterns (PatternInterpreter generates patterns from observations)
   - Pattern 1:N ValidationResults (GrammarValidator tests across 3+ samples)
   - ValidationResult 1:1 Grammar (validated pattern produces Rulebook grammar)
   - Agent N:M Skills (agents reference skills for methodology guidance)
   - Agent 1:N ToolResults (agents invoke tools via ToolsRegistry)

3. **State Transitions**
   - Session: created → active → completed → archived
   - Observation: recorded → analyzed → pattern-matched
   - Pattern: detected → validated/rejected → grammar-generated/discarded
   - Grammar: drafted → cross-sample-tested (3+ samples) → promoted-to-rulebook/rejected
   - Skill: versioned markdown files (immutable per version)

### API Contracts

Create `contracts/` with:

1. **agent-interface.md**: Base Agent class contract
   ```python
   class Agent(ABC):
       def __init__(self, session_id: str, books: BooksAccess):
           self.session_id = session_id
           self.books = books
       
       @abstractmethod
       async def execute(self, context: Dict[str, Any]) -> AgentResult:
           """Main execution method with Books access"""
       
       @abstractmethod
       def get_required_skills(self) -> List[str]:
           """Return list of required skill file paths"""
       
       def invoke_tool(self, tool_name: str, args: Dict) -> ToolResult:
           """Invoke tool via ToolsRegistry"""
       
       def read_notebook(self, key: str) -> Optional[Dict]:
           """Read from session Notebook"""
       
       def write_notebook(self, key: str, data: Dict) -> None:
           """Write to session Notebook (ephemeral)"""
       
       def read_rulebook(self, grammar_name: str) -> Optional[str]:
           """Read validated grammar from Rulebook"""
       
       def write_rulebook(self, grammar_name: str, content: str) -> None:
           """Write validated grammar (GrammarValidator only)"""
       
       def read_skill(self, skill_path: str) -> str:
           """Read methodology guidance from Skills library"""
   ```

2. **books-schemas.json**: JSON schemas for Books system
   ```json
   {
     "notebook_entry": {
       "type": "object",
       "required": ["timestamp", "agent", "operation", "data"],
       "properties": {
         "timestamp": {"type": "string", "format": "date-time"},
         "agent": {"type": "string", "enum": [
           "DataInterpreter", "PatternInterpreter", "TaskOrchestrator",
           "SessionManager", "GrammarValidator", "ConsistencyChecker"
         ]},
         "operation": {"type": "string"},
         "data": {"type": "object"}
       }
     },
     "pattern": {
       "type": "object",
       "required": ["pattern_id", "claim", "confidence", "evidence"],
       "properties": {
         "pattern_id": {"type": "string"},
         "claim": {"type": "string"},
         "confidence": {"type": "number", "minimum": 0, "maximum": 1},
         "evidence": {"type": "array", "minItems": 1}
       }
     },
     "grammar": {
       "type": "object",
       "required": ["name", "version", "format", "content", "validation_samples"],
       "properties": {
         "name": {"type": "string"},
         "version": {"type": "string"},
         "format": {"type": "string", "enum": ["kaitai", "python"]},
         "content": {"type": "string"},
         "validation_samples": {"type": "array", "minItems": 3}
       }
     }
   }
   ```

3. **validation-contract.md**: GrammarValidator requirements
   - Constitutional: Minimum 3 independent samples (hardcoded)
   - Test suite generation: pytest tests from grammar
   - Pass criteria: All 3+ samples parse successfully
   - Evidence requirements: Cross-sample consistency metrics
   - Rejection handling: Document failure reasons in Notebook

### Quickstart Guide

Create `quickstart.md` with:

#### Getting Started: Two Paths

**Path 1: Private Analysis Work (Recommended for Sensitive Protocols)**

Use this path when analyzing proprietary/confidential protocols where findings must remain private:

1. **Download Repository as ZIP**
   - Visit https://github.com/kmaneesh/reShark
   - Click "Code" → "Download ZIP"
   - Extract to your local workspace
   - **Why ZIP?** No git history, no accidental pushes to public repo

2. **Create Your Own Private Repository**
   ```bash
   cd reshark-main  # Extracted directory
   rm -rf .git      # Remove any git metadata
   git init
   git remote add origin <your-private-repo-url>
   git add -A
   git commit -m "Initial reShark framework setup"
   git push -u origin main
   ```

3. **Configure for Private Work**
   - Add `books/` to `.gitignore` (already configured)
   - Add custom entries to `.gitignore`:
     ```
     # Private analysis data
     books/notebook/sessions/*
     books/rulebook/grammars/*
     books/cookbook/methods/proprietary-*
     
     # Proprietary skills (keep private)
     books/skills/proprietary/
     
     # Custom tools for proprietary protocols
     tools/proprietary_*
     
     # Client/project-specific data
     **/client-*
     **/project-*
     *.pcap
     *.pcapng
     ```

4. **Add Your Proprietary Knowledge**
   - Create custom skills: `books/skills/proprietary/your-protocol-analysis.md`
   - Add proprietary tools: `tools/proprietary_parser.py`
   - Document internal methods: `books/cookbook/methods/proprietary-workflow.md`
   - **Your sensitive knowledge stays in YOUR repo**

**Path 2: Contributing to reShark (For Framework Improvements)**

Use this path when contributing general framework enhancements, bug fixes, or non-sensitive features:

1. **Fork and Clone**
   ```bash
   # Fork https://github.com/kmaneesh/reShark on GitHub first
   git clone https://github.com/<your-username>/reShark.git
   cd reShark
   git remote add upstream https://github.com/kmaneesh/reShark.git
   ```

2. **Create Feature Branch**
   ```bash
   git checkout -b feature/your-contribution
   # Examples: feature/improved-entropy-analysis, fix/tool-wrapper-timeout
   ```

3. **Make Non-Sensitive Changes**
   - Framework code: `reshark/agents/`, `reshark/books/`, `reshark/tools/`
   - General-purpose skills: `books/skills/protocol/`, `books/skills/pattern/`
   - Tests: `tests/agents/`, `tests/integration/`
   - Documentation: `docs/`, `README.md`
   - **NEVER commit**:
     - Proprietary protocol samples
     - Client-specific analysis results
     - Confidential grammars or methods
     - Private skills or tools

4. **Submit Pull Request**
   ```bash
   git add <changed-files>
   git commit -m "Clear description of your contribution"
   git push origin feature/your-contribution
   ```
   - Open PR at https://github.com/kmaneesh/reShark/pulls
   - Describe: What problem does this solve? How does it work?
   - Maintainers will review and merge

**Best Practices for Keeping Sensitive Data Private:**

- **Use Path 1 for analysis work**, Path 2 for framework contributions
- **Never mix**: Keep your private repo separate from your fork
- **Regularly sync**: `git pull upstream main` in your fork to get updates
- **Apply updates to private repo**: Cherry-pick framework improvements to your private repo as needed
- **Review before commit**: Always check `git diff` for sensitive strings, IPs, domains, protocol specifics
- **Use environment variables**: Store API keys, credentials, paths in `.env` (add to `.gitignore`)
- **Separate branches**: Use `main` for framework, `private-analysis` for sensitive work in your private repo

#### Setup (Phase 1 HITL Mode)

1. **Development Environment**
   - Open in VS Code Dev Container: "Reopen in Container"
   - Verify Ollama running: `ollama list`
   - Install dependencies: `poetry install`
   - Prepare Books workspace: `mkdir -p books/{notebook/sessions,rulebook/grammars,cookbook/methods}`
   - Verify environment: `poetry run pytest tests/agents/test_base.py`
   - Load Cline extension and configure with Ollama endpoint

#### First Analysis (HITL with Cline)
   ```bash
   # Start interactive session with Cline
   # Cline prompt: "Analyze protocol_x.pcap using DataInterpreter"
   
   # Cline will:
   # 1. Read skills/protocol/data-extraction.md for methodology
   # 2. Invoke DataInterpreter agent
   # 3. Use ToolsRegistry to run tshark/hexdump
   # 4. Write observations to books/notebook/sessions/<session_id>/
   # 5. Present findings for human review
   
   # Human iterates with Cline to refine analysis
   # Cline prompt: "Use PatternInterpreter to find patterns in the observations"
   
   # Cline will:
   # 1. Read skills/pattern/entropy-analysis.md and sequence-detection.md
   # 2. Invoke PatternInterpreter agent
   # 3. Analyze Notebook observations for statistical patterns
   # 4. Write pattern hypotheses to Notebook
   # 5. Ask human for validation direction
   
   # Human requests validation
   # Cline prompt: "Use GrammarValidator to validate patterns across 3 samples"
   
   # Cline will:
   # 1. Read skills/grammar/grammar-validation.md
   # 2. Invoke GrammarValidator with 3+ sample paths
   # 3. Generate pytest tests from grammar hypothesis
   # 4. Run cross-sample validation
   # 5. On success: Promote grammar to Rulebook
   # 6. On failure: Document rejection reasons in Notebook
   ```

3. **Verify Results**
   - Check Notebook: `cat books/notebook/sessions/<session_id>/*.json`
   - Check Rulebook: `ls books/rulebook/grammars/`
   - View skills used: `ls books/skills/protocol/ books/skills/pattern/ books/skills/grammar/`
   - Verify constitutional compliance: `poetry run pytest tests/integration/test_constitutional_compliance.py`

**Deliverables**: 
- `data-model.md`
- `contracts/` directory with interface specs
- `quickstart.md`

---

## Phase 2: Implementation Tasks

**Note**: Detailed task breakdown will be generated by `/speckit.tasks` command after Phase 0 and Phase 1 are complete. This section provides a high-level overview.

### Implementation Order (by dependency)

1. **Foundation** (Week 1-2)
   - Books system: Notebook, Rulebook, Cookbook, Skills base classes
   - Agent base class with Books access methods
   - ToolsRegistry with tshark/hexdump/strings/file wrappers
   - Session management (UUID-based, filesystem isolation)
   - Logging infrastructure (evidence trail)
   - Skills library: Initial 5-7 markdown methodology files
   - Dev Container setup with tshark + Ollama
   - pytest fixtures for synthetic protocol data

2. **Core Agents** (Week 3-4)
   - DataInterpreter: Tool invocation, artifact extraction, Notebook writing
   - PatternInterpreter: Statistical analysis, entropy calculation, pattern detection
   - GrammarValidator: Cross-sample validation (3+ samples), pytest test generation
   - ConsistencyChecker: Result verification, evidence chain validation
   - SessionManager: Lifecycle management, Books cleanup
   - Skills library expansion: 12+ methodology files across protocol/pattern/binary/grammar categories

3. **Orchestration** (Week 5)
   - TaskOrchestrator: Multi-agent workflow coordination
   - HITL workflow integration with Cline + Ollama
   - Agent communication patterns through Notebook
   - Error handling and agent failure recovery
   - Constitutional compliance enforcement (3-sample minimum)

4. **Testing & Validation** (Week 6-7)
   - Unit tests for all agents (>80% coverage)
   - Integration tests: Complete HITL workflow
   - Constitutional compliance tests (3-sample requirement)
   - Performance benchmarks: Analysis time, memory usage
   - Synthetic protocol generator for test fixtures
   - Skills library validation: Ensure LLM readability

5. **CLI & Documentation** (Week 8)
   - Command-line interface for standalone agent invocation
   - README updates with v2.0.0 architecture
   - Quickstart guide for Dev Container + Cline + Ollama
   - Skills documentation: Usage examples for each skill file
   - Architecture diagrams: Agent interactions, Books flow

6. **Phase 2 Preparation** (Optional for MVP)
   - OpenHands adapter design (autonomous task execution)
   - Patternbook integration: Qdrant vector DB, embedding generation
   - AutonomousPatternInterpreter: Semantic search over Patternbook
   - Docker Compose: OpenHands + Ollama + Qdrant

### Risks and Mitigations

**High Risk**:
- Tool wrapper error handling (tshark timeouts, malformed output) → Comprehensive timeout handling, stderr parsing, fallback strategies
- LLM hallucination in pattern detection → Skills-guided prompts, evidence-based validation, 3-sample constitutional requirement
- Constitutional compliance enforcement (3-sample minimum) → Hardcoded validation in GrammarValidator, pytest compliance tests

**Medium Risk**:
- Skills library maintainability (markdown sprawl) → Taxonomic organization, version control, regular review cycle
- Agent coordination complexity → Clear Notebook schemas, well-defined agent responsibilities, integration tests
- Performance on large PCAPs → Streaming processing, progress feedback, resource monitoring

**Low Risk**:
- Dev Container setup complexity → docker-compose with pre-configured services, documented setup process
- Documentation completeness → Document as you go, peer review, automated doc generation

---

## Success Criteria Review

From spec, these must be achievable with the designed architecture:

✅ **SC-001**: 30-minute end-to-end analysis  
→ Achieved via: ToolsRegistry for efficient tool invocation, skills-guided LLM prompts reduce iteration, Cline HITL workflow for fast feedback

✅ **SC-002**: 100% constitutional compliance (3-sample validation)  
→ Achieved via: Hardcoded 3-sample minimum in GrammarValidator, pytest compliance tests, evidence chain in Notebook

✅ **SC-003**: Traceable operations  
→ Achieved via: Notebook JSON format with timestamps and scope attribution

✅ **SC-004**: Validation rejects failing hypotheses  
→ Achieved via: Validator cross-PCAP testing logic

✅ **SC-005**: Grammars include test suites  
→ Achieved via: Archivist generates both grammar and pytest files

✅ **SC-003**: 100% agent responsibility isolation  
→ Achieved via: Agent base class with defined interfaces, Books access boundaries, clear role separation

✅ **SC-004**: Skills-guided methodology consistency  
→ Achieved via: 12+ markdown methodology files in books/skills/, LLM reads skills before agent invocation, version-controlled skills

✅ **SC-005**: <8000 LOC for Phase 1 MVP  
→ Achieved via: Focus on HITL workflows only, defer autonomous mode to Phase 2, lean agent implementations

✅ **SC-006**: Ollama LLM integration performance (<10s per agent call)  
→ Achieved via: Local Ollama instance, optimized prompts with skills context, async agent execution

✅ **SC-007**: ToolsRegistry completeness (tshark, hexdump, strings, file)  
→ Achieved via: Wrapper classes with unified interface, error handling, timeout management

✅ **SC-008**: Cross-sample validation (3+ samples)  
→ Achieved via: GrammarValidator with hardcoded 3-sample minimum, pytest test generation per grammar

✅ **SC-009**: Notebook session isolation  
→ Achieved via: UUID-based session directories, filesystem permissions, cleanup on SessionManager close

✅ **SC-010**: Dev Container HITL workflow  
→ Achieved via: docker-compose with VS Code + Cline + Ollama + tshark, documented setup in docs/setup.md

**All success criteria are achievable with the proposed agent-based architecture.**

---

## Next Steps

1. **Execute Phase 0**: Run research workflow to resolve all technical unknowns (tool wrappers, skills design, agent coordination)
2. **Execute Phase 1**: Define data models, agent contracts, Books schemas, and quickstart guide
3. **Review Constitution**: Re-check constitutional compliance after design is complete (focus on 3-sample requirement enforcement)
4. **Generate Tasks**: Run `/speckit.tasks` to create detailed implementation task breakdown for agents and Books
5. **Begin Implementation**: Start with Books system foundation and Agent base class, then DataInterpreter

---

## Appendix: Constitutional Cross-Reference

| Constitution Section | Implementation Component | Compliance Mechanism |
|---------------------|-------------------------|---------------------|
| Section 2: Epistemic Law | Notebook evidence trails | JSON with timestamps, agent attribution, operation metadata |
| Section 3.1: Notebook | `reshark/books/notebook/` | Ephemeral JSON storage, session-based directories, UUID isolation |
| Section 3.2: Rulebook | `reshark/books/rulebook/` | Versioned grammars (.ksy or .py), 3+ sample validation, promotion gate |
| Section 3.3: Cookbook | `reshark/books/cookbook/` | Workflow documentation (markdown), methods library, read-only |
| Section 3.4: Skills | `reshark/books/skills/` | Methodology guidance (markdown), LLM-readable, version-controlled |
| Section 4: Validation | `reshark/agents/validation/grammar_validator.py` | Hardcoded 3+ sample requirement, pytest test generation, cross-sample metrics |
| Section 5: Agent Responsibilities | `reshark/agents/base.py` | Agent interface with Books access methods, clear role definitions |
| Section 10.1: Phase 1 HITL | Cline + Ollama + Dev Container | Human-in-the-loop workflows, skills-guided prompts, VS Code integration |
| Section 10.2: Phase 2 Autonomous | `reshark/autonomous/` (future) | OpenHands adapter, Patternbook vector DB, autonomous task execution |
