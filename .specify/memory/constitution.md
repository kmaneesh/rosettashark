<!--
SYNC IMPACT REPORT: Constitution v2.0.0
================================================================================
Version Change: 1.0.0 → 2.0.0 (phased implementation)
Ratification Date: 2026-01-11

Major Changes:
- NEW: Section 10 - Phased Implementation Strategy (Phase 1: Manual orchestration, Phase 2: Autonomous orchestration)
- MODIFIED: Section 5 - Scopes (adapted for phased rollout)
- MODIFIED: Section 6 - Tools and Capabilities (execution environment clarifications)
- MODIFIED: Architecture references throughout to support phased approach

Sections Unchanged:
- Mission and Scope (Section 1)
- Epistemic Law (Section 2)
- Memory Constitution (Section 3)
- Validation and Falsification (Section 4)
- Open-Core Boundary (Section 7)
- Security and Dual-Use Safeguards (Section 8)
- Governance and Evolution (Section 9)

Templates Requiring Updates:
- ✅ Phase 1 implementation can begin with current section structure
- ⚠️ Phase 2 will require workflow automation definitions
- ✅ Notebook/Rulebook/Patternbook memory model remains consistent across phases

Follow-up TODOs:
- Create System Requirements Specification (SRS)
- Create System Implementation Technical Design (SITD)
- Create Implementation Plan for Phase 1

Commit Message:
docs: ratify reShark constitution v2.0.0 (phased implementation strategy)
================================================================================
-->

# The reShark Constitution

## 1. Mission and Scope

reShark is a system for *understanding* network protocols through rigorous analysis of captured network traffic. It exists to reconstruct protocol specifications from empirical evidence, not to exploit, weaponize, or reverse engineer protocols for malicious purposes.

**What reShark Does**:
- Infers protocol structure, message boundaries, field semantics, and state machines from PCAP files
- Applies statistical analysis, entropy measurement, and hypothesis testing to protocol data
- Validates protocol models through cross-PCAP testing and negative case analysis
- Produces executable, reproducible protocol grammars that can be independently verified

**What reShark Does Not Do**:
- Attack, exploit, or weaponize protocols
- Provide automated vulnerability discovery
- Generate exploits or payloads
- Operate on live network traffic without explicit PCAP intermediation
- Make claims without empirical validation

**Intended Users**:
- Security researchers investigating undocumented protocols
- Network engineers reverse-engineering proprietary systems
- Defensive teams building protocol parsers and anomaly detectors
- Academic researchers studying network protocol design patterns

## 2. Epistemic Law (Truth and Evidence)

reShark recognizes only *evidence-based knowledge*. A protocol claim is knowledge if and only if it meets the following criteria:

**Required Properties**:
- **Testable**: The claim MUST be falsifiable through PCAP analysis
- **Reproducible**: The claim MUST produce consistent results across multiple PCAP files containing the protocol
- **Traceable**: The claim MUST be traceable to specific byte sequences, statistical measurements, or behavioral observations in source PCAPs

**Prohibited Sources of Knowledge**:
- Model intuition, hunches, or "feels like" reasoning without empirical support
- Pattern similarity or vector proximity without validation
- Analogy to other protocols without explicit testing
- Human assumptions not grounded in observed traffic

**Burden of Proof**:
- The system bears the burden of proof for all claims
- Absence of evidence is not evidence of structure
- Uncertainty MUST be quantified and recorded
- Failure to validate a hypothesis is a valid outcome and MUST be logged

## 3. Memory Constitution

reShark maintains three distinct memory systems with non-overlapping responsibilities:

### 3.1 Notebook (Short-Term Working Memory)

**Purpose**: Transient storage for hypotheses, intermediate statistics, and experimental results.

**Allowed Contents**:
- Active hypotheses under investigation
- Statistical measurements (entropy, frequency distributions, correlation matrices)
- Partial field boundaries and candidate control bytes
- Validation attempts and their outcomes (success or failure)
- Agent reasoning traces and decision logs

**Forbidden Contents**:
- Validated protocol grammars (belong in Rulebook)
- Similarity patterns or embeddings (belong in Patternbook)
- Long-term historical data (must be archived or promoted)

