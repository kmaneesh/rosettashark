# reShark System Requirements Specification (SRS)

**Version**: 2.0.0  
**Date**: 2026-01-15  
**Status**: Draft  
**Governed By**: reShark Constitution v2.0.0

---

## 1. Introduction

### 1.1 Purpose

This document specifies the functional and non-functional requirements for reShark, a general-purpose reverse engineering framework. It serves as the authoritative requirements baseline for both Human-in-the-Loop (HITL) mode (Dev Container + Cline + Local LLM) and Autonomous mode (OpenHands + Local LLM).

### 1.2 Scope

reShark is a hybrid framework for reverse engineering complex systems. The framework operates in two complementary modes:

**Human-in-the-Loop Mode**:
- Interactive analysis and exploration
- Knowledge acquisition and skill development
- Dev Container + Cline + Local LLM
- Direct tool access and debugging

**Autonomous Mode**:
- Self-directed analysis campaigns
- Learned behavior application
- OpenHands + Local LLM
- Scaled parallel processing

**Core Capabilities**:
- Statistical analysis and pattern detection
- Hypothesis generation and validation
- Grammar and model construction
- Knowledge capture and reuse

**Out of Scope**:
- Vulnerability discovery or exploitation
- Live production system analysis
- Encrypted data breaking
- Semantic interpretation without structural understanding

### 1.3 Definitions

- **HITL (Human-in-the-Loop)**: Interactive mode where human operator guides analysis with AI assistance
- **Autonomous Mode**: Self-directed execution where agents apply learned patterns
- **Agent**: Autonomous component that performs specialized analysis tasks
- **Skill**: Reusable capability that can be invoked by agents or humans
- **Books**: Knowledge containers organized by type (notebook, rulebook, cookbook, patternbook)
- **Notebook**: Session-specific working memory for hypotheses and measurements
- **Rulebook**: Validated grammars, schemas, and structural models
- **Patternbook**: Pattern embeddings and similarity memory
- **Cookbook**: Procedural knowledge and analysis methods
- **Tool**: Executable script or utility (pcap processing, statistical analysis, etc.)
- **Lab**: Containerized environment containing pre-configured toolset
- **Session**: Bounded analysis context with specific target and objectives

### 1.4 References

- reShark Constitution v2.0.0
- VSCode Dev Containers Documentation
- Cline AI Assistant Documentation
- OpenHands Documentation
- Local LLM Integration (Ollama/LM Studio)
- Reverse Engineering Best Practices

---

## 2. System Overview

### 2.1 System Context

```
┌──────────────────────────────────────────────────────────┐
│                    reShark Framework                      │
│                                                           │
│  ┌──────────────────────┐    ┌─────────────────────┐   │
│  │ Human-in-the-Loop    │    │  Autonomous Mode    │   │
│  │                      │    │                     │   │
│  │  Dev Container       │    │  OpenHands          │   │
│  │  + Cline             │    │  + Local LLM        │   │
│  │  + Local LLM         │    │                     │   │
│  │                      │    │                     │   │
│  │  ┌────────────────┐ │    │  ┌────────────────┐ │   │
│  │  │ Interactive    │ │    │  │ Learned        │ │   │
│  │  │ Exploration    │ │    │  │ Behavior       │ │   │
│  │  │ Knowledge      │◄─┼────┼──┤ Application    │ │   │
│  │  │ Acquisition    │ │    │  │ Scaled Exec    │ │   │
│  │  └────────────────┘ │    │  └────────────────┘ │   │
│  └──────────┬───────────┘    └───────────┬─────────┘   │
│             │                            │             │
│             └────────────┬───────────────┘             │
│                          │                             │
│         ┌────────────────▼─────────────────┐          │
│         │         Agents                    │          │
│         │  - interpreters (analyze)         │          │
│         │  - orchestration (coordinate)     │          │
│         │  - validation (verify)            │          │
│         │  - scopes (specialized views)     │          │
│         └────────┬──────────────────────────┘          │
│                  │                                      │
│         ┌────────▼──────────┐    ┌──────────────┐    │
│         │      Skills        │    │    Tools     │    │
│         │  - Pattern detect  │    │  - Scripts   │    │
│         │  - Grammar build   │    │  - Utilities │    │
│         │  - Analysis flows  │    │  - Analyzers │    │
│         └────────┬───────────┘    └──────┬───────┘    │
│                  │                       │             │
│         ┌────────▼───────────────────────▼───┐        │
│         │            Books                    │        │
│         │  - Notebook (working memory)        │        │
│         │  - Rulebook (validated knowledge)   │        │
│         │  - Cookbook (procedures)            │        │
│         │  - Patternbook (embeddings)         │        │
│         └─────────────────────────────────────┘        │
│                          │                             │
│         ┌────────────────▼─────────────────┐          │
         │          Lab                      │          │
│         │  Pre-configured environment       │          │
│         │  with loaded toolset              │          │
│         └───────────────────────────────────┘          │
└──────────────────────────────────────────────────────────┘
           │
           │ Operates On
           ↓
    ┌──────────────┐
    │ Target Data  │
    │ - PCAPs      │
    │ - Binaries   │
    │ - Logs       │
    │ - Artifacts  │
    └──────────────┘
```

