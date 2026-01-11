# reShark System Requirements Specification (SRS)

**Version**: 1.0.0  
**Date**: 2026-01-11  
**Status**: Draft  
**Governed By**: reShark Constitution v2.0.0

---

## 1. Introduction

### 1.1 Purpose

This document specifies the functional and non-functional requirements for reShark, an autonomous protocol reverse-engineering system. It serves as the authoritative requirements baseline for both Phase 1 (OpenHands-only) and Phase 2 (LangGraph orchestration).

### 1.2 Scope

reShark reconstructs undocumented network protocol specifications from PCAP files through:
- Statistical analysis of packet byte streams
- Hypothesis generation and validation
- Protocol grammar construction
- State machine inference

**Out of Scope**:
- Vulnerability discovery or exploitation
- Live traffic analysis
- Encrypted protocol breaking
- Payload semantics (focus is control-plane)

### 1.3 Definitions

- **PCAP**: Packet capture file containing network traffic
- **Scope**: Bounded agent role with specific analysis authority
- **Notebook**: Short-term working memory for hypotheses and measurements
- **Rulebook**: Validated protocol grammar repository
- **Patternbook**: (Phase 2 only) Vector similarity memory for analogies
- **Cookbook**: Procedural knowledge for analysis methods
- **Golden Session**: Reference PCAP used for initial model construction
- **Control-Plane**: Protocol state, message types, and structure (vs. payload data)

### 1.4 References

- reShark Constitution v2.0.0
- OpenHands Documentation
- LangGraph Documentation (Phase 2)
- Wireshark/tshark User Guides
- Scapy Documentation

---

## 2. System Overview

### 2.1 System Context

```
┌─────────────┐
│   Human     │
│  Operator   │
└──────┬──────┘
       │
       │ Initiates Analysis
       │ Reviews Results
       ↓
┌──────────────────────────────────────────────┐
│           reShark System                │
│                                              │
│  ┌────────────┐  ┌──────────────┐          │
│  │ OpenHands  │  │  LangGraph   │ (Phase 2)│
│  │ (Executor) │  │ (Orchestrator│          │
│  └─────┬──────┘  └───────┬──────┘          │
│        │                  │                  │
│  ┌─────▼──────────────────▼─────┐          │
│  │        Scopes                 │          │
│  │  Observer | Theorist |        │          │
│  │  Validator | Archivist        │          │
│  └─────┬──────────────────┬──────┘          │
│        │                  │                  │
│  ┌─────▼──────┐    ┌─────▼──────┐          │
│  │   Tools    │    │   Memory   │          │
│  │ tshark     │    │ Notebook   │          │
│  │ scapy      │    │ Rulebook   │          │
│  │ numpy      │    │ Cookbook   │          │
│  └────────────┘    │ Patternbook│ (Phase 2)│
│                    └────────────┘          │
└──────────────────────────────────────────────┘
       │
       │ Reads/Writes
       ↓
┌──────────────┐
│  File System │
│  - PCAPs     │
│  - Grammars  │
│  - Logs      │
└──────────────┘
```

### 2.2 System Architecture (Phase 1)

**Three-Pillar Model**:

1. **Storage (File System)**
   - PCAP repository
   - Notebook (JSON/Markdown)
   - Rulebook (Python/YAML grammars)
   - Cookbook (Markdown documentation)
   - Analysis logs and archives

2. **Execution (OpenHands + Docker)**
   - Sandboxed tool execution
   - File change auditing
   - Reproducible environments
   - Tool isolation

3. **Intelligence (Scopes)**
   - Observer: Measures and detects patterns
   - Theorist: Generates hypotheses
   - Validator: Tests and falsifies
   - Archivist: Manages memory

### 2.3 System Architecture (Phase 2 Additions)

- LangGraph orchestrates Scope execution
- Patternbook (Qdrant) adds vector similarity
- Reduced human operator involvement
- Multi-protocol parallel analysis

---

## 3. Functional Requirements

### 3.1 PCAP Processing

**REQ-PCAP-001**: System SHALL accept PCAP and PCAPNG format files as input.

**REQ-PCAP-002**: System SHALL extract bidirectional TCP/UDP streams from PCAPs.

**REQ-PCAP-003**: System SHALL preserve packet timestamps and flow direction metadata.

**REQ-PCAP-004**: System SHALL support PCAP files up to 10GB in size.

**REQ-PCAP-005**: System SHALL detect and report truncated or corrupted PCAP files.

### 3.2 Statistical Analysis (Observer Scope)

**REQ-STAT-001**: System SHALL compute byte-level entropy for message streams.

**REQ-STAT-002**: System SHALL calculate byte frequency distributions across messages.

**REQ-STAT-003**: System SHALL detect byte position invariants (fixed values at specific offsets).