**Retention**: Notebook entries are ephemeral and MUST be cleared, archived, or promoted after analysis completion.

**Implementation**: Notebook is implemented as JSON or Markdown files in a designated workspace directory with timestamp-based organization.

### 3.2 Rulebook (Validated Protocol Truth)

**Purpose**: Authoritative storage for validated, executable protocol specifications.

**Allowed Contents**:
- Protocol grammars that have passed validation
- Message boundary rules
- Field definitions with byte ranges and semantic interpretations
- State machine definitions
- Validation test suites that confirmed the grammar

**Forbidden Contents**:
- Unvalidated hypotheses (remain in Notebook until proven)
- Statistical measurements (supporting data lives in archives)
- Procedural instructions or tool usage guides (belong in Cookbook)
- Vector embeddings or similarity metrics (belong in Patternbook)

**Promotion Rules**:
- A hypothesis MAY be promoted from Notebook to Rulebook only after passing validation requirements defined in Section 4
- Promotion MUST include the validation evidence and test suite
- Promotion MUST be reversible if later evidence contradicts the rule

**Implementation**: Rulebook entries are stored as versioned Python/YAML grammar definitions with accompanying pytest test suites.

### 3.3 Patternbook (Similarity and Analogy Memory)

**Purpose**: Non-authoritative storage for protocol similarities, analogies, and exploration hints.

**Allowed Contents**:
- Vector embeddings of protocol behaviors
- Similarity scores between protocol features
- Clustering results and protocol family groupings
- Suggested analogies for new protocol analysis

**Forbidden Contents**:
- Protocol grammars or specifications (belong in Rulebook)
- Claims of protocol equivalence without validation
- Raw PCAP data or byte sequences (processed only)

**Authority Level**: Patternbook entries are *advisory only* and MUST NOT be treated as ground truth. Any pattern-derived hypothesis MUST be validated before promotion to Rulebook.

**Implementation** (Phase 2 only): Patternbook uses Qdrant or similar vector database for similarity search. Phase 1 does not implement Patternbook.

### 3.4 Cookbook (Procedural Knowledge)

**Purpose**: Reusable analysis procedures, statistical methods, and measurement techniques.

**Allowed Contents**:
- Analysis recipes and workflows
- Statistical method descriptions
- Alignment strategies
- Validation patterns
- Tool usage templates

**Forbidden Contents**:
- Protocol-specific grammars (belong in Rulebook)
- Active hypotheses (belong in Notebook)
- Tool outputs or measurements (belong in Notebook)

**Implementation**: Cookbook is stored as Markdown documentation with executable code examples in the repository.

## 4. Validation and Falsification

All protocol claims MUST undergo explicit validation before acceptance into the Rulebook.

**Minimum Validation Standards**:
- **Cross-PCAP Testing**: The protocol model MUST parse at least three distinct PCAP files containing the protocol without error
- **Negative Testing**: The model MUST reject or flag malformed traffic when presented with corrupted or partial captures
- **Boundary Testing**: Message boundary detection MUST correctly identify edge cases (minimum/maximum message sizes, fragmentation)
- **Failure Logging**: All validation failures MUST be recorded in the Notebook with failure mode analysis

**Validation Workflow**:
1. Hypothesis generated in Notebook
2. Test suite created specifying expected parsing outcomes
3. Model executed against validation PCAPs
4. Results compared to test suite
5. Discrepancies investigated and logged
6. If validation passes: promote to Rulebook with evidence
7. If validation fails: log failure, refine hypothesis, repeat

**Falsification as Success**:
- Disproving a hypothesis is a successful outcome
- Negative results MUST be logged to prevent repeated investigation of dead ends
- Failed hypotheses contribute to understanding protocol space

## 5. Scopes (Agent Authority)

reShark operates through *Scopes*: licensed agent roles with bounded authority over specific analysis domains.

**Scope Model**:
- Scopes are defined in separate YAML files in the `/scopes` directory
- Each Scope has a declared scope of authority (e.g., entropy analysis, field extraction, state machine inference)
- Scopes MAY read from any memory system
- Scopes MAY write only to Notebook unless explicitly granted Rulebook promotion authority
- Scopes MUST declare their reasoning and evidence when proposing claims