### 2.2 Architectural Components

**1. Execution Modes**

- **HITL Mode (Dev Container + Cline + Local LLM)**
  - Interactive workspace for exploration
  - Human guides analysis direction
  - AI assists with pattern detection and hypothesis generation
  - Knowledge captured for autonomous reuse
  - Direct debugging and tool invocation

- **Autonomous Mode (OpenHands + Local LLM)**
  - Self-directed execution of learned workflows
  - Applies patterns from Books
  - Parallel multi-target analysis
  - Checkpoint-based human oversight
  - Automated validation and reporting

**2. Agents (`/agents`)**
   - **Interpreters**: Analyze and extract structure from raw data
   - **Orchestration**: Coordinate multi-agent workflows
   - **Validation**: Verify hypotheses and results
   - **Scopes**: Specialized analytical perspectives

**3. Skills (`/skills`)**
   - Reusable capabilities invocable by agents or humans
   - Pattern detection algorithms
   - Grammar construction methods
   - Statistical analysis procedures
   - Structured as composable modules

**4. Tools (`/tools`)**
   - Scripts for data processing
   - Utilities for analysis tasks
   - Format converters and parsers
   - Visualization generators
   - Organized by function

**5. Books (`/books`)**
   - **Notebook**: Session-specific working memory (ephemeral)
   - **Rulebook**: Validated models and grammars (persistent)
   - **Cookbook**: Procedural knowledge (HOW-TO guides)
   - **Patternbook**: Vector embeddings for similarity search

**6. Lab (`/labs`)**
   - Pre-configured analysis environment
   - Loaded with all tools and dependencies
   - Used by both HITL and Autonomous modes
   - Network-isolated for security
   - Reproducible across systems

---

## 3. Functional Requirements

### 3.1 Execution Modes

**REQ-MODE-001**: System SHALL support Human-in-the-Loop (HITL) mode via Dev Container + Cline + Local LLM.

**REQ-MODE-002**: System SHALL support Autonomous mode via OpenHands + Local LLM.

**REQ-MODE-003**: Knowledge acquired in HITL mode SHALL be transferable to Autonomous mode.

**REQ-MODE-004**: System SHALL allow mode switching within the same analysis session.

**REQ-MODE-005**: Both modes SHALL operate within the same Lab environment.

**REQ-MODE-006**: System SHALL maintain execution context when transitioning between modes.

### 3.2 Data Processing

**REQ-DATA-001**: System SHALL accept multiple input formats (PCAP, PCAPNG, binary files, logs, network dumps).

**REQ-DATA-002**: System SHALL extract structured information from raw data sources.

