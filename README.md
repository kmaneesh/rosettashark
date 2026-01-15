
# 🦈 reShark  
General Purpose Reverse Engineering Framework

**Version**: 2.0.0  
**Status**: Active Development  
**Governed By**: reShark Constitution v2.0.0

---

## Overview

reShark is a hybrid reverse engineering framework that combines human expertise with autonomous analysis. Initially focused on network protocol analysis from PCAP files, it provides a general-purpose system for discovering structure and behavior in undocumented systems.

### Dual-Mode Operation

**Human-in-the-Loop (HITL) Mode**  
- Interactive analysis with AI assistance (Cline + Local LLM)
- Knowledge capture and skill development
- Dev Container environment for direct exploration
- Teaches the system through guided workflows

**Autonomous Mode**  
- Self-directed execution using learned patterns
- OpenHands + Local LLM orchestration
- Applies captured knowledge at scale
- Parallel multi-target analysis

---

## Architecture

```
┌─────────────────────────────────────────────┐
│            reShark Framework                │
├─────────────────┬───────────────────────────┤
│   HITL Mode     │    Autonomous Mode       │
│   (Teaching)    │    (Applying)            │
└────────┬────────┴────────┬──────────────────┘
         │                 │
         └────────┬─────────┘
                  │
    ┌─────────────┴──────────────┐
    │        Components          │
    ├────────────────────────────┤
    │ • Agents (interpreters,    │
    │   orchestration,           │
    │   validation, scopes)      │
    │ • Skills (reusable         │
    │   capabilities)            │
    │ • Tools (scripts &         │
    │   utilities)               │
    │ • Books (knowledge         │
    │   storage)                 │
    │ • Labs (execution envs)    │
    └────────────────────────────┘
```

---

## Core Components

### 📚 Books (Knowledge Storage)
- **Notebook**: Session-specific working memory
- **Rulebook**: Validated models and grammars
- **Cookbook**: Procedural knowledge (HOW-TOs)
- **Patternbook**: Vector embeddings for similarity search

### 🤖 Agents (`/agents`)
- **Interpreters**: Analyze and extract structure
- **Orchestration**: Coordinate workflows
- **Validation**: Verify hypotheses
- **Scopes**: Specialized analytical perspectives

### ⚡ Skills (`/skills`)
- Pattern detection
- Entropy analysis
- Hypothesis generation
- Grammar construction
- Test generation

### 🔧 Tools (`/tools`)
- PCAP parsers (tshark, scapy)
- Binary analyzers
- Statistical utilities
- Format converters

### 🧪 Labs (`/labs`)
- Pre-configured analysis environments
- Container-based isolation
- Loaded with all necessary tools

---

## Workflow

### HITL Mode (Learning)
1. Human initiates analysis via Cline
2. Agents perform analysis using skills and tools
3. Results written to Notebook
4. Human reviews and approves
5. Validated results promote to Rulebook
6. Workflow captured in Cookbook

### Autonomous Mode (Applying)
1. Load workflow from Cookbook
2. Retrieve similar patterns from Patternbook
3. Execute workflow autonomously
4. Checkpoint on failures or low confidence
5. Human review at checkpoints
6. Continue execution

---

## Quick Start

### HITL Mode (Development)

```bash
# Open in Dev Container (VSCode)
# Container includes Cline + Local LLM support

# Start analysis session
python -m reshark.agents.orchestration.hitl_orchestrator \
  --target data/input/sample.pcap \
  --mode hitl

# Cline will guide the analysis workflow
```

### Autonomous Mode

```bash
# Start lab environment
cd labs
docker-compose up -d

# Run autonomous analysis
docker exec reshark-autonomous python -m reshark.agents.orchestration.autonomous_orchestrator \
  --workflow analysis_workflow \
  --target /data/input/sample.pcap
```

---

## Use Cases

- Network protocol reverse engineering (PCAP analysis)
- Binary format documentation
- Data structure discovery
- Communication pattern analysis
- System behavior modeling

---

## Documentation

- [System Requirements Specification](specs/system-requirements-specification.md)
- [System Implementation Technical Design](specs/system-implementation-technical-design.md)
- [Constitution v2.0](docs/constitution-v2.md)

---

## Open Core

reShark framework is open-source.  
Specific analysis results (grammars for proprietary protocols) remain private unless authorized for release.

---

## License

See [LICENSE](LICENSE) for details.