**Required Scope Behaviors**:
- **Authority Compliance**: Scopes MUST NOT exceed their declared authority domain
- **Evidence Recording**: All observations and reasoning MUST be written to shared state (Notebook)
- **Memory Boundaries**: Scopes MUST respect the Memory Constitution (Section 3)
- **No Hidden State**: Scopes MUST NOT maintain private memory or assumptions invisible to other system components

**Prohibited Behaviors**:
- Silent assumptions or implicit background knowledge
- Accessing tool outputs without recording them in Notebook
- Promoting hypotheses to Rulebook without validation authority
- Making claims outside declared expertise area

**Phase-Specific Implementation**:
- **Phase 1**: Scopes are implemented as modules with explicit contracts; orchestration is manual/script-based
- **Phase 2**: Scopes become workflow nodes with state transitions managed by the orchestration framework

**Core Scopes**:
1. **Observer**: Entropy measurement, byte frequency analysis, structural pattern detection
2. **Theorist**: Hypothesis generation, model proposal, analogical reasoning
3. **Validator**: Test suite creation, cross-PCAP validation, negative case testing
4. **Archivist**: Notebook management, Rulebook promotion, memory organization

## 6. Tools and Capabilities

Tools are execution capabilities, not sources of knowledge. Tool outputs are raw observations that MUST be interpreted and validated by Scopes.

**Tool Classification**:
- **Analysis Tools**: PCAP parsing and dissection tools
- **Statistical Tools**: Mathematical analysis libraries
- **Manipulation Tools**: PCAP generation and modification tools
- **Orchestration Tools**: Execution environment and workflow orchestration frameworks

**Tool Governance**:
- Tool usage instructions and invocation patterns belong in the Cookbook (procedural documentation), not Rulebook
- Tool outputs MUST be recorded in Notebook for traceability
- Tools MUST NOT autonomously write to Rulebook
- Tool failures MUST be logged and investigated

**Execution Environment**:
- Tools execute in sandboxed containers with network isolation
- PCAPs are mounted read-only unless explicit write authority granted
- Tool configurations MUST be versioned and reproducible
- All file changes MUST be auditable

**MCP (Model Context Protocol) Integration**:
- Analysis tools are exposed via MCP servers as callable functions
- MCP provides tool schema definitions and parameter validation
- Tool knowledge is stored in MCP server configurations, not in Rulebook
- Phase 1 uses basic MCP for tool exposure; Phase 2 may add custom MCP servers

## 7. Open-Core Boundary

reShark follows an open-core model with clear separation between infrastructure and knowledge:

**Open-Source Components** (public repository):
- Analysis tooling and orchestration framework
- Scope implementations and agent logic
- Statistical analysis libraries and validation frameworks
- Synthetic protocol generators for demonstration and testing
- Toy protocol grammars used in documentation and examples
- Execution environment configurations
- Cookbook (procedural knowledge)
- This Constitution

**Restricted Components** (not in public repository):
- Real-world proprietary protocol grammars extracted from production traffic
- Protocol-specific Rulebook entries for commercial or sensitive systems
- Customer-specific PCAP analysis results
- Grammars derived from protocols with active security implications

**Disclosure Policy**:
- Synthetic protocols MAY be published freely
- Open-standard protocols (e.g., HTTP, DNS) MAY be published as examples
- Proprietary protocols MUST NOT be published without explicit authorization from the protocol owner
- Dual-use protocols with security implications require case-by-case review

## 8. Security and Dual-Use Safeguards

reShark acknowledges that protocol analysis is dual-use technology. The following safeguards apply:

**Technical Restraints**:
- No automated export of Rulebook contents to external systems
- No bulk protocol grammar exfiltration APIs
- Grammar access requires explicit authentication and audit logging
- Validation tests MUST NOT include exploitation payloads

**Operational Safeguards**:
- Protocol grammars are knowledge, not weapons; they do not enable attacks without additional weaponization effort
- Users are responsible for lawful use of extracted protocol knowledge
- reShark does not implement or suggest vulnerability exploitation workflows

**Reporting and Disclosure**:
- If protocol analysis reveals security vulnerabilities, users MUST follow responsible disclosure practices
- reShark does not provide vulnerability reporting infrastructure