**REQ-DATA-003**: System SHALL preserve metadata (timestamps, source context, provenance).

**REQ-DATA-004**: System SHALL support files up to 10GB in size.

**REQ-DATA-005**: System SHALL detect and report corrupted or incomplete data.

**REQ-DATA-006**: System SHALL handle multiple concurrent data processing tasks in Autonomous mode.

### 3.3 Pattern Analysis

**REQ-PAT-001**: System SHALL perform statistical analysis on data structures.

**REQ-PAT-002**: System SHALL detect repeating patterns and invariants.

**REQ-PAT-003**: System SHALL identify structural boundaries and delimiters.

**REQ-PAT-004**: System SHALL compute entropy, frequency distributions, and correlation matrices.

**REQ-PAT-005**: System SHALL detect anomalies and outliers in data patterns.

**REQ-PAT-006**: System SHALL output all measurements to Notebook with timestamps and evidence.

### 3.4 Hypothesis Generation

**REQ-HYPO-001**: System SHALL generate testable hypotheses about data structure and behavior.

**REQ-HYPO-002**: System SHALL identify candidate structural elements (headers, fields, boundaries).

**REQ-HYPO-003**: System SHALL infer relationships and dependencies between elements.

**REQ-HYPO-004**: System SHALL propose state models and behavioral patterns.

**REQ-HYPO-005**: System SHALL document reasoning and evidence for each hypothesis in Notebook.

**REQ-HYPO-006**: System SHALL rank hypotheses by confidence and supporting evidence.

**REQ-HYPO-007**: In HITL mode, hypotheses SHALL be presented for human review before validation.

**REQ-HYPO-008**: In Autonomous mode, hypotheses SHALL proceed to automated validation.

### 3.5 Validation

**REQ-VAL-001**: System SHALL create executable test suites for hypotheses.

**REQ-VAL-002**: System SHALL test hypotheses against minimum 3 distinct data sources (cross-validation).

**REQ-VAL-003**: System SHALL perform negative testing with edge cases.

**REQ-VAL-004**: System SHALL test boundary conditions and exceptional scenarios.

**REQ-VAL-005**: System SHALL log all validation attempts with pass/fail outcomes to Notebook.

**REQ-VAL-006**: System SHALL prevent Rulebook promotion for hypotheses that fail validation.

**REQ-VAL-007**: System SHALL record failed hypotheses to prevent re-investigation.

**REQ-VAL-008**: Validation failures in Autonomous mode SHALL trigger human review checkpoints.

### 3.6 Grammar and Model Construction

**REQ-GRAM-001**: System SHALL generate formal grammars for validated structures.

**REQ-GRAM-002**: Grammars SHALL define boundaries, types, fields, and relationships.

**REQ-GRAM-003**: Grammars SHALL include parsing logic and validation rules.

**REQ-GRAM-004**: Grammars SHALL be versioned and timestamped.

**REQ-GRAM-005**: Grammars SHALL reference source data and validation evidence.

**REQ-GRAM-006**: System SHALL generate schemas in multiple formats (Python, YAML, JSON Schema).

### 3.7 Agent Management

**REQ-AGENT-001**: System SHALL provide interpreters for data analysis tasks.

**REQ-AGENT-002**: System SHALL provide orchestration agents for workflow coordination.

**REQ-AGENT-003**: System SHALL provide validation agents for hypothesis testing.

**REQ-AGENT-004**: System SHALL provide scope agents for specialized analytical perspectives.

**REQ-AGENT-005**: Agents in Autonomous mode SHALL execute learned behaviors from Books.

**REQ-AGENT-006**: Agents SHALL communicate via shared Notebook state.

**REQ-AGENT-007**: Agent failures SHALL be logged and SHALL NOT crash the system.

### 3.8 Skills Library

**REQ-SKILL-001**: System SHALL maintain library of reusable skills.