**REQ-STAT-004**: System SHALL identify low-entropy control bytes (verbs).

**REQ-STAT-005**: System SHALL compute correlation matrices for byte sequences.

**REQ-STAT-006**: System SHALL detect message length patterns and distributions.

**REQ-STAT-007**: System SHALL output all statistical measurements to Notebook with timestamps.

### 3.3 Hypothesis Generation (Theorist Scope)

**REQ-HYPO-001**: System SHALL propose message boundary hypotheses based on statistical anchors.

**REQ-HYPO-002**: System SHALL identify candidate message type fields (control bytes).

**REQ-HYPO-003**: System SHALL infer field boundaries using entropy and variance patterns.

**REQ-HYPO-004**: System SHALL propose state transition models based on message sequencing.

**REQ-HYPO-005**: System SHALL document reasoning and evidence for each hypothesis in Notebook.

**REQ-HYPO-006**: System SHALL NOT propose hypotheses that contradict Epistemic Law (Constitution Section 2).

### 3.4 Validation (Validator Scope)

**REQ-VAL-001**: System SHALL create executable test suites for protocol hypotheses.

**REQ-VAL-002**: System SHALL test hypotheses against minimum 3 distinct PCAPs (cross-PCAP validation).

**REQ-VAL-003**: System SHALL perform negative testing with corrupted/truncated PCAPs.

**REQ-VAL-004**: System SHALL test message boundary detection on edge cases (min/max sizes, fragmentation).

**REQ-VAL-005**: System SHALL log all validation attempts with pass/fail outcomes to Notebook.

**REQ-VAL-006**: System SHALL prevent Rulebook promotion for hypotheses that fail validation.

**REQ-VAL-007**: System SHALL record failed hypotheses to prevent re-investigation.

### 3.5 Grammar Construction

**REQ-GRAM-001**: System SHALL generate executable protocol grammars in Python.

**REQ-GRAM-002**: Grammars SHALL define message boundaries, types, and field structures.

**REQ-GRAM-003**: Grammars SHALL include parsing functions and validation logic.

**REQ-GRAM-004**: Grammars SHALL be versioned and timestamped.

**REQ-GRAM-005**: Grammars SHALL reference source PCAPs and validation evidence.

### 3.6 Memory Management (Archivist Scope)

**REQ-MEM-001**: System SHALL maintain Notebook as JSON/Markdown in timestamped workspace.

**REQ-MEM-002**: System SHALL store Rulebook entries as versioned Python/YAML files.

**REQ-MEM-003**: System SHALL enforce memory boundaries per Constitution Section 3.

**REQ-MEM-004**: System SHALL provide promotion workflow from Notebook to Rulebook.

**REQ-MEM-005**: System SHALL archive completed analysis sessions.

**REQ-MEM-006**: System SHALL support Rulebook rollback if evidence contradicts promoted entry.

**REQ-MEM-007**: (Phase 2) System SHALL maintain Patternbook as Qdrant vector database.

### 3.7 Tool Integration

**REQ-TOOL-001**: System SHALL expose `tshark`, `scapy`, `numpy`, `scipy`, `pandas` via MCP.

**REQ-TOOL-002**: System SHALL execute tools in Docker-isolated environment.

**REQ-TOOL-003**: System SHALL mount PCAPs as read-only unless write authority granted.

**REQ-TOOL-004**: System SHALL log all tool invocations and outputs to Notebook.

**REQ-TOOL-005**: System SHALL handle tool execution failures gracefully.

**REQ-TOOL-006**: System SHALL validate tool outputs before using in hypotheses.

### 3.8 Orchestration

**REQ-ORCH-001** (Phase 1): Human operator SHALL initiate analysis workflows via scripts.

**REQ-ORCH-002** (Phase 1): System SHALL provide workflow templates in Cookbook.

**REQ-ORCH-003** (Phase 1): Scopes SHALL be invoked as Python modules with explicit contracts.

**REQ-ORCH-004** (Phase 2): LangGraph SHALL orchestrate Scope execution.

**REQ-ORCH-005** (Phase 2): System SHALL support autonomous multi-PCAP campaigns.

**REQ-ORCH-006** (Phase 2): Human operator SHALL provide oversight via checkpoints.

### 3.9 Reporting and Output

**REQ-RPT-001**: System SHALL generate analysis reports summarizing findings.

**REQ-RPT-002**: Reports SHALL include statistical measurements, hypotheses, and validation results.

**REQ-RPT-003**: System SHALL export validated grammars with test suites.

**REQ-RPT-004**: System SHALL maintain audit trail of all analysis decisions.

**REQ-RPT-005**: System SHALL generate reproducible workflow logs.

---