## 9. Governance and Evolution

This Constitution governs all reShark operations and supersedes other procedural documentation.

**Amendment Procedure**:
- Amendments MUST be proposed in writing with rationale and impact analysis
- Amendments to Sections 2 (Epistemic Law) and 3 (Memory Constitution) require conservative review; correctness outranks convenience
- Amendments MUST increment the version number according to semantic versioning:
  - **MAJOR**: Breaking changes to epistemic rules or memory boundaries
  - **MINOR**: New sections, expanded governance, or Scope model changes
  - **PATCH**: Clarifications, wording improvements, non-semantic fixes

**Version Control**:
- All Constitution versions MUST be retained in repository history
- Current version displayed at document end with ratification and amendment dates

**Compliance Review**:
- All code changes, Scope implementations, and tool integrations MUST be reviewed for Constitutional compliance
- Violations MUST be documented and justified if unavoidable (technical debt)
- Complexity that violates principles MUST be explicitly justified and time-bound

**Primacy**:
- Correctness > Speed
- Evidence > Intuition
- Falsifiability > Convenience
- Transparency > Efficiency

## 10. Phased Implementation Strategy

reShark development proceeds in two distinct phases to manage complexity and prove core concepts before adding orchestration overhead.

### 10.1 Phase 1: Foundation (Manual Orchestration)

**Scope**: Build and validate core analysis capabilities with manual orchestration.

**Goals**:
- Establish sandboxed execution environment
- Implement Notebook, Rulebook, and Cookbook (Patternbook deferred)
- Build Observer, Theorist, Validator, and Archivist Scopes as modules
- Demonstrate successful protocol reconstruction on synthetic protocols
- Validate epistemic framework through real analysis workflows

**Architecture**:
- Execution environment provides sandbox and file auditing
- Scripts coordinate Scope execution
- Human operator initiates workflows and reviews results
- Orchestration is script-based or manual
- Tool access layer provides standardized interfaces

**Deliverables**:
- Working sandboxed execution environment
- Functional Notebook/Rulebook/Cookbook file structures
- Operational Observer, Theorist, Validator, Archivist modules
- At least one complete protocol analysis from PCAP to validated Rulebook entry
- Documentation of workflow patterns discovered

**Success Criteria**:
- Protocol reconstruction workflow executes end-to-end
- Validation framework catches invalid hypotheses
- Memory boundaries are respected
- Tool outputs are properly recorded in Notebook
- Rulebook contains at least one validated grammar

### 10.2 Phase 2: Autonomous Orchestration

**Scope**: Add autonomous orchestration and pattern memory.

**Goals**:
- Implement graph-based workflow orchestration
- Add Patternbook (vector memory) for analogical reasoning
- Enable autonomous multi-PCAP analysis campaigns
- Reduce human operator involvement to oversight
- Scale to complex multi-protocol scenarios

**Architecture**:
- Workflow graph manages Scope sequencing and state transitions
- Sandboxed execution environment remains core infrastructure
- Scopes become workflow nodes with defined edges
- Patternbook (vector database) enables cross-protocol learning
- Tool access layer may be extended with custom protocol analysis servers

**Deliverables**:
- Graph-based orchestration framework
- Patternbook implementation with vector similarity search
- Autonomous workflow execution with human checkpoints
- Multi-protocol analysis campaigns
- Performance metrics and optimization documentation

**Success Criteria**:
- System analyzes new protocols with minimal human guidance
- Orchestration framework enforces Scope boundaries and state transitions
- Patternbook suggestions improve hypothesis quality
- System scales to 5+ concurrent protocol analyses
- Validation rigor is maintained

**Phase Boundary**:
- Phase 1 MUST achieve all success criteria before Phase 2 begins
- Phase 1 code MUST NOT be discarded; Phase 2 wraps it
- Constitutional principles MUST remain unchanged across phases
- Phase 2 MAY add new Scopes but MUST NOT violate existing Scope contracts

---

**Version**: 2.0.0  
**Ratified**: 2026-01-11  
**Last Amended**: 2026-01-11  
**Supersedes**: Constitution v1.0.0