**REQ-SKILL-002**: Skills SHALL be invocable by both humans and agents.

**REQ-SKILL-003**: Skills SHALL be composable into complex workflows.

**REQ-SKILL-004**: New skills learned in HITL mode SHALL be added to library for autonomous reuse.

**REQ-SKILL-005**: Skills SHALL include documentation and usage examples.

**REQ-SKILL-006**: Skills SHALL specify input requirements and output formats.

### 3.9 Books Management

**REQ-BOOK-001**: System SHALL maintain Notebook as session-specific working memory (JSON/Markdown).

**REQ-BOOK-002**: System SHALL store Rulebook entries as versioned formal models.

**REQ-BOOK-003**: System SHALL maintain Cookbook as procedural knowledge (Markdown).

**REQ-BOOK-004**: System SHALL maintain Patternbook as vector embeddings database.

**REQ-BOOK-005**: System SHALL enforce memory boundaries (no Notebook data in Rulebook without validation).

**REQ-BOOK-006**: System SHALL provide promotion workflow from Notebook to Rulebook.

**REQ-BOOK-007**: System SHALL archive completed analysis sessions.

**REQ-BOOK-008**: System SHALL support Rulebook rollback if evidence contradicts promoted entry.

**REQ-BOOK-009**: Patternbook SHALL enable similarity-based pattern retrieval.

### 3.10 Tool Integration

**REQ-TOOL-001**: System SHALL provide tool scripts organized by function.

**REQ-TOOL-002**: Tools SHALL execute in Lab environment.

**REQ-TOOL-003**: System SHALL provide direct tool access in HITL mode.

**REQ-TOOL-004**: System SHALL provide programmatic tool invocation in Autonomous mode.

**REQ-TOOL-005**: System SHALL log all tool invocations and outputs to Notebook.

**REQ-TOOL-006**: System SHALL handle tool execution failures gracefully.

**REQ-TOOL-007**: System SHALL validate tool outputs before using in hypotheses.

**REQ-TOOL-008**: Tools SHALL be versioned and documented.

### 3.11 Orchestration

**REQ-ORCH-001** (HITL): Human operator SHALL initiate and guide workflows via Cline interface.

**REQ-ORCH-002** (HITL): System SHALL provide workflow suggestions based on context.

**REQ-ORCH-003** (HITL): System SHALL support interactive debugging and inspection.

**REQ-ORCH-004** (HITL): System SHALL capture human decisions for autonomous learning.

**REQ-ORCH-005** (Autonomous): OpenHands SHALL execute workflows autonomously.

**REQ-ORCH-006** (Autonomous): System SHALL support parallel multi-target analysis.

**REQ-ORCH-007** (Autonomous): System SHALL provide checkpoints for human oversight.

**REQ-ORCH-008** (Autonomous): System SHALL resume from checkpoints after human intervention.

### 3.12 Reporting and Output

**REQ-RPT-001**: System SHALL generate analysis reports summarizing findings.

**REQ-RPT-002**: Reports SHALL include measurements, hypotheses, validation results, and conclusions.

**REQ-RPT-003**: System SHALL export validated grammars with test suites.

**REQ-RPT-004**: System SHALL maintain audit trail of all analysis decisions.

**REQ-RPT-005**: System SHALL generate reproducible workflow logs.

**REQ-RPT-006**: System SHALL provide visualizations for complex patterns and relationships.

**REQ-RPT-007**: Reports SHALL differentiate between HITL and Autonomous contributions.

---

## 4. Non-Functional Requirements

### 4.1 Performance

**REQ-PERF-001**: System SHALL process 1GB data files within 10 minutes.

**REQ-PERF-002**: Pattern analysis SHALL complete within 2 minutes for typical datasets (<100MB).

**REQ-PERF-003**: Autonomous mode SHALL support concurrent analysis of up to 5 targets.

**REQ-PERF-004**: Memory usage SHALL NOT exceed 16GB for single-target analysis.