## 4. Non-Functional Requirements

### 4.1 Performance

**REQ-PERF-001**: System SHALL process 1GB PCAP files within 10 minutes.

**REQ-PERF-002**: Statistical analysis SHALL complete within 2 minutes for typical PCAP (<100MB).

**REQ-PERF-003**: System SHALL support concurrent analysis of up to 5 protocols (Phase 2).

**REQ-PERF-004**: Memory usage SHALL NOT exceed 16GB for single-protocol analysis.

### 4.2 Reliability

**REQ-REL-001**: System SHALL handle PCAP parsing errors without crashing.

**REQ-REL-002**: System SHALL checkpoint progress for long-running analyses.

**REQ-REL-003**: System SHALL recover from tool execution failures.

**REQ-REL-004**: Notebook writes SHALL be atomic to prevent corruption.

**REQ-REL-005**: System SHALL validate file integrity before processing.

### 4.3 Maintainability

**REQ-MAIN-001**: Code SHALL follow PEP 8 style guidelines (Python).

**REQ-MAIN-002**: All Scopes SHALL have unit tests achieving >80% coverage.

**REQ-MAIN-003**: System SHALL use type hints for all Python functions.

**REQ-MAIN-004**: Documentation SHALL be maintained in Markdown with code examples.

**REQ-MAIN-005**: Git commits SHALL follow Conventional Commits specification.

### 4.4 Security

**REQ-SEC-001**: System SHALL execute analysis in network-isolated Docker containers.

**REQ-SEC-002**: PCAP files SHALL be mounted read-only by default.

**REQ-SEC-003**: System SHALL NOT export Rulebook grammars without authentication.

**REQ-SEC-004**: Validation tests SHALL NOT include exploit payloads.

**REQ-SEC-005**: System SHALL log all file system modifications.

**REQ-SEC-006**: Grammars for proprietary protocols SHALL NOT be committed to public repository.

### 4.5 Usability

**REQ-USE-001**: System SHALL provide clear error messages for common failure modes.

**REQ-USE-002**: Workflow templates SHALL include examples for typical use cases.

**REQ-USE-003**: Analysis reports SHALL be human-readable with visualizations.

**REQ-USE-004**: System SHALL support dry-run mode for testing workflows.

**REQ-USE-005**: Cookbook SHALL provide step-by-step guides for new users.

### 4.6 Portability

**REQ-PORT-001**: System SHALL be platform-independent (via containerization).

**REQ-PORT-002**: Dependencies SHALL be managed via Poetry (Python) and Docker Compose.

**REQ-PORT-003**: System SHALL NOT depend on proprietary tools.

**REQ-PORT-004**: Docker images SHALL be versioned and published to registry.

### 4.7 Compliance

**REQ-COMP-001**: System SHALL adhere to reShark Constitution v2.0.0.

**REQ-COMP-002**: All Scopes SHALL respect memory boundaries (Constitution Section 3).

**REQ-COMP-003**: Validation SHALL follow requirements in Constitution Section 4.

**REQ-COMP-004**: Open-core licensing SHALL follow Constitution Section 7.

**REQ-COMP-005**: Code reviews SHALL verify Constitutional compliance.

---

## 5. Data Requirements

### 5.1 Input Data

**REQ-DATA-IN-001**: PCAPs in libpcap or pcapng format.

**REQ-DATA-IN-002**: Optional: Protocol hints (e.g., "binary RPC protocol").

**REQ-DATA-IN-003**: Optional: Golden Session designation for reference PCAP.

### 5.2 Persistent Data

**REQ-DATA-PERS-001**: Notebook entries (JSON/Markdown).

**REQ-DATA-PERS-002**: Rulebook grammars (Python/YAML).

**REQ-DATA-PERS-003**: Cookbook procedures (Markdown).

**REQ-DATA-PERS-004**: Analysis archives (compressed JSON/logs).

**REQ-DATA-PERS-005** (Phase 2): Patternbook embeddings (Qdrant).

### 5.3 Output Data

**REQ-DATA-OUT-001**: Validated protocol grammars (Python).

**REQ-DATA-OUT-002**: Test suites (pytest).

**REQ-DATA-OUT-003**: Analysis reports (Markdown/HTML).

**REQ-DATA-OUT-004**: Workflow logs (JSON).

**REQ-DATA-OUT-005**: Statistical visualizations (PNG/SVG).

---

## 6. Interface Requirements

### 6.1 User Interface

**REQ-UI-001** (Phase 1): Command-line interface for workflow initiation.

**REQ-UI-002**: OpenHands web interface for execution monitoring.

**REQ-UI-003**: File-based configuration (YAML).

**REQ-UI-004** (Phase 2): LangGraph visualization dashboard.

### 6.2 External Interfaces

