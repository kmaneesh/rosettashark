# Agents - Autonomous Analysis Components

**Version**: 2.0.0  
**Purpose**: Specialized agents for analysis, orchestration, and validation

---

## Overview

Agents are autonomous components that perform specialized tasks in the reverse engineering process. They operate in both HITL and Autonomous modes, invoking skills and tools to accomplish their objectives.

## Agent Architecture

```
agents/
├── base.py                 # Base agent class
├── interpreters/           # Data analysis agents
│   ├── pattern_analyzer.py
│   └── structure_detector.py
├── orchestration/          # Workflow coordination
│   ├── hitl_orchestrator.py
│   └── autonomous_orchestrator.py
├── validation/             # Hypothesis testing
│   ├── hypothesis_tester.py
│   └── cross_validator.py
└── scopes/                 # Specialized perspectives
    ├── observer.py
    └── theorist.py
```

---

## Agent Types

### 1. Interpreters (`/interpreters`)

**Purpose**: Analyze and extract structure from raw data

**Pattern Analyzer**
- Detects repeating patterns in data
- Identifies structural boundaries
- Computes statistical properties
- Invokes pattern detection skills

**Structure Detector**
- Infers field boundaries
- Identifies data types
- Maps relationships between elements

**Example Usage**:
```python
from reshark.agents.interpreters import PatternAnalyzer

analyzer = PatternAnalyzer(workspace, skills, tools)
results = analyzer.execute({
    "data_path": "data/input/sample.pcap",
    "mode": "hitl"
})
```

### 2. Orchestration (`/orchestration`)

**Purpose**: Coordinate multi-agent workflows

**HITL Orchestrator**
- Guides human through analysis steps
- Suggests next actions based on state
- Captures workflow for Cookbook
- Requires human approval for each step

**Autonomous Orchestrator**
- Loads workflows from Cookbook
- Executes steps independently
- Implements checkpoints for human review
- Uses Patternbook for pattern matching

**Example Usage**:
```python
from reshark.agents.orchestration import HITLOrchestrator

orchestrator = HITLOrchestrator(workspace, skills, tools)
session_id = orchestrator.start_session("Analyze protocol X")

# Get AI suggestion for next step
suggestion = orchestrator.suggest_next_step(current_state)

# Execute with human approval
result = orchestrator.execute_step(suggestion, human_approved=True)
```

### 3. Validation (`/validation`)

**Purpose**: Test hypotheses and verify results

**Hypothesis Tester**
- Generates test suites from hypotheses
- Executes tests against multiple data samples
- Records pass/fail results
- Prevents invalid promotions to Rulebook

**Cross Validator**
- Validates across diverse data sets (min 3)
- Tests edge cases and boundaries
- Ensures generalization of findings

**Example Usage**:
```python
from reshark.agents.validation import HypothesisTester

tester = HypothesisTester(workspace, skills, tools)
results = tester.execute({
    "hypotheses": "notebook:hypotheses",
    "test_data": ["sample1.pcap", "sample2.pcap", "sample3.pcap"]
})
```

### 4. Scopes (`/scopes`)

**Purpose**: Specialized analytical perspectives

**Observer**
- Statistical analysis of data
- Entropy measurement
- Frequency distribution
- Invariant detection

**Theorist**
- Hypothesis generation
- Pattern interpretation
- Model construction
- State inference

**Example Usage**:
```python
from reshark.agents.scopes import Observer

observer = Observer(workspace, skills, tools)
observations = observer.execute({
    "session_id": session_id,
    "data_path": "data/input/sample.pcap"
})
```

---

## Agent Base Class

All agents inherit from the base `Agent` class:

```python
from abc import ABC, abstractmethod
from typing import Dict, Any

class Agent(ABC):
    """Base class for all reShark agents"""
    
    def __init__(self, workspace, skills_registry, tools_registry):
        self.workspace = workspace
        self.skills = skills_registry
        self.tools = tools_registry
        self.mode = None  # 'hitl' or 'autonomous'
    
    @abstractmethod
    def execute(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
        """Main execution method"""
        pass
    
    def invoke_skill(self, skill_name: str, params: Dict[str, Any]) -> Any:
        """Invoke a reusable skill"""
        return self.skills[skill_name].execute(params)
    
    def invoke_tool(self, tool_name: str, params: Dict[str, Any]) -> Any:
        """Invoke a tool"""
        return self.tools[tool_name].run(params)
    
    def read_notebook(self, key: str) -> Any:
        """Read from session Notebook"""
        pass
    
    def write_notebook(self, key: str, data: Any) -> None:
        """Write to session Notebook"""
        pass
```

---

## Agent Responsibilities

### ✅ Agents SHOULD:
- Use skills for reusable analysis logic
- Use tools for external program execution
- Write all findings to Notebook
- Read validated knowledge from Rulebook
- Respect Constitutional memory boundaries
- Log all actions for audit trail

### ❌ Agents SHOULD NOT:
- Directly promote to Rulebook (Archivist only)
- Modify Rulebook entries
- Execute without proper authorization
- Skip validation steps
- Hide failures or errors

---

## Mode-Specific Behavior

### HITL Mode
- Wait for human approval before major actions
- Provide explanations for suggestions
- Capture decision rationale
- Allow interactive debugging

### Autonomous Mode
- Execute based on learned workflows
- Implement confidence thresholds
- Trigger checkpoints on anomalies
- Continue after human checkpoint approval

---

## Constitutional Compliance

Agents enforce Constitutional principles:

1. **Epistemic Law**: Only measurable claims
2. **Memory Boundaries**: Notebook ≠ Rulebook
3. **Validation Required**: 3+ samples minimum
4. **Transparency**: All reasoning logged
5. **Authority Scopes**: Stay within domain

---

## Development

### Creating New Agents

```python
# reshark/agents/custom/my_agent.py

from reshark.agents.base import Agent
from typing import Dict, Any

class MyAgent(Agent):
    """Description of agent purpose"""
    
    def execute(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
        # 1. Validate inputs
        self._validate_inputs(inputs)
        
        # 2. Invoke skills/tools
        result = self.invoke_skill("pattern_detection", {
            "data": inputs["data"]
        })
        
        # 3. Write to Notebook
        self.write_notebook("my_analysis", result)
        
        # 4. Return results
        return result
```

### Testing Agents

```python
# tests/test_agents/test_my_agent.py

import pytest
from reshark.agents.custom import MyAgent

def test_agent_execution():
    agent = MyAgent(workspace, skills, tools)
    result = agent.execute({"data": test_data})
    assert result["status"] == "success"
```

---

## References

- [System Requirements Specification](../specs/system-requirements-specification.md)
- [System Implementation Technical Design](../specs/system-implementation-technical-design.md)
- [Constitution v2.0](../docs/constitution-v2.md)
- [Skills README](../skills/README.md)
- [Tools README](../tools/README.md)