**REQ-PERF-005**: HITL mode SHALL provide interactive response times (<2 seconds for tool invocation).

**REQ-PERF-006**: Pattern similarity search SHALL return results within 1 second.

### 4.2 Reliability

**REQ-REL-001**: System SHALL handle data parsing errors without crashing.

**REQ-REL-002**: System SHALL checkpoint progress for long-running analyses.

**REQ-REL-003**: System SHALL recover from agent or tool execution failures.

**REQ-REL-004**: Notebook writes SHALL be atomic to prevent corruption.

**REQ-REL-005**: System SHALL validate file integrity before processing.

**REQ-REL-006**: Autonomous mode SHALL implement error recovery and retry logic.

**REQ-REL-007**: System SHALL maintain state consistency across mode transitions.

### 4.3 Maintainability

**REQ-MAIN-001**: Code SHALL follow language-specific style guidelines (PEP 8 for Python).

**REQ-MAIN-002**: All agents and skills SHALL have unit tests achieving >80% coverage.

**REQ-MAIN-003**: System SHALL use type hints for all Python functions.

**REQ-MAIN-004**: Documentation SHALL be maintained in Markdown with code examples.

**REQ-MAIN-005**: Git commits SHALL follow Conventional Commits specification.

**REQ-MAIN-006**: Skills SHALL be modular and independently testable.

**REQ-MAIN-007**: Agents SHALL have clear contracts and interfaces.

### 4.4 Security

**REQ-SEC-001**: System SHALL execute analysis in network-isolated Docker containers.

**REQ-SEC-002**: Input data files SHALL be mounted read-only by default.

**REQ-SEC-003**: System SHALL NOT export sensitive data without authentication.

**REQ-SEC-004**: Validation tests SHALL NOT include exploit payloads.

**REQ-SEC-005**: System SHALL log all file system modifications.

**REQ-SEC-006**: Proprietary or sensitive grammars SHALL NOT be committed to public repository.

**REQ-SEC-007**: Local LLM SHALL NOT transmit data to external services.

**REQ-SEC-008**: System SHALL implement least-privilege access for agents.

### 4.5 Usability

**REQ-USE-001**: System SHALL provide clear error messages for common failure modes.

**REQ-USE-002**: Cookbook SHALL include step-by-step guides for typical workflows.

**REQ-USE-003**: Analysis reports SHALL be human-readable with visualizations.

**REQ-USE-004**: System SHALL support dry-run mode for testing workflows.

**REQ-USE-005**: HITL mode SHALL provide contextual suggestions and guidance.

**REQ-USE-006**: System SHALL provide progress indicators for long-running operations.

**REQ-USE-007**: Documentation SHALL include both HITL and Autonomous usage examples.

### 4.6 Portability

**REQ-PORT-001**: System SHALL be platform-independent (via Docker containerization).

**REQ-PORT-002**: Dependencies SHALL be managed via declarative configuration.

**REQ-PORT-003**: System SHALL NOT depend on proprietary tools.

**REQ-PORT-004**: Docker images SHALL be versioned and reproducible.

**REQ-PORT-005**: System SHALL support local LLM deployment (no cloud dependency).

### 4.7 Extensibility

**REQ-EXT-001**: System SHALL support adding new agent types.

**REQ-EXT-002**: System SHALL support adding new skills to library.

**REQ-EXT-003**: System SHALL support adding new tools to toolkit.

**REQ-EXT-004**: System SHALL support custom data format handlers.

**REQ-EXT-005**: Skills and tools SHALL follow plugin architecture for easy extension.

### 4.8 Compliance

**REQ-COMP-001**: System SHALL adhere to reShark Constitution v2.0.0.

**REQ-COMP-002**: All agents SHALL respect memory boundaries per Constitution.

**REQ-COMP-003**: Validation SHALL follow requirements in Constitution.

**REQ-COMP-004**: Open-core licensing SHALL follow Constitution.

