# reShark System Implementation Technical Design (SITD)

**Version**: 2.0.0  
**Date**: 2026-01-15  
**Status**: Draft  
**Implements**: System Requirements Specification v2.0.0  
**Governed By**: reShark Constitution v2.0.0

---

## 1. Introduction

### 1.1 Purpose

This document provides the technical design for implementing reShark, a general-purpose reverse engineering framework. It details architecture, components, interfaces, and data structures for both Human-in-the-Loop (HITL) mode (Dev Container + Cline + Local LLM) and Autonomous mode (OpenHands + Local LLM), emphasizing knowledge transfer between modes.

### 1.2 Scope

This design covers:
- Detailed component architecture
- Module interfaces and APIs
- Data models and schemas
- File system organization
- Execution flows
- Technology stack and dependencies

### 1.3 Design Principles

1. **Files are Truth**: All authoritative state lives on disk, not in memory
2. **Explicit over Implicit**: No hidden state or silent assumptions
3. **Testable by Default**: Every component has defined test interfaces
4. **Learn then Automate**: HITL mode teaches, Autonomous mode applies learned patterns
5. **Mode Symmetry**: Both modes use same components (agents, skills, tools, books, labs)
6. **Constitutional Compliance**: All designs enforce Constitution v2.0.0
7. **General Purpose**: Framework supports multiple reverse engineering domains, not just protocols

---

## 2. System Architecture

### 2.1 High-Level Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                    reShark Framework                              │
│                                                                   │
│  ┌─────────────────────────┐    ┌─────────────────────────┐    │
│  │   HITL Mode             │    │   Autonomous Mode       │    │
│  │   (Interactive)         │    │   (Self-Directed)       │    │
│  │                         │    │                         │    │
│  │  Dev Container          │    │  OpenHands              │    │
│  │  + Cline                │    │  + Local LLM            │    │
│  │  + Local LLM            │    │                         │    │
│  │                         │    │                         │    │
│  │  Human Guidance ────────┼────┼──> Learned Patterns     │    │
│  │  Knowledge Capture      │    │    Pattern Application  │    │
│  └────────────┬────────────┘    └──────────┬──────────────┘    │
│               │                            │                    │
│               └────────────┬───────────────┘                    │
│                            │                                    │
│               ┌────────────▼────────────┐                       │
│               │      Components         │                       │
│               │   (Shared by Both)      │                       │
│               └────────────┬────────────┘                       │
│                            │                                    │
│         ┌──────────────────┼───────────────────┐               │
│         │                  │                   │               │
│    ┌────▼────┐      ┌─────▼─────┐      ┌─────▼─────┐         │
│    │ Agents  │      │  Skills   │      │   Tools   │         │
│    │(/agents)│      │ (/skills) │      │  (/tools) │         │
│    └────┬────┘      └─────┬─────┘      └─────┬─────┘         │
│         │                  │                   │               │
│         │    ┌─────────────▼────────────┐      │               │
│         │    │        Books             │      │               │
│         └───>│  (/books)                │<─────┘               │
│              │  - notebook (working)    │                      │
│              │  - rulebook (validated)  │                      │
│              │  - cookbook (procedures) │                      │
│              │  - patternbook (vectors) │                      │
│              └──────────────┬───────────┘                      │
│                             │                                  │
│                    ┌────────▼─────────┐                        │
│                    │   Lab (/labs)    │                        │
│                    │  Pre-configured  │                        │
│                    │   Environment    │                        │
│                    └──────────────────┘                        │
└──────────────────────────────────────────────────────────────────┘
```

### 2.2 Component Interactions (HITL Mode)

```
Human Operator (via Cline)
      │
      │ 1. Initiates analysis workflow
      ↓
Cline AI Assistant
      │
      │ 2. Suggests approach, invokes agents
      ↓
Interpreter Agent (/agents/interpreters)
      │
      │ 3. Calls tool scripts (via /tools)
      ↓
Tool execution → Data parsed
      │
      │ 4. Returns structured data
      ↓
Interpreter Agent
      │
      │ 5. Applies skill (pattern detection)
      ↓
Pattern Detection Skill (/skills)
      │
      │ 6. Computes statistics
      │ 7. Writes to Notebook
      ↓
Notebook/session-X/observations.json
      │
      │ 8. Human reviews via Cline
      ↓
Human Operator
      │
      │ 9. Approves, requests hypothesis generation
      ↓
Orchestration Agent (/agents/orchestration)
      │
      │ 10. Reads observations from Notebook
      │ 11. Invokes hypothesis generation skill
      ↓
Hypothesis Skill (/skills)
      │
      │ 12. Generates hypotheses
      │ 13. Writes to Notebook
      ↓
Notebook/session-X/hypotheses.json
      │
      │ 14. Human reviews via Cline
      ↓
Human Operator
      │
      │ 15. Approves validation
      ↓
Validation Agent (/agents/validation)
      │
      │ 16. Reads hypotheses from Notebook
      │ 17. Generates test suite
      │ 18. Executes tests via tools
      │ 19. Writes results to Notebook
      ↓
Notebook/session-X/validation.json
      │
      │ 20. Human reviews validation results
      │ 21. Learned workflow captured in Cookbook
      ↓
Human Operator
      │
      │ 22. Approves promotion
      ↓
Orchestration Agent
      │
      │ 23. Reads validated hypothesis
      │ 24. Generates grammar
      │ 25. Writes to Rulebook
      ↓
Rulebook/grammars/target_v1.py
Rulebook/tests/test_target_v1.py
Cookbook/workflows/analysis_workflow.md
```

### 2.3 Autonomous Mode Architecture

```
┌──────────────────────────────────────────────────────┐
│         Autonomous Mode (OpenHands + Local LLM)       │
│                                                       │
│  OpenHands Execution Environment                     │
│       │                                               │
│       │ 1. Reads workflow from Cookbook              │
│       ↓                                               │
│  Cookbook/workflows/analysis_workflow.md             │
│       │                                               │
│       │ 2. Loads learned patterns from Patternbook   │
│       ↓                                               │
│  Patternbook (Vector DB)                             │
│       │                                               │
│       │ 3. Autonomous orchestration begins           │
│       ↓                                               │
│  Orchestration Agent                                 │
│       │                                               │
│       ├──> Interpreter Agent                         │
│       │    (applies learned skills)                  │
│       │                                               │
│       ├──> Validation Agent                          │
│       │    (automated testing)                       │
│       │                                               │
│       └──> Scope Agents                              │
│            (specialized analysis)                    │
│                                                       │
│  All agents write to Notebook                        │
│  Validated results promote to Rulebook              │
│                                                       │
│  Checkpoints trigger human review:                   │
│  - Validation failures                               │
│  - Confidence thresholds                             │
│  - Conflicting patterns                              │
│                                                       │
└──────────────────────────────────────────────────────┘

Knowledge Transfer Flow:
HITL Mode → Cookbook (procedures)
         → Skills (reusable capabilities)
         → Patternbook (pattern embeddings)
         → Rulebook (validated models)
                    ↓
            Autonomous Mode reads and applies
```

---

## 3. Component Design

### 3.1 Agent System

Agents are autonomous components that perform specialized analysis tasks. They operate in both HITL and Autonomous modes, using skills and tools to accomplish their objectives.

#### 3.1.1 Agent Base Class

```python
# reshark/agents/base.py

from abc import ABC, abstractmethod
from typing import Dict, Any, Optional, List
from pathlib import Path
import json
from datetime import datetime