**REQ-EXT-001**: MCP servers expose tools to Scopes.

**REQ-EXT-002**: File system API for Notebook/Rulebook/Cookbook access.

**REQ-EXT-003**: Docker CLI for container management.

**REQ-EXT-004** (Phase 2): Qdrant HTTP API for Patternbook.

### 6.3 Internal Interfaces

**REQ-INT-001**: Scopes communicate via shared Notebook state.

**REQ-INT-002**: Observer outputs conform to defined JSON schema.

**REQ-INT-003**: Theorist accepts Observer outputs as structured input.

**REQ-INT-004**: Validator receives hypotheses in standardized format.

**REQ-INT-005**: Archivist provides promotion API for Rulebook updates.

---

## 7. Constraints

### 7.1 Technical Constraints

**CONST-TECH-001**: System MUST have sufficient memory for concurrent PCAP processing.

**CONST-TECH-002**: Python 3.11+ required.

**CONST-TECH-003**: Docker 24+ required.

**CONST-TECH-004**: OpenHands version compatibility TBD during implementation.

### 7.2 Operational Constraints

**CONST-OPS-001**: Phase 1 MUST complete before Phase 2 begins.

**CONST-OPS-002**: PCAPs MUST NOT contain live production traffic without sanitization.

**CONST-OPS-003**: Analysis results MUST be reviewed by human operator before Rulebook promotion.

### 7.3 Legal/Regulatory Constraints

**CONST-LEG-001**: System MUST comply with applicable network traffic analysis laws.

**CONST-LEG-002**: Proprietary protocol grammars MUST NOT be published without authorization.

**CONST-LEG-003**: System MUST NOT facilitate unauthorized access to systems.

---

## 8. Assumptions and Dependencies

### 8.1 Assumptions

**ASSUME-001**: Input PCAPs contain complete protocol handshakes.

**ASSUME-002**: Protocols are binary and structured (not plaintext/HTTP).

**ASSUME-003**: Sufficient PCAP diversity exists for validation (minimum 3 captures).

**ASSUME-004**: Human operator has basic networking knowledge.

**ASSUME-005**: Golden Session represents typical protocol behavior.

### 8.2 Dependencies

**DEP-001**: OpenHands framework for execution environment.

**DEP-002**: Docker for container orchestration.

**DEP-003**: tshark, Zeek for PCAP dissection.

**DEP-004**: scapy for PCAP manipulation.

**DEP-005**: numpy, scipy, pandas for statistical analysis.

**DEP-006**: pytest for validation testing.

**DEP-007** (Phase 2): LangGraph for orchestration.

**DEP-008** (Phase 2): Qdrant for vector similarity.

---

## 9. Success Criteria

### 9.1 Phase 1 Success Criteria

**SUCCESS-P1-001**: Complete end-to-end protocol analysis from PCAP to validated Rulebook entry.

**SUCCESS-P1-002**: Observer generates statistical measurements for synthetic protocol.

**SUCCESS-P1-003**: Theorist proposes testable hypotheses based on observations.

**SUCCESS-P1-004**: Validator correctly rejects invalid hypotheses.

**SUCCESS-P1-005**: At least one validated grammar promotes to Rulebook.

**SUCCESS-P1-006**: Workflow is reproducible via documented procedures.

**SUCCESS-P1-007**: Memory boundaries are enforced (no Notebook data in Rulebook).

### 9.2 Phase 2 Success Criteria

**SUCCESS-P2-001**: LangGraph orchestrates Scope execution autonomously.

**SUCCESS-P2-002**: System analyzes new protocol with minimal human intervention.

**SUCCESS-P2-003**: Patternbook suggestions improve hypothesis quality (measured by validation pass rate).

**SUCCESS-P2-004**: System scales to 5 concurrent protocol analyses.

**SUCCESS-P2-005**: Constitutional principles remain enforced under LangGraph orchestration.

---

## 10. Appendices

### 10.1 Glossary

See Section 1.3 Definitions.

### 10.2 Traceability Matrix

| Requirement ID | Constitution Section | Priority |
|----------------|---------------------|----------|
| REQ-STAT-* | Section 2 (Epistemic Law) | High |
| REQ-MEM-* | Section 3 (Memory Constitution) | Critical |
| REQ-VAL-* | Section 4 (Validation) | Critical |
| REQ-ORCH-* | Section 10 (Phased Implementation) | Medium |
| REQ-SEC-* | Section 8 (Security Safeguards) | High |

### 10.3 Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | 2026-01-11 | reShark Team | Initial SRS for Phase 1/2 |

---

**Approval**

This SRS is subject to approval and governed by the reShark Constitution v2.0.0.

**Next Steps**: Proceed to System Implementation Technical Design (SITD).