**REQ-COMP-005**: Code reviews SHALL verify Constitutional compliance.

---

## 5. Data Requirements

### 5.1 Input Data

**REQ-DATA-IN-001**: Multiple input formats (PCAP/PCAPNG, binary files, logs, network dumps, text data).

**REQ-DATA-IN-002**: Optional: Analysis hints (e.g., "binary protocol", "state-based system").

**REQ-DATA-IN-003**: Optional: Reference samples for baseline comparison.

**REQ-DATA-IN-004**: Optional: Known constraints or domain knowledge.

### 5.2 Persistent Data

**REQ-DATA-PERS-001**: Notebook entries (JSON/Markdown) - session-specific working memory.

**REQ-DATA-PERS-002**: Rulebook grammars and schemas (Python/YAML/JSON) - validated models.

**REQ-DATA-PERS-003**: Cookbook procedures (Markdown) - procedural knowledge.

**REQ-DATA-PERS-004**: Patternbook embeddings (vector database) - similarity patterns.

**REQ-DATA-PERS-005**: Analysis archives (compressed JSON/logs) - historical sessions.

**REQ-DATA-PERS-006**: Skills library (Python/scripts) - reusable capabilities.

**REQ-DATA-PERS-007**: Tool configurations and metadata.

### 5.3 Output Data

**REQ-DATA-OUT-001**: Validated grammars and schemas (Python/YAML/JSON Schema).

**REQ-DATA-OUT-002**: Test suites (pytest, unittest).

**REQ-DATA-OUT-003**: Analysis reports (Markdown/HTML/PDF).

**REQ-DATA-OUT-004**: Workflow logs (JSON) - execution audit trail.

**REQ-DATA-OUT-005**: Statistical visualizations (PNG/SVG/interactive).

**REQ-DATA-OUT-006**: Extracted models and relationships (graphs, diagrams).

### 5.4 Temporary Data

**REQ-DATA-TEMP-001**: Session workspaces SHALL be isolated and timestamped.

**REQ-DATA-TEMP-002**: Intermediate processing artifacts SHALL be logged for debugging.

**REQ-DATA-TEMP-003**: Temporary data SHALL be cleanable without affecting persistent Books.

**REQ-DATA-TEMP-004**: Checkpoint data SHALL be preserved for session resumption.

---

## 6. Interface Requirements

### 6.1 User Interface

**REQ-UI-001** (HITL): Dev Container interface via VSCode.

**REQ-UI-002** (HITL): Cline AI assistant interface for interactive guidance.

**REQ-UI-003** (Autonomous): OpenHands web interface for execution monitoring.

**REQ-UI-004**: File-based configuration (YAML/JSON).

**REQ-UI-005**: Command-line interface for workflow initiation.

**REQ-UI-006**: Dashboard for Books visualization and navigation.

### 6.2 External Interfaces

**REQ-EXT-INT-001**: Local LLM API (Ollama, LM Studio, or compatible).

**REQ-EXT-INT-002**: Container runtime API for Lab management.

**REQ-EXT-INT-003**: File system API for Books access.

**REQ-EXT-INT-004**: Git API for version control integration.

**REQ-EXT-INT-005**: Vector database API for Patternbook (Qdrant or compatible).

### 6.3 Internal Interfaces

**REQ-INT-INT-001**: Agents communicate via shared Notebook state.

**REQ-INT-INT-002**: Skills expose standardized invocation interface.

**REQ-INT-INT-003**: Tools accept and return structured data formats.

**REQ-INT-INT-004**: Interpreters output conform to defined schemas.

**REQ-INT-INT-005**: Validation agents receive hypotheses in standardized format.

**REQ-INT-INT-006**: Orchestration provides workflow execution API.

**REQ-INT-INT-007**: Books provide query and update APIs.

### 6.4 Mode Transition Interface

**REQ-MODE-INT-001**: System SHALL provide API to transition from HITL to Autonomous mode.