class Agent(ABC):
    """
    Base class for all reShark Agents.
    Enforces Constitutional constraints and memory boundaries.
    Agents can invoke skills and tools to accomplish tasks.
    """
    
    def __init__(self, notebook_path: Path, rulebook_path: Path, 
                 cookbook_path: Path, skills_registry: Dict,
                 tools_registry: Dict):
        self.notebook_path = notebook_path
        self.rulebook_path = rulebook_path
        self.cookbook_path = cookbook_path
        self.skills = skills_registry
        self.tools = tools_registry
        self.session_id = None
        self.mode = None  # 'hitl' or 'autonomous'
        
    @abstractmethod
    def execute(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
        """
        Main execution method for Agent logic.
        
        Args:
            inputs: Dictionary of input parameters
            
        Returns:
            Dictionary of results
            
        Raises:
            AgentError: If execution fails
        """
        pass
    
    def invoke_skill(self, skill_name: str, params: Dict[str, Any]) -> Any:
        """Invoke a reusable skill from the skills library."""
        if skill_name not in self.skills:
            raise ValueError(f"Skill {skill_name} not found")
        return self.skills[skill_name].execute(params)
    
    def invoke_tool(self, tool_name: str, params: Dict[str, Any]) -> Any:
        """Invoke a tool from the tools library."""
        if tool_name not in self.tools:
            raise ValueError(f"Tool {tool_name} not found")
        return self.tools[tool_name].run(params)
    
    def read_notebook(self, key: str) -> Optional[Any]:
        """Read data from Notebook."""
        notebook_file = self.notebook_path / self.session_id / f"{key}.json"
        if not notebook_file.exists():
            return None
        with open(notebook_file) as f:
            return json.load(f)
    
    def write_notebook(self, key: str, data: Any) -> None:
        """Write data to Notebook with timestamp."""
        session_dir = self.notebook_path / self.session_id
        session_dir.mkdir(parents=True, exist_ok=True)
        
        entry = {
            "timestamp": datetime.utcnow().isoformat(),
            "scope": self.__class__.__name__,
            "data": data
        }
        
        notebook_file = session_dir / f"{key}.json"
        with open(notebook_file, 'w') as f:
            json.dump(entry, f, indent=2)
    
    def read_rulebook(self, grammar_name: str) -> Optional[str]:
        """Read grammar from Rulebook (read-only)."""
        grammar_file = self.rulebook_path / "grammars" / f"{grammar_name}.py"
        if not grammar_file.exists():
            return None
        return grammar_file.read_text()
    
    def read_cookbook(self, recipe_name: str) -> Optional[str]:
        """Read procedure from Cookbook."""
        recipe_file = self.cookbook_path / f"{recipe_name}.md"
        if not recipe_file.exists():
            return None
        return recipe_file.read_text()
    
    @property
    @abstractmethod
    def authority_scope(self) -> str:
        """Return description of Scope's authority domain."""
        pass
    
    def _log_evidence(self, evidence: Dict[str, Any]) -> None:
        """Log evidence trail for Constitutional compliance."""
        self.write_notebook(f"evidence_{datetime.utcnow().timestamp()}", 
                           evidence)
```

#### 3.1.2 Observer Scope

```python
# reshark/scopes/observer.py

from .base import Scope
from typing import Dict, Any, List
import numpy as np
from scipy.stats import entropy
import subprocess
import json

class Observer(Scope):
    """
    Observer Scope: Measures and detects patterns in network traffic.
    
    Authority Scope:
    - Entropy measurement
    - Byte frequency analysis
    - Invariant detection
    - Statistical correlation
    
    Prohibited:
    - Hypothesis generation (Theorist's domain)
    - Grammar validation (Validator's domain)
    """
    
    @property
    def authority_scope(self) -> str:
        return "statistical_analysis_and_pattern_detection"
    
    def execute(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
        """
        Analyze PCAP and produce statistical observations.
        
        Args:
            inputs: {
                "pcap_path": Path to PCAP file,
                "stream_filter": Optional tshark filter,
                "session_id": Analysis session identifier
            }
            
        Returns:
            {
                "entropy_map": Byte-position entropy values,
                "frequency_dist": Byte frequency distribution,
                "invariants": List of fixed byte positions,
                "control_bytes": Candidate control byte positions
            }
        """
        self.session_id = inputs["session_id"]
        pcap_path = inputs["pcap_path"]
        
        # Extract byte streams via MCP/tshark
        streams = self._extract_streams(pcap_path, 
                                       inputs.get("stream_filter"))
        
        # Compute statistics
        results = {
            "entropy_map": self._compute_entropy_map(streams),
            "frequency_dist": self._compute_frequency_dist(streams),
            "invariants": self._detect_invariants(streams),
            "control_bytes": self._identify_control_bytes(streams),
            "message_lengths": self._analyze_lengths(streams)
        }
        
        # Record evidence
        self._log_evidence({
            "pcap": str(pcap_path),
            "stream_count": len(streams),
            "method": "entropy_and_frequency_analysis"
        })
        
        # Write to Notebook
        self.write_notebook("observations", results)
        
        return results
    
    def _extract_streams(self, pcap_path: str, 
                        filter_expr: str = None) -> List[bytes]:
        """Extract byte streams using tshark directly."""
        # Direct tshark subprocess call
        cmd = ["tshark", "-r", pcap_path, "-T", "fields", 
               "-e", "data.data"]
        if filter_expr:
            cmd.extend(["-Y", filter_expr])
        
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        # Parse hex strings to bytes
        streams = []
        for line in result.stdout.strip().split('\n'):
            if line:
                streams.append(bytes.fromhex(line.replace(':', '')))
        
        return streams
    
    def _compute_entropy_map(self, streams: List[bytes]) -> Dict[int, float]:
        """Compute Shannon entropy for each byte position."""
        max_len = max(len(s) for s in streams)
        entropy_map = {}
        
        for pos in range(max_len):
            bytes_at_pos = [s[pos] for s in streams if len(s) > pos]
            if bytes_at_pos:
                freq = np.bincount(bytes_at_pos, minlength=256)
                freq = freq / freq.sum()
                entropy_map[pos] = float(entropy(freq, base=2))
        
        return entropy_map
    
    def _compute_frequency_dist(self, streams: List[bytes]) -> Dict[str, Any]:
        """Compute overall and per-position byte frequencies."""
        all_bytes = b''.join(streams)
        freq = np.bincount(list(all_bytes), minlength=256)
        
        return {
            "global": freq.tolist(),
            "most_common": np.argsort(freq)[-10:].tolist()
        }
    
    def _detect_invariants(self, streams: List[bytes]) -> List[Dict[str, Any]]:
        """Detect byte positions with fixed values."""
        max_len = max(len(s) for s in streams)
        invariants = []
        
        for pos in range(max_len):
            bytes_at_pos = [s[pos] for s in streams if len(s) > pos]
            unique = set(bytes_at_pos)
            
            if len(unique) == 1:
                invariants.append({
                    "position": pos,
                    "value": list(unique)[0],
                    "confidence": 1.0
                })
        
        return invariants
    
    def _identify_control_bytes(self, streams: List[bytes]) -> List[int]:
        """Identify low-entropy positions (likely control bytes)."""
        entropy_map = self._compute_entropy_map(streams)
        
        # Control bytes typically have entropy < 2 bits
        control_positions = [
            pos for pos, ent in entropy_map.items() 
            if ent < 2.0
        ]
        
        return control_positions
    
    def _analyze_lengths(self, streams: List[bytes]) -> Dict[str, Any]:
        """Analyze message length distribution."""
        lengths = [len(s) for s in streams]
        
        return {
            "min": min(lengths),
            "max": max(lengths),
            "mean": float(np.mean(lengths)),
            "std": float(np.std(lengths)),
            "distribution": np.bincount(lengths).tolist()
        }
```

#### 3.1.3 Theorist Scope

```python
# reshark/scopes/theorist.py

from .base import Scope
from typing import Dict, Any, List

class Theorist(Scope):
    """
    Theorist Scope: Generates testable hypotheses from observations.
    
    Authority Scope:
    - Hypothesis generation
    - Field boundary inference
    - State model proposals
    
    Prohibited:
    - Direct measurement (Observer's domain)
    - Hypothesis validation (Validator's domain)
    - Rulebook promotion (Archivist's domain)
    """
    
    @property
    def authority_scope(self) -> str:
        return "hypothesis_generation_and_inference"
    
    def execute(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
        """
        Generate hypotheses from Observer's observations.
        
        Args:
            inputs: {
                "session_id": Analysis session identifier,
                "pcap_context": Optional protocol hints
            }
            
        Returns:
            {
                "hypotheses": List of testable hypothesis objects
            }
        """
        self.session_id = inputs["session_id"]
        
        # Read observations from Notebook
        observations = self.read_notebook("observations")
        if not observations:
            raise ValueError("No observations found in Notebook")
        
        # Generate hypotheses
        hypotheses = []
        
        # Hypothesis 1: Message boundaries via invariants
        hypotheses.extend(self._hypothesize_boundaries(observations))
        
        # Hypothesis 2: Message types via control bytes
        hypotheses.extend(self._hypothesize_message_types(observations))
        
        # Hypothesis 3: Field structures
        hypotheses.extend(self._hypothesize_fields(observations))
        
        # Write to Notebook
        self.write_notebook("hypotheses", hypotheses)
        
        # Log reasoning
        self._log_evidence({
            "observation_count": len(observations.get("data", {})),
            "hypothesis_count": len(hypotheses),
            "method": "statistical_inference"
        })
        
        return {"hypotheses": hypotheses}
    
    def _hypothesize_boundaries(self, observations: Dict) -> List[Dict]:
        """Propose message boundary detection rules."""
        invariants = observations.get("data", {}).get("invariants", [])
        
        hypotheses = []
        for inv in invariants[:3]:  # Top 3 invariants
            hypotheses.append({
                "type": "message_boundary",
                "claim": f"Messages start with byte 0x{inv['value']:02x} "
                        f"at position {inv['position']}",
                "testable": True,
                "test_method": "scan_for_delimiter",
                "parameters": {
                    "delimiter": inv["value"],
                    "position": inv["position"]
                }
            })
        
        return hypotheses
    
    def _hypothesize_message_types(self, observations: Dict) -> List[Dict]:
        """Propose message type field locations."""
        control_bytes = observations.get("data", {}).get("control_bytes", [])
        
        hypotheses = []
        if control_bytes:
            hypotheses.append({
                "type": "message_type_field",
                "claim": f"Byte at position {control_bytes[0]} indicates "
                        "message type",
                "testable": True,
                "test_method": "group_by_byte_value",
                "parameters": {
                    "position": control_bytes[0]
                }
            })
        
        return hypotheses
    
    def _hypothesize_fields(self, observations: Dict) -> List[Dict]:
        """Propose field boundary locations."""
        entropy_map = observations.get("data", {}).get("entropy_map", {})
        
        # High-entropy regions likely contain variable data fields
        high_entropy_positions = [
            int(pos) for pos, ent in entropy_map.items() 
            if ent > 6.0
        ]
        
        hypotheses = []
        if len(high_entropy_positions) > 1:
            hypotheses.append({
                "type": "field_boundaries",
                "claim": f"Variable fields exist at positions "
                        f"{high_entropy_positions}",
                "testable": True,
                "test_method": "extract_fields",
                "parameters": {
                    "positions": high_entropy_positions
                }
            })
        
        return hypotheses
```

#### 3.1.4 Validator Scope

```python
# reshark/scopes/validator.py

from .base import Scope
from typing import Dict, Any, List
from pathlib import Path
import subprocess

class Validator(Scope):
    """
    Validator Scope: Tests and falsifies hypotheses.
    
    Authority Scope:
    - Test suite generation
    - Cross-PCAP validation
    - Negative testing
    - Falsification
    
    Prohibited:
    - Hypothesis generation (Theorist's domain)
    - Statistical measurement (Observer's domain)
    """
    
    @property
    def authority_scope(self) -> str:
        return "hypothesis_testing_and_validation"
    
    def execute(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
        """
        Validate hypotheses against multiple PCAPs.
        
        Args:
            inputs: {
                "session_id": Analysis session identifier,
                "validation_pcaps": List of PCAP paths (min 3),
                "negative_pcaps": List of corrupted PCAPs (optional)
            }
            
        Returns:
            {
                "validation_results": List of results per hypothesis,
                "passed": List of validated hypothesis IDs,
                "failed": List of failed hypothesis IDs
            }
        """
        self.session_id = inputs["session_id"]
        
        # Read hypotheses from Notebook
        hypotheses = self.read_notebook("hypotheses")
        if not hypotheses:
            raise ValueError("No hypotheses found in Notebook")
        
        validation_pcaps = inputs["validation_pcaps"]
        if len(validation_pcaps) < 3:
            raise ValueError("Constitutional requirement: minimum 3 PCAPs for validation")
        
        # Test each hypothesis
        results = []
        passed = []
        failed = []
        
        for i, hyp in enumerate(hypotheses.get("data", [])):
            result = self._validate_hypothesis(hyp, validation_pcaps)
            results.append(result)
            
            if result["passed"]:
                passed.append(i)
            else:
                failed.append(i)
        
        # Negative testing (if provided)
        if inputs.get("negative_pcaps"):
            negative_results = self._negative_testing(
                hypotheses.get("data", []), 
                inputs["negative_pcaps"]
            )
            results.extend(negative_results)
        
        # Write to Notebook
        output = {
            "validation_results": results,
            "passed": passed,
            "failed": failed,
            "pcap_count": len(validation_pcaps)
        }
        self.write_notebook("validation", output)
        
        # Log evidence
        self._log_evidence({
            "validated_count": len(hypotheses.get("data", [])),
            "passed": len(passed),
            "failed": len(failed),
            "pcaps_tested": len(validation_pcaps)
        })
        
        return output
    
    def _validate_hypothesis(self, hypothesis: Dict, 
                            pcaps: List[str]) -> Dict[str, Any]:
        """Validate single hypothesis across PCAPs."""
        test_method = hypothesis.get("test_method")
        parameters = hypothesis.get("parameters", {})
        
        # Generate test code
        test_code = self._generate_test(hypothesis)
        
        # Execute tests
        passed_count = 0
        for pcap in pcaps:
            if self._run_test(test_code, pcap, parameters):
                passed_count += 1
        
        # Hypothesis passes if ALL PCAPs validate
        passed = (passed_count == len(pcaps))
        
        return {
            "hypothesis": hypothesis["claim"],
            "passed": passed,
            "pcaps_tested": len(pcaps),
            "pcaps_passed": passed_count,
            "test_code": test_code
        }
    
    def _generate_test(self, hypothesis: Dict) -> str:
        """Generate pytest test code for hypothesis."""
        test_template = f"""
def test_{hypothesis['type']}(pcap_file):
    '''Test: {hypothesis['claim']}'''
    # Test implementation based on {hypothesis['test_method']}
    # Parameters: {hypothesis['parameters']}
    
    streams = extract_streams(pcap_file)
    
    # Test logic here
    assert validate_{hypothesis['test_method']}(
        streams, 
        {hypothesis['parameters']}
    )
"""
        return test_template
    
    def _run_test(self, test_code: str, pcap: str, 
                  params: Dict) -> bool:
        """Execute test against PCAP."""
        # In real implementation, would use pytest programmatically
        # For now, simplified pass/fail logic
        
        try:
            # Parse PCAP and apply test logic
            # This invokes scapy/tshark directly
            return True  # Placeholder
        except AssertionError:
            return False
    
    def _negative_testing(self, hypotheses: List[Dict], 
                         negative_pcaps: List[str]) -> List[Dict]:
        """Test that hypotheses correctly reject malformed data."""
        results = []
        
        for hyp in hypotheses:
            # Hypothesis should FAIL on corrupted data
            should_reject = True
            
            for pcap in negative_pcaps:
                test_code = self._generate_test(hyp)
                passed = self._run_test(test_code, pcap, 
                                       hyp.get("parameters", {}))
                
                # We WANT this to fail
                if not passed:
                    should_reject = should_reject and True
                else:
                    should_reject = False
            
            results.append({
                "hypothesis": hyp["claim"],
                "negative_test_passed": should_reject,
                "type": "negative_validation"
            })
        
        return results
```

#### 3.1.5 Archivist Scope

```python
# reshark/scopes/archivist.py

from .base import Scope
from typing import Dict, Any
from pathlib import Path
import shutil
from datetime import datetime

class Archivist(Scope):
    """
    Archivist Scope: Manages memory and promotes validated hypotheses.
    
    Authority Scope:
    - Notebook management
    - Rulebook promotion
    - Session archival
    - Memory boundary enforcement
    
    Prohibited:
    - Hypothesis generation (Theorist's domain)
    - Validation (Validator's domain)
    """
    
    @property
    def authority_scope(self) -> str:
        return "memory_management_and_promotion"
    
    def execute(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
        """
        Manage memory operations: archive, promote, or cleanup.
        
        Args:
            inputs: {
                "operation": "promote" | "archive" | "cleanup",
                "session_id": Session identifier,
                "hypothesis_id": (for promote) ID of validated hypothesis
            }
            
        Returns:
            {
                "status": "success" | "failed",
                "details": Operation-specific results
            }
        """
        self.session_id = inputs["session_id"]
        operation = inputs["operation"]
        
        if operation == "promote":
            return self._promote_to_rulebook(inputs)
        elif operation == "archive":
            return self._archive_session()
        elif operation == "cleanup":
            return self._cleanup_notebook()
        else:
            raise ValueError(f"Unknown operation: {operation}")
    
    def _promote_to_rulebook(self, inputs: Dict) -> Dict[str, Any]:
        """
        Promote validated hypothesis to Rulebook.
        
        Constitutional Requirements:
        - Hypothesis MUST have passed validation (Validator authority)
        - Evidence MUST be included
        - Test suite MUST accompany grammar
        """
        hypothesis_id = inputs["hypothesis_id"]
        
        # Read validation results
        validation = self.read_notebook("validation")
        if not validation:
            return {
                "status": "failed",
                "reason": "No validation results found"
            }
        
        # Check if hypothesis passed
        if hypothesis_id not in validation.get("data", {}).get("passed", []):
            return {
                "status": "failed",
                "reason": "Hypothesis did not pass validation"
            }
        
        # Read hypothesis details
        hypotheses = self.read_notebook("hypotheses")
        hypothesis = hypotheses.get("data", [])[hypothesis_id]
        
        # Generate grammar file
        grammar_name = f"protocol_{self.session_id}_v1"
        grammar_code = self._generate_grammar(hypothesis, validation)
        
        # Write to Rulebook
        grammar_path = self.rulebook_path / "grammars" / f"{grammar_name}.py"
        grammar_path.write_text(grammar_code)
        
        # Generate test suite
        test_code = self._generate_test_suite(hypothesis, validation)
        test_path = self.rulebook_path / "tests" / f"test_{grammar_name}.py"
        test_path.write_text(test_code)
        
        # Record promotion metadata
        metadata = {
            "session_id": self.session_id,
            "hypothesis_id": hypothesis_id,
            "promoted_at": datetime.utcnow().isoformat(),
            "validation_pcap_count": validation.get("data", {}).get("pcap_count"),
            "grammar_file": str(grammar_path),
            "test_file": str(test_path)
        }
        
        metadata_path = self.rulebook_path / "grammars" / f"{grammar_name}.meta.json"
        import json
        with open(metadata_path, 'w') as f:
            json.dump(metadata, f, indent=2)
        
        return {
            "status": "success",
            "grammar": str(grammar_path),
            "tests": str(test_path),
            "metadata": str(metadata_path)
        }
    
    def _generate_grammar(self, hypothesis: Dict, 
                         validation: Dict) -> str:
        """Generate executable protocol grammar."""
        grammar_template = f"""
# Generated Protocol Grammar
# Hypothesis: {hypothesis['claim']}
# Validated: {datetime.utcnow().isoformat()}

from typing import List, Dict, Optional
import struct

class ProtocolParser:
    '''
    Parser for protocol based on validated hypothesis.
    '''
    
    def __init__(self):
        self.message_type_field = {hypothesis['parameters'].get('position', 0)}
    
    def parse_message(self, data: bytes) -> Dict:
        '''Parse single message.'''
        if len(data) < self.message_type_field + 1:
            raise ValueError("Message too short")
        
        msg_type = data[self.message_type_field]
        
        return {{
            "type": msg_type,
            "raw": data,
            "length": len(data)
        }}
    
    def parse_stream(self, stream: bytes) -> List[Dict]:
        '''Parse stream into messages.'''
        messages = []
        # Boundary detection logic here
        return messages
"""
        return grammar_template
    
    def _generate_test_suite(self, hypothesis: Dict, 
                            validation: Dict) -> str:
        """Generate pytest test suite."""
        test_template = f"""
# Test Suite for Protocol Grammar
# Generated: {datetime.utcnow().isoformat()}

import pytest
from pathlib import Path

def test_parsing():
    '''Test basic parsing functionality.'''
    # Test implementation
    assert True

def test_cross_pcap_validation():
    '''Test validated across multiple PCAPs.'''
    # Test implementation
    assert True
"""
        return test_template
    
    def _archive_session(self) -> Dict[str, Any]:
        """Archive completed session from Notebook."""
        session_dir = self.notebook_path / self.session_id
        archive_dir = self.notebook_path / "archives" / self.session_id
        
        if session_dir.exists():
            shutil.move(str(session_dir), str(archive_dir))
            return {
                "status": "success",
                "archived_to": str(archive_dir)
            }
        else:
            return {
                "status": "failed",
                "reason": "Session directory not found"
            }
    
    def _cleanup_notebook(self) -> Dict[str, Any]:
        """Clean up ephemeral Notebook entries."""
        # Remove old sessions (older than 7 days)
        cutoff = datetime.utcnow().timestamp() - (7 * 24 * 3600)
        
        cleaned = []
        for session_dir in self.notebook_path.iterdir():
            if session_dir.is_dir() and session_dir.name != "archives":
                if session_dir.stat().st_mtime < cutoff:
                    shutil.rmtree(session_dir)
                    cleaned.append(session_dir.name)
        
        return {
            "status": "success",
            "cleaned_sessions": cleaned
        }
```

### 3.2 Books System Design (Memory Layer)

Books are the knowledge storage system, organized by purpose and persistence level.

#### 3.2.1 File System Structure

```
reshark/
├── books/                       # Knowledge storage
│   ├── notebook/               # Short-term working memory (session-specific)
│   │   ├── sessions/
│   │   │   ├── session-20260115-001/
│   │   │   │   ├── observations.json
│   │   │   │   ├── hypotheses.json
│   │   │   │   ├── validation.json
│   │   │   │   ├── evidence_*.json
│   │   │   │   └── session.log
│   │   │   └── session-20260115-002/
│   │   └── archives/           # Completed sessions
│   │       └── session-20260114-001/
│   │
│   ├── rulebook/               # Validated models and grammars
│   │   ├── grammars/
│   │   │   ├── protocol_foo_v1.py
│   │   │   ├── protocol_foo_v1.meta.json
│   │   │   ├── binary_format_bar_v1.py
│   │   │   └── binary_format_bar_v1.meta.json
│   │   ├── schemas/            # Structured schemas
│   │   │   ├── protocol_foo_v1.yaml
│   │   │   └── binary_format_bar_v1.json
│   │   ├── tests/
│   │   │   ├── test_protocol_foo_v1.py
│   │   │   └── test_binary_format_bar_v1.py
│   │   └── README.md
│   │
│   ├── cookbook/               # Procedural knowledge (HOW-TO)
│   │   ├── workflows/          # Learned workflows
│   │   │   ├── analysis_workflow.md
│   │   │   ├── validation_workflow.md
│   │   │   └── grammar_construction.md
│   │   ├── methods/            # Analysis methods
│   │   │   ├── pattern_detection.md
│   │   │   ├── field_extraction.md
│   │   │   └── state_inference.md
│   │   ├── examples/           # Example analyses
│   │   │   ├── binary_protocol_example.md
│   │   │   └── data_format_example.md
│   │   └── README.md
│   │
│   └── patternbook/            # Vector embeddings (Autonomous mode)
│       ├── embeddings/         # Pattern vectors
│       ├── index/              # Vector index
│       └── config.yaml         # Qdrant configuration
│
├── agents/                     # Agent implementations
│   ├── __init__.py
│   ├── base.py
│   ├── interpreters/           # Data interpreters
│   │   ├── __init__.py
│   │   ├── pattern_analyzer.py
│   │   └── structure_detector.py
│   ├── orchestration/          # Workflow coordinators
│   │   ├── __init__.py
│   │   ├── hitl_orchestrator.py
│   │   └── autonomous_orchestrator.py
│   ├── validation/             # Validation agents
│   │   ├── __init__.py
│   │   ├── hypothesis_tester.py
│   │   └── cross_validator.py
│   └── scopes/                 # Specialized scopes
│       ├── __init__.py
│       ├── observer.py
│       └── theorist.py
│
├── skills/                     # Reusable capabilities
│   ├── __init__.py
│   ├── pattern_detection.py
│   ├── entropy_analysis.py
│   ├── hypothesis_generation.py
│   ├── grammar_construction.py
│   └── test_generation.py
│
├── tools/                      # Scripts and utilities
│   ├── scripts/               # Executable scripts
│   │   ├── pcap_parser.py
│   │   ├── binary_analyzer.py
│   │   └── format_converter.py
│   ├── __init__.py
│   ├── pcap_tools.py          # PCAP-specific utilities
│   ├── binary_tools.py        # Binary analysis utilities
│   └── statistics.py          # Statistical utilities
│
├── labs/                       # Lab environment configs
│   ├── Dockerfile.openhands   # Autonomous mode container
│   ├── Dockerfile.pcap        # Analysis tools container
│   └── README.md
│
├── data/                       # Input/output data (not in books)
│   ├── input/                  # Raw input data
│   │   ├── pcaps/
│   │   ├── binaries/
│   │   └── logs/
│   └── output/                 # Generated artifacts
│       └── reports/
│
├── tests/                       # System tests
│   ├── test_agents/
│   │   ├── test_interpreters.py
│   │   ├── test_orchestration.py
│   │   └── test_validation.py
│   ├── test_skills/
│   │   ├── test_pattern_detection.py
│   │   └── test_entropy_analysis.py
│   └── test_integration/
│       ├── test_hitl_mode.py
│       └── test_autonomous_mode.py
│
├── .devcontainer/               # VSCode dev container config
│   ├── devcontainer.json
│   └── Dockerfile
│
├── docs/                        # Documentation
│   ├── constitution-v2.md
│   ├── srs-v2.md
│   └── sitd-v2.md
│
├── pyproject.toml               # Poetry dependencies
├── README.md
└── .gitignore
```

#### 3.2.2 JSON Schemas

**Observation Schema** (`observations.json`):
```json
{
  "timestamp": "2026-01-11T10:30:00Z",
  "scope": "Observer",
  "data": {
    "entropy_map": {
      "0": 1.2,
      "1": 7.8,
      "2": 0.5
    },
    "frequency_dist": {
      "global": [120, 45, 33, ...],
      "most_common": [0x00, 0xFF, 0x20]
    },
    "invariants": [
      {
        "position": 0,
        "value": 255,
        "confidence": 1.0
      }
    ],
    "control_bytes": [0, 4, 8],
    "message_lengths": {
      "min": 64,
      "max": 1024,
      "mean": 256.5,
      "std": 50.2
    }
  }
}
```

**Hypothesis Schema** (`hypotheses.json`):
```json
{
  "timestamp": "2026-01-11T10:35:00Z",
  "scope": "Theorist",
  "data": [
    {
      "type": "message_boundary",
      "claim": "Messages start with 0xFF at position 0",
      "testable": true,
      "test_method": "scan_for_delimiter",
      "parameters": {
        "delimiter": 255,
        "position": 0
      },
      "confidence": 0.95
    }
  ]
}
```

**Validation Schema** (`validation.json`):
```json
{
  "timestamp": "2026-01-11T10:40:00Z",
  "scope": "Validator",
  "data": {
    "validation_results": [
      {
        "hypothesis": "Messages start with 0xFF at position 0",
        "passed": true,
        "pcaps_tested": 3,
        "pcaps_passed": 3,
        "test_code": "def test_boundary()..."
      }
    ],
    "passed": [0],
    "failed": [],
    "pcap_count": 3
  }
}
```

### 3.3 Skills Library Design

Skills are reusable capabilities that can be invoked by agents or humans. They encapsulate specific analysis techniques and can be composed into complex workflows.

#### 3.3.1 Skill Base Class

```python
# reshark/skills/base.py

from abc import ABC, abstractmethod
from typing import Dict, Any, List, Optional
from pathlib import Path
import json

class Skill(ABC):
    """
    Base class for all reShark Skills.
    Skills are reusable, composable capabilities.
    """
    
    def __init__(self, name: str, description: str, 
                 version: str = "1.0.0"):
        self.name = name
        self.description = description
        self.version = version
        self.execution_count = 0
        
    @abstractmethod
    def execute(self, params: Dict[str, Any]) -> Dict[str, Any]:
        """
        Execute the skill with given parameters.
        
        Args:
            params: Input parameters for the skill
            
        Returns:
            Results dictionary
        """
        pass
    
    @abstractmethod
    def get_schema(self) -> Dict[str, Any]:
        """
        Return JSON schema for input parameters.
        Used for validation and documentation.
        """
        pass
    
    def validate_params(self, params: Dict[str, Any]) -> bool:
        """Validate input parameters against schema."""
        schema = self.get_schema()
        # Implement JSON schema validation
        return True  # Simplified
    
    def log_execution(self, params: Dict[str, Any], 
                     result: Dict[str, Any]) -> None:
        """Log skill execution for learning."""
        self.execution_count += 1
        # Log to appropriate location for pattern learning


# Example: Pattern Detection Skill
class PatternDetectionSkill(Skill):
    """
    Detects repeating patterns in byte sequences.
    """
    
    def __init__(self):
        super().__init__(
            name="pattern_detection",
            description="Detects repeating patterns in binary data",
            version="1.0.0"
        )
    
    def execute(self, params: Dict[str, Any]) -> Dict[str, Any]:
        """
        Args:
            params: {
                "data": bytes or list of bytes,
                "min_pattern_length": int,
                "max_pattern_length": int
            }
        """
        data = params["data"]
        min_len = params.get("min_pattern_length", 2)
        max_len = params.get("max_pattern_length", 16)
        
        patterns = self._detect_patterns(data, min_len, max_len)
        
        return {
            "patterns": patterns,
            "pattern_count": len(patterns),
            "coverage": self._calculate_coverage(patterns, data)
        }
    
    def get_schema(self) -> Dict[str, Any]:
        return {
            "type": "object",
            "properties": {
                "data": {"type": ["string", "array"]},
                "min_pattern_length": {"type": "integer", "minimum": 1},
                "max_pattern_length": {"type": "integer", "minimum": 1}
            },
            "required": ["data"]
        }
    
    def _detect_patterns(self, data, min_len, max_len):
        """Pattern detection algorithm implementation."""
        # Implementation details
        return []
    
    def _calculate_coverage(self, patterns, data):
        """Calculate how much of data is covered by patterns."""
        return 0.0
```

#### 3.3.2 Skills Registry

```python
# reshark/skills/registry.py

from typing import Dict, Optional
from .base import Skill
from .pattern_detection import PatternDetectionSkill
from .entropy_analysis import EntropyAnalysisSkill
from .hypothesis_generation import HypothesisGenerationSkill

class SkillsRegistry:
    """
    Central registry for all available skills.
    Used by agents to discover and invoke capabilities.
    """
    
    def __init__(self):
        self._skills: Dict[str, Skill] = {}
        self._register_builtin_skills()
    
    def _register_builtin_skills(self):
        """Register all built-in skills."""
        self.register(PatternDetectionSkill())
        self.register(EntropyAnalysisSkill())
        self.register(HypothesisGenerationSkill())
    
    def register(self, skill: Skill) -> None:
        """Register a new skill."""
        self._skills[skill.name] = skill
    
    def get(self, name: str) -> Optional[Skill]:
        """Get skill by name."""
        return self._skills.get(name)
    
    def list_skills(self) -> List[str]:
        """List all available skills."""
        return list(self._skills.keys())
    
    def get_skill_info(self, name: str) -> Optional[Dict[str, Any]]:
        """Get detailed information about a skill."""
        skill = self.get(name)
        if not skill:
            return None
        
        return {
            "name": skill.name,
            "description": skill.description,
            "version": skill.version,
            "schema": skill.get_schema(),
            "execution_count": skill.execution_count
        }
```

### 3.4 Tool Integration Layer

#### 3.3.1 Direct Tool Wrappers

```python
# reshark/tools/pcap_tools.py

from typing import Dict, Any, List
import subprocess
import json
from pathlib import Path

class TsharkWrapper:
    """
    Direct wrapper for tshark command-line tool.
    Provides consistent interface for PCAP analysis.
    """
    
    def __init__(self, tshark_path: str = "tshark"):
        self.tshark_path = tshark_path
    
    def extract_streams(self, pcap_path: Path, filter_expr: str = None,
                       fields: List[str] = None) -> Dict[str, Any]:
        """
        Extract data from PCAP file.
        
        Args:
            pcap_path: Path to PCAP file
            filter_expr: Optional Wireshark display filter
            fields: Optional list of fields to extract
            
        Returns:
            Parsed output from tshark
        """
        cmd = [self.tshark_path, "-r", str(pcap_path)]
        
        if filter_expr:
            cmd.extend(["-Y", filter_expr])
        
        if fields:
            cmd.extend(["-T", "fields"])
            for field in fields:
                cmd.extend(["-e", field])
        else:
            cmd.extend(["-T", "fields", "-e", "data.data"])
        
        result = subprocess.run(cmd, capture_output=True, text=True, check=True)
        
        return {
            "stdout": result.stdout,
            "stderr": result.stderr,
            "returncode": result.returncode
        }
    
    def get_stream_count(self, pcap_path: Path) -> int:
        """Count TCP/UDP streams in PCAP."""
        cmd = [self.tshark_path, "-r", str(pcap_path), "-q", "-z", "conv,tcp"]
        result = subprocess.run(cmd, capture_output=True, text=True)
        # Parse conversation statistics
        return len([l for l in result.stdout.split('\n') if '<->' in l])


class ScapyWrapper:
    """
    Wrapper for scapy-based PCAP manipulation.
    """
    
    def __init__(self):
        try:
            from scapy.all import rdpcap, IP, TCP, UDP
            self.rdpcap = rdpcap
            self.IP = IP
            self.TCP = TCP
            self.UDP = UDP
        except ImportError:
            raise RuntimeError("scapy not installed. Install via: pip install scapy")
    
    def read_pcap(self, pcap_path: Path) -> List:
        """Read PCAP file and return packet list."""
        return self.rdpcap(str(pcap_path))
    
    def extract_payloads(self, pcap_path: Path, protocol: str = "TCP") -> List[bytes]:
        """Extract raw payloads from PCAP."""
        packets = self.read_pcap(pcap_path)
        payloads = []
        
        for pkt in packets:
            if protocol == "TCP" and self.TCP in pkt:
                if hasattr(pkt[self.TCP], 'payload'):
                    payload = bytes(pkt[self.TCP].payload)
                    if payload:
                        payloads.append(payload)
            elif protocol == "UDP" and self.UDP in pkt:
                if hasattr(pkt[self.UDP], 'payload'):
                    payload = bytes(pkt[self.UDP].payload)
                    if payload:
                        payloads.append(payload)
        
        return payloads
```

### 3.5 Orchestration Design

Orchestration differs by mode: HITL is interactive with Cline guidance, Autonomous applies learned workflows.

#### 3.5.1 HITL Mode Orchestration

```python
# reshark/agents/orchestration/hitl_orchestrator.py

"""
Human-in-the-Loop orchestration guided by Cline AI assistant.
Interactive workflow that captures knowledge for autonomous reuse.
"""
# reshark/orchestration/analyze_protocol.py

from pathlib import Path
from reshark.scopes import Observer, Theorist, Validator, Archivist
from datetime import datetime
import uuid

def analyze_protocol(golden_pcap: Path, 
                     validation_pcaps: List[Path],
                     workspace: Path) -> Dict[str, Any]:
    """
    Main orchestration workflow for Phase 1.
    
    Args:
        golden_pcap: Reference PCAP for initial analysis
        validation_pcaps: Additional PCAPs for validation (min 3)
        workspace: Root directory for memory system
        
    Returns:
        Analysis results and promoted grammar info
    """
    
    # Initialize session
    session_id = f"session-{datetime.utcnow().strftime('%Y%m%d-%H%M%S')}"
    
    print(f"Starting analysis session: {session_id}")
    print(f"Golden PCAP: {golden_pcap}")
    print(f"Validation PCAPs: {len(validation_pcaps)}")
    
    # Initialize Scopes
    notebook_path = workspace / "notebook"
    rulebook_path = workspace / "rulebook"
    cookbook_path = workspace / "cookbook"
    
    observer = Observer(notebook_path, rulebook_path, cookbook_path)
    theorist = Theorist(notebook_path, rulebook_path, cookbook_path)
    validator = Validator(notebook_path, rulebook_path, cookbook_path)
    archivist = Archivist(notebook_path, rulebook_path, cookbook_path)
    
    # Step 1: Observe
    print("\n[1] Observer: Analyzing Golden PCAP...")
    observations = observer.execute({
        "pcap_path": str(golden_pcap),
        "session_id": session_id
    })
    print(f"  - Detected {len(observations['control_bytes'])} control bytes")
    print(f"  - Found {len(observations['invariants'])} invariants")
    
    # Step 2: Theorize
    print("\n[2] Theorist: Generating hypotheses...")
    hypotheses = theorist.execute({
        "session_id": session_id
    })
    print(f"  - Generated {len(hypotheses['hypotheses'])} hypotheses")
    
    # Step 3: Validate
    print("\n[3] Validator: Testing hypotheses...")
    validation = validator.execute({
        "session_id": session_id,
        "validation_pcaps": [str(p) for p in validation_pcaps]
    })
    print(f"  - Passed: {len(validation['passed'])}")
    print(f"  - Failed: {len(validation['failed'])}")
    
    # Step 4: Human Review Checkpoint
    print("\n[4] Human Review Required")
    print("Validation results:")
    for i in validation['passed']:
        hyp = hypotheses['hypotheses'][i]
        print(f"  ✓ {hyp['claim']}")
    
    for i in validation['failed']:
        hyp = hypotheses['hypotheses'][i]
        print(f"  ✗ {hyp['claim']}")
    
    approve = input("\nPromote validated hypotheses to Rulebook? [y/N]: ")
    
    if approve.lower() == 'y':
        # Step 5: Promote
        print("\n[5] Archivist: Promoting to Rulebook...")
        for hyp_id in validation['passed']:
            result = archivist.execute({
                "operation": "promote",
                "session_id": session_id,
                "hypothesis_id": hyp_id
            })
            print(f"  - Grammar: {result['grammar']}")
            print(f"  - Tests: {result['tests']}")
        
        # Step 6: Archive
        print("\n[6] Archivist: Archiving session...")
        archive_result = archivist.execute({
            "operation": "archive",
            "session_id": session_id
        })
        print(f"  - Archived to: {archive_result['archived_to']}")
        
        return {
            "session_id": session_id,
            "status": "completed",
            "hypotheses_validated": len(validation['passed']),
            "grammars_promoted": len(validation['passed'])
        }
    else:
        print("\nPromotion declined. Session remains in Notebook.")
        return {
            "session_id": session_id,
            "status": "pending_review"
        }

if __name__ == "__main__":
    import sys
    
    if len(sys.argv) < 4:
        print("Usage: python analyze_protocol.py <golden_pcap> <validation_pcap1> <validation_pcap2> ...")
        sys.exit(1)
    
    golden = Path(sys.argv[1])
    validation = [Path(p) for p in sys.argv[2:]]
    workspace = Path("./workspace")
    
    result = analyze_protocol(golden, validation, workspace)
    print(f"\nFinal result: {result}")
```

---

## 4. Data Flow Diagrams

### 4.1 End-to-End Analysis Flow

```
[Golden PCAP]
      │
      ↓
┌─────────────────┐
│  1. Observer    │
│  Scope          │
└────────┬────────┘
         │ writes
         ↓
   [Notebook: observations.json]
         │ reads
         ↓
┌─────────────────┐
│  2. Theorist    │
│  Scope          │
└────────┬────────┘
         │ writes
         ↓
   [Notebook: hypotheses.json]
         │ reads
         ↓
┌─────────────────┐
│  3. Validator   │ ← [Validation PCAPs]
│  Scope          │
└────────┬────────┘
         │ writes
         ↓
   [Notebook: validation.json]
         │
         ↓
┌─────────────────┐
│  4. Human       │
│  Review         │
└────────┬────────┘
         │ approves
         ↓
┌─────────────────┐
│  5. Archivist   │
│  Scope          │
└────────┬────────┘
         │ writes
         ↓
   [Rulebook: grammar.py, test_grammar.py]
```

### 4.2 Scope Interaction Matrix

| Scope    | Reads From           | Writes To          | Calls Tools |
|-----------|---------------------|-------------------|-------------|
| Observer  | PCAPs              | Notebook          | tshark, scapy |
| Theorist  | Notebook           | Notebook          | (none)      |
| Validator | Notebook, PCAPs    | Notebook          | scapy, pytest |
| Archivist | Notebook           | Rulebook, Archives| (none)      |

---

## 5. Mode-Specific Implementations

### 5.1 HITL Mode Implementation (Dev Container + Cline)

#### 5.1.1 Graph Definition

```python
# reshark/orchestration/langgraph_workflow.py (Phase 2)

from langgraph.graph import Graph, END
from reshark.scopes import Observer, Theorist, Validator, Archivist

def create_analysis_graph(workspace: Path) -> Graph:
    """
    Create LangGraph workflow for autonomous protocol analysis.
    """
    
    # Initialize Scopes
    notebook_path = workspace / "notebook"
    rulebook_path = workspace / "rulebook"
    cookbook_path = workspace / "cookbook"
    
    observer = Observer(notebook_path, rulebook_path, cookbook_path)
    theorist = Theorist(notebook_path, rulebook_path, cookbook_path)
    validator = Validator(notebook_path, rulebook_path, cookbook_path)
    archivist = Archivist(notebook_path, rulebook_path, cookbook_path)
    
    # Define graph
    workflow = Graph()
    
    # Add nodes
    workflow.add_node("observe", observer.execute)
    workflow.add_node("theorize", theorist.execute)
    workflow.add_node("validate", validator.execute)
    workflow.add_node("review", human_review_checkpoint)
    workflow.add_node("archive", archivist.execute)
    
    # Define edges
    workflow.add_edge("observe", "theorize")
    workflow.add_edge("theorize", "validate")
    workflow.add_edge("validate", "review")
    
    # Conditional routing after review
    workflow.add_conditional_edges(
        "review",
        route_after_review,
        {
            "promote": "archive",
            "reject": END,
            "refine": "theorize"  # Loop back for refinement
        }
    )
    
    workflow.add_edge("archive", END)
    
    # Set entry point
    workflow.set_entry_point("observe")
    
    return workflow.compile()

def human_review_checkpoint(state: Dict) -> Dict:
    """Human review step (Phase 2 may be optional)."""
    # In Phase 2, this could be async or triggered by threshold
    validation = state["validation"]
    
    if len(validation["passed"]) > 0:
        # Auto-promote if confidence is high
        if all(h.get("confidence", 0) > 0.9 
               for h in validation["passed"]):
            return {"review_decision": "promote"}
    
    # Otherwise, request human input
    return {"review_decision": "review_required"}

def route_after_review(state: Dict) -> str:
    """Route based on review decision."""
    decision = state.get("review_decision", "reject")
    
    if decision == "promote":
        return "promote"
    elif decision == "refine":
        return "refine"
    else:
        return "reject"
```

### 5.2 Patternbook (Vector Memory)

```python
# reshark/memory/patternbook.py (Phase 2)

from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct
from typing import List, Dict, Any
import numpy as np

class Patternbook:
    """
    Patternbook: Vector similarity memory for protocol patterns.
    
    NON-AUTHORITATIVE: Suggestions only, must be validated.
    """
    
    def __init__(self, qdrant_url: str = "http://localhost:6333"):
        self.client = QdrantClient(url=qdrant_url)
        self.collection_name = "protocol_patterns"
        
        # Initialize collection if not exists
        self._init_collection()
    
    def _init_collection(self):
        """Initialize Qdrant collection."""
        collections = self.client.get_collections().collections
        if self.collection_name not in [c.name for c in collections]:
            self.client.create_collection(
                collection_name=self.collection_name,
                vectors_config=VectorParams(
                    size=384,  # Embedding dimension
                    distance=Distance.COSINE
                )
            )
    
    def store_pattern(self, pattern_id: str, embedding: np.ndarray, 
                     metadata: Dict[str, Any]):
        """Store protocol pattern embedding."""
        self.client.upsert(
            collection_name=self.collection_name,
            points=[
                PointStruct(
                    id=pattern_id,
                    vector=embedding.tolist(),
                    payload=metadata
                )
            ]
        )
    
    def find_similar(self, query_embedding: np.ndarray, 
                     top_k: int = 5) -> List[Dict]:
        """Find similar protocol patterns."""
        results = self.client.search(
            collection_name=self.collection_name,
            query_vector=query_embedding.tolist(),
            limit=top_k
        )
        
        return [
            {
                "id": r.id,
                "score": r.score,
                "metadata": r.payload
            }
            for r in results
        ]
    
    def suggest_analogies(self, observations: Dict) -> List[str]:
        """
        Suggest protocol analogies based on observations.
        
        WARNING: Suggestions are NON-AUTHORITATIVE and must be validated.
        """
        # Embed observations
        embedding = self._embed_observations(observations)
        
        # Find similar patterns
        similar = self.find_similar(embedding)
        
        # Return suggestions with disclaimer
        suggestions = []
        for s in similar:
            suggestions.append({
                "protocol": s["metadata"].get("protocol_name"),
                "similarity": s["score"],
                "disclaimer": "SUGGESTION ONLY - MUST BE VALIDATED"
            })
        
        return suggestions
    
    def _embed_observations(self, observations: Dict) -> np.ndarray:
        """Convert observations to embedding vector."""
        # Simplified: in real implementation would use proper embedding model
        features = []
        
        # Entropy statistics
        entropy_map = observations.get("entropy_map", {})
        features.extend([entropy_map.get(str(i), 0) for i in range(10)])
        
        # Control byte count
        features.append(len(observations.get("control_bytes", [])))
        
        # Pad to 384 dimensions
        features.extend([0] * (384 - len(features)))
        
        return np.array(features[:384])
```

---

## 6. Testing Strategy

### 6.1 Unit Tests

```python
# reshark/tests/test_observer.py

import pytest
from pathlib import Path
from reshark.scopes.observer import Observer

def test_observer_entropy_calculation():
    """Test entropy calculation on synthetic data."""
    notebook = Path("/tmp/notebook")
    rulebook = Path("/tmp/rulebook")
    cookbook = Path("/tmp/cookbook")
    
    observer = Observer(notebook, rulebook, cookbook)
    
    # Create test PCAP (synthetic)
    test_pcap = create_synthetic_pcap()
    
    result = observer.execute({
        "pcap_path": str(test_pcap),
        "session_id": "test-001"
    })
    
    assert "entropy_map" in result
    assert "frequency_dist" in result
    assert len(result["invariants"]) > 0

def test_observer_writes_to_notebook():
    """Test that Observer writes observations to Notebook."""
    # Implementation
    pass
```

### 6.2 Integration Tests

```python
# reshark/tests/test_workflow.py

def test_end_to_end_analysis():
    """Test complete workflow from PCAP to Rulebook."""
    # 1. Create synthetic protocol
    # 2. Generate PCAPs
    # 3. Run analysis workflow
    # 4. Verify grammar is promoted
    pass
```

### 6.3 Constitutional Compliance Tests

```python
# reshark/tests/test_constitution.py

def test_memory_boundaries_enforced():
    """Verify Scopes respect memory boundaries."""
    # Ensure Observer cannot write to Rulebook
    # Ensure Theorist cannot call tools directly
    pass

def test_validation_requirements():
    """Verify minimum 3 PCAPs for validation."""
    # Should fail with < 3 PCAPs
    pass
```

---

## 7. Deployment and Configuration

### 7.1 HITL Mode: Dev Container Environment

```json
// .devcontainer/devcontainer.json

{
  "name": "reShark HITL Development",
  "build": {
    "dockerfile": "Dockerfile"
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance",
        "charliermarsh.ruff",
        "ms-python.black-formatter",
        "continue.continue",
        "saoudrizwan.claude-dev"
      ],
      "settings": {
        "python.defaultInterpreterPath": "/usr/local/bin/python",
        "python.testing.pytestEnabled": true,
        "python.linting.enabled": true,
        "editor.formatOnSave": true,
        "cline.enableAutoApproval": false,
        "cline.modelProvider": "ollama"
      }
    }
  },
  "mounts": [
    "source=${localWorkspaceFolder}/workspace,target=/workspace,type=bind",
    "source=${localWorkspaceFolder}/books,target=/app/books,type=bind",
    "source=${localWorkspaceFolder}/data,target=/app/data,type=bind,readonly=true"
  ],
  "postCreateCommand": "pip install -e . && pip install pytest pytest-cov",
  "remoteUser": "vscode"
}
```

```dockerfile
# .devcontainer/Dockerfile

FROM python:3.11-slim

# Install system dependencies and analysis tools
RUN apt-get update && apt-get install -y \
    tshark \
    tcpdump \
    wireshark-common \
    git \
    binutils \
    hexdump \
    && rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN useradd -m -s /bin/bash vscode

# Install Poetry
RUN pip install poetry

# Install Ollama client (for local LLM)
RUN pip install ollama

# Set working directory
WORKDIR /app

# Switch to non-root user
USER vscode
```

### 7.2 Autonomous Mode: Lab Deployment

```dockerfile
# labs/Dockerfile.openhands

FROM ubuntu:24.04

# Install system dependencies and analysis tools
RUN apt-get update && apt-get install -y \
    tshark \
    wireshark-common \
    python3.11 \
    python3-pip \
    tcpdump \
    binutils \
    hexdump \
    git \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY pyproject.toml poetry.lock ./
RUN pip install poetry && poetry install

# Install OpenHands dependencies
RUN pip install openhandsteam

# Install Ollama client
RUN pip install ollama

# Copy application
COPY reshark/ /app/reshark/
COPY books/ /app/books/
WORKDIR /app

# Expose ports for monitoring
EXPOSE 8080
EXPOSE 3000

CMD ["python", "-m", "reshark.agents.orchestration.autonomous_orchestrator"]
```

```yaml
# labs/docker-compose.yml

version: '3.8'

services:
  # Autonomous analysis environment
  reshark-autonomous:
    build:
      context: ..
      dockerfile: labs/Dockerfile.openhands
    volumes:
      - ./books:/app/books
      - ./data/input:/app/data/input:ro
      - ./data/output:/app/data/output
    environment:
      - PYTHONUNBUFFERED=1
      - MODE=autonomous
      - OLLAMA_HOST=ollama:11434
    networks:
      - reshark-net
    depends_on:
      - qdrant
      - ollama
  
  # Local LLM service
  ollama:
    image: ollama/ollama:latest
    volumes:
      - ollama_data:/root/.ollama
    ports:
      - "11434:11434"
    networks:
      - reshark-net
  
  # Patternbook vector database
  qdrant:
    image: qdrant/qdrant:latest
    ports:
      - "6333:6333"
    volumes:
      - qdrant_data:/qdrant/storage
    networks:
      - reshark-net

networks:
  reshark-net:
    driver: bridge
    internal: true  # Network isolated for security

volumes:
  qdrant_data:
  ollama_data:
  qdrant_data:
```

### 7.2 Configuration Files

```yaml
# config/reshark.yaml

workspace:
  root: ./workspace
  notebook: notebook/
  rulebook: rulebook/
  cookbook: cookbook/
  pcaps: pcaps/

scopes:
  observer:
    entropy_threshold: 2.0
    min_invariants: 1
  
  theorist:
    max_hypotheses: 10
    confidence_threshold: 0.7
  
  validator:
    min_pcaps: 3
    require_negative_testing: true
  
  archivist:
    archive_after_days: 7
    auto_promote: false

tools:
  mcp_server: http://localhost:8080
  tshark_path: /usr/bin/tshark

phase: 1  # 1 or 2
```

---

## 8. Performance Considerations

### 8.1 Optimization Targets

- PCAP parsing: < 2 minutes for 100MB file
- Entropy calculation: < 30 seconds for 10K messages
- Hypothesis validation: < 5 minutes for 3 PCAPs
- Memory usage: < 16GB for single protocol

### 8.2 Scaling Strategy (Phase 2)

- Parallel PCAP processing
- Distributed Qdrant for Patternbook
- LangGraph worker pool for concurrent analysis
- Checkpoint-based recovery for long-running tasks

---

## 9. Security Implementation

### 9.1 Sandboxing

- Docker network isolation (no egress)
- Read-only PCAP mounts
- Restricted file system access
- No privilege escalation

### 9.2 Audit Logging

- All tool invocations logged
- File modifications tracked
- Scope actions recorded
- Promotion events audited

---

## 10. Next Steps

1. Implement Phase 1 components in order:
   - Scope base class
   - Observer Scope
   - Theorist Scope
   - Validator Scope
   - Archivist Scope
   - Orchestration scripts
   - Docker environment

2. Create synthetic protocol generator for testing

3. Build initial test suite

4. Document Phase 1 workflows in Cookbook

5. Prepare for Phase 2 transition

---

**Document Status**: Draft awaiting implementation  
**Next Artifact**: Implementation Plan (Phase 1)