**REQ-MODE-INT-002**: System SHALL provide API to transition from Autonomous to HITL mode.

**REQ-MODE-INT-003**: Context SHALL be preserved across mode transitions.

**REQ-MODE-INT-004**: Checkpoint data SHALL be accessible in both modes.

---

## 7. Constraints

### 7.1 Technical Constraints

**CONST-TECH-001**: System MUST have sufficient memory for concurrent data processing.

**CONST-TECH-002**: Python 3.11+ required.

**CONST-TECH-003**: Docker 24+ required.

**CONST-TECH-004**: Local LLM with minimum 7B parameters recommended.

**CONST-TECH-005**: Minimum 16GB RAM for HITL mode, 32GB for Autonomous mode.

**CONST-TECH-006**: GPU optional but recommended for LLM performance.

### 7.2 Operational Constraints

**CONST-OPS-001**: HITL mode MUST be functional before full Autonomous mode deployment.

**CONST-OPS-002**: Input data MUST NOT contain sensitive production information without sanitization.

**CONST-OPS-003**: Analysis results MUST be reviewed before Rulebook promotion.

**CONST-OPS-004**: Autonomous mode SHALL implement checkpoints for human oversight.

**CONST-OPS-005**: System SHALL operate without internet connectivity (airgapped capability).

### 7.3 Legal/Regulatory Constraints

**CONST-LEG-001**: System MUST comply with applicable data analysis and reverse engineering laws.

**CONST-LEG-002**: Proprietary or sensitive grammars MUST NOT be published without authorization.

**CONST-LEG-003**: System MUST NOT facilitate unauthorized system access.

**CONST-LEG-004**: Analysis SHALL respect intellectual property rights.

### 7.4 Learning Transfer Constraints

**CONST-LEARN-001**: Knowledge transfer from HITL to Autonomous SHALL be validated.

**CONST-LEARN-002**: Autonomous agents SHALL NOT exceed authority learned in HITL sessions.

**CONST-LEARN-003**: Failed autonomous attempts SHALL trigger HITL review.

---

## 8. Assumptions and Dependencies

### 8.1 Assumptions

**ASSUME-001**: Input data contains sufficient information for structural analysis.

**ASSUME-002**: Target systems exhibit consistent patterns across samples.

**ASSUME-003**: Sufficient data diversity exists for validation (minimum 3 samples).

**ASSUME-004**: Human operator has domain knowledge relevant to analysis target.

**ASSUME-005**: Local LLM has adequate capacity for analysis assistance.

**ASSUME-006**: HITL sessions provide sufficient training examples for autonomous learning.

### 8.2 Dependencies

**DEP-001**: Container runtime (Docker/Podman) for Lab containerization.

**DEP-002**: Local LLM (Ollama, LM Studio, or compatible).

**DEP-003**: Cline AI assistant for HITL mode.

**DEP-004**: OpenHands framework for Autonomous mode.

**DEP-005**: Analysis tools (tshark, scapy, numpy, scipy, pandas, etc.).

**DEP-006**: pytest for validation testing.

**DEP-007**: Vector database (Qdrant or compatible) for Patternbook.

**DEP-008**: Git for version control and knowledge tracking.

**DEP-009**: VSCode Dev Container support.

---

## 9. Success Criteria

### 9.1 HITL Mode Success Criteria

**SUCCESS-HITL-001**: Complete end-to-end analysis from raw data to validated Rulebook entry.

**SUCCESS-HITL-002**: Human operator successfully guided through analysis workflow by Cline.

**SUCCESS-HITL-003**: Pattern analysis generates actionable hypotheses.

**SUCCESS-HITL-004**: Validation correctly rejects invalid hypotheses.

**SUCCESS-HITL-005**: At least one validated grammar promotes to Rulebook.

**SUCCESS-HITL-006**: Workflow is documented in Cookbook for reuse.

**SUCCESS-HITL-007**: Memory boundaries are enforced (no Notebook data in Rulebook without validation).

**SUCCESS-HITL-008**: Skills learned are captured in library for autonomous reuse.

### 9.2 Autonomous Mode Success Criteria

**SUCCESS-AUTO-001**: Autonomous execution of learned workflows without errors.

**SUCCESS-AUTO-002**: System analyzes new targets using patterns from Books.

**SUCCESS-AUTO-003**: Patternbook suggestions improve hypothesis quality.

**SUCCESS-AUTO-004**: System scales to 5 concurrent target analyses.

**SUCCESS-AUTO-005**: Checkpoints enable effective human oversight.

**SUCCESS-AUTO-006**: Failed validations trigger appropriate HITL escalation.

**SUCCESS-AUTO-007**: Constitutional principles remain enforced under autonomous execution.

### 9.3 Knowledge Transfer Success Criteria

**SUCCESS-TRANSFER-001**: Skills learned in HITL are successfully applied in Autonomous mode.

**SUCCESS-TRANSFER-002**: Patterns captured in Patternbook improve analysis efficiency.

**SUCCESS-TRANSFER-003**: Cookbook procedures are reproducible in both modes.

**SUCCESS-TRANSFER-004**: Rulebook entries are equally valid regardless of mode that created them.

### 9.4 Integration Success Criteria

**SUCCESS-INTEG-001**: Lab provides consistent environment for both modes.

**SUCCESS-INTEG-002**: Books maintain consistency across mode transitions.

**SUCCESS-INTEG-003**: Tools function identically in both HITL and Autonomous modes.

**SUCCESS-INTEG-004**: Agents coordinate effectively via shared Notebook state.

**SUCCESS-INTEG-005**: Local LLM provides adequate assistance in both modes.

---

## 10. Appendices

### 10.1 Glossary

See Section 1.3 Definitions.

### 10.2 Architecture Components Summary

| Component | Location | Purpose | Used By |
|-----------|----------|---------|---------|
| Agents | `/agents` | Autonomous analysis components | Both modes |
| Skills | `/skills` | Reusable capabilities | Both modes |
| Tools | `/tools` | Scripts and utilities | Both modes |
| Books | `/books` | Knowledge storage | Both modes |
| Lab | `/labs` | Execution environment | Both modes |

### 10.3 Traceability Matrix

| Requirement Category | Constitution Section | Priority | Mode |
|---------------------|---------------------|----------|------|
| REQ-MODE-* | Section 10 (Implementation) | Critical | Both |
| REQ-PAT-* | Section 2 (Epistemic Law) | High | Both |
| REQ-BOOK-* | Section 3 (Memory) | Critical | Both |
| REQ-VAL-* | Section 4 (Validation) | Critical | Both |
| REQ-AGENT-* | Section 5 (Scope Boundaries) | High | Autonomous |
| REQ-SKILL-* | N/A | Medium | Both |
| REQ-SEC-* | Section 8 (Security) | High | Both |

### 10.4 Mode Comparison

| Aspect | HITL Mode | Autonomous Mode |
|--------|-----------|----------------|
| **Execution** | Human-guided | Self-directed |
| **Tools** | Dev Container + Cline | OpenHands |
| **LLM** | Local LLM (assistant) | Local LLM (orchestrator) |
| **Learning** | Knowledge acquisition | Pattern application |
| **Speed** | Interactive pace | Parallel execution |
| **Oversight** | Continuous | Checkpoint-based |
| **Use Case** | Exploration, training | Scaled analysis |

### 10.5 Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | 2026-01-11 | reShark Team | Initial SRS for Phase 1/2 |
| 2.0.0 | 2026-01-15 | reShark Team | Redraft for general RE framework with HITL/Autonomous modes |

---

**Approval**

This SRS is subject to approval and governed by the reShark Constitution v2.0.0.

**Next Steps**: Update System Implementation Technical Design (SITD) to reflect new architecture.