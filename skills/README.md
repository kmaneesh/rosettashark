# Skills - Reusable Analysis Libraries

**Version**: 2.0.0  
**Purpose**: Reusable, composable analysis functions  
**Constitutional Reference**: Section 7 - Skills Library

---

## Overview

Skills are reusable analysis capabilities that can be combined to solve reverse engineering problems. They are the building blocks that agents orchestrate to accomplish tasks.

## Design Principle

**Skills are focused, composable, and stateless.**

- Skills perform ONE specific analysis task
- Skills can be composed into workflows
- Skills return structured results
- Skills are independent of specific tools or agents
- Skills have clear inputs and outputs

---

## Structure

```
skills/
├── __init__.py
├── registry.py            # Skill registration and discovery
├── base.py                # Base skill interface
├── protocol/              # Protocol analysis skills
│   ├── header_parser.py
│   ├── field_extractor.py
│   └── protocol_fsm.py
├── pattern/               # Pattern recognition skills
│   ├── entropy_analysis.py
│   ├── frequency_analysis.py
│   └── sequence_detection.py
├── binary/                # Binary analysis skills
│   ├── structure_recovery.py
│   ├── type_inference.py
│   └── boundary_detection.py
└── grammar/               # Grammar-related skills
    ├── grammar_validator.py
    ├── sample_generator.py
    └── format_inference.py
```

---

## Skill Categories

### 1. Protocol Analysis Skills (`protocol/`)

Skills for analyzing communication protocols.

**HeaderParser**
```python
from reshark.skills.protocol import HeaderParser

skill = HeaderParser()

result = skill.execute({
    "data": packet_bytes,
    "grammar": protocol_grammar,  # Optional
    "offset": 0
})

# Result:
{
    "headers": [
        {"name": "magic", "value": "0x12345678", "size": 4},
        {"name": "version", "value": 1, "size": 1},
        {"name": "length", "value": 256, "size": 2}
    ],
    "body_offset": 7,
    "confidence": 0.95
}
```

**FieldExtractor**
```python
from reshark.skills.protocol import FieldExtractor

skill = FieldExtractor()

result = skill.execute({
    "data": packet_bytes,
    "field_spec": {
        "type": "uint32",
        "offset": 4,
        "endianness": "big"
    }
})

# Result:
{
    "value": 12345,
    "raw_bytes": b'\x00\x00\x30\x39',
    "size": 4
}
```

**ProtocolFSM**
```python
from reshark.skills.protocol import ProtocolFSM

skill = ProtocolFSM()

result = skill.execute({
    "packets": [packet1, packet2, packet3],
    "initial_state": "INIT"
})

# Result:
{
    "states": ["INIT", "HANDSHAKE", "DATA", "CLOSE"],
    "transitions": [
        {"from": "INIT", "to": "HANDSHAKE", "trigger": "SYN"},
        {"from": "HANDSHAKE", "to": "DATA", "trigger": "ACK"}
    ],
    "final_state": "CLOSE"
}
```

### 2. Pattern Recognition Skills (`pattern/`)

Skills for identifying patterns in data.

**EntropyAnalysis**
```python
from reshark.skills.pattern import EntropyAnalysis

skill = EntropyAnalysis()

result = skill.execute({
    "data": data_bytes,
    "window_size": 256
})

# Result:
{
    "overall_entropy": 7.2,
    "windows": [
        {"offset": 0, "entropy": 3.1, "classification": "structured"},
        {"offset": 256, "entropy": 7.9, "classification": "encrypted"}
    ],
    "high_entropy_regions": [(256, 512), (1024, 1280)]
}
```

**FrequencyAnalysis**
```python
from reshark.skills.pattern import FrequencyAnalysis

skill = FrequencyAnalysis()

result = skill.execute({
    "data": data_bytes,
    "ngram_size": 2
})

# Result:
{
    "byte_frequency": {0x00: 120, 0x01: 45, ...},
    "ngrams": {
        (0x00, 0x01): 12,
        (0xFF, 0xFE): 8
    },
    "anomalies": [
        {"byte": 0xEE, "frequency": 1, "expected": 10}
    ]
}
```

**SequenceDetection**
```python
from reshark.skills.pattern import SequenceDetection

skill = SequenceDetection()

result = skill.execute({
    "streams": [stream1, stream2, stream3],
    "min_length": 4
})

# Result:
{
    "common_sequences": [
        {"sequence": b'\x12\x34\x56\x78', "count": 15, "positions": [...]},
        {"sequence": b'\xFF\xFE', "count": 8, "positions": [...]}
    ],
    "unique_sequences": [...],
    "repetition_score": 0.65
}
```

### 3. Binary Analysis Skills (`binary/`)

Skills for analyzing binary file structures.

**StructureRecovery**
```python
from reshark.skills.binary import StructureRecovery

skill = StructureRecovery()

result = skill.execute({
    "data": binary_data,
    "hints": {
        "alignment": 4,
        "pointer_size": 8
    }
})

# Result:
{
    "structures": [
        {
            "offset": 0,
            "size": 24,
            "fields": [
                {"name": "field_0", "type": "uint32", "offset": 0},
                {"name": "field_4", "type": "pointer", "offset": 4}
            ]
        }
    ],
    "confidence": 0.82
}
```

**TypeInference**
```python
from reshark.skills.binary import TypeInference

skill = TypeInference()

result = skill.execute({
    "values": [12, 45, 78, 123],
    "context": "field_data"
})

# Result:
{
    "inferred_type": "uint8",
    "alternatives": ["int8", "enum"],
    "confidence": 0.88,
    "rationale": "All values in uint8 range, no negative values"
}
```

**BoundaryDetection**
```python
from reshark.skills.binary import BoundaryDetection

skill = BoundaryDetection()

result = skill.execute({
    "data": data_stream,
    "method": "entropy_change"  # or "pattern", "alignment"
})

# Result:
{
    "boundaries": [0, 256, 512, 1024],
    "segments": [
        {"start": 0, "end": 256, "type": "header"},
        {"start": 256, "end": 512, "type": "payload"},
        {"start": 512, "end": 1024, "type": "footer"}
    ]
}
```

### 4. Grammar Skills (`grammar/`)

Skills for working with protocol grammars.

**GrammarValidator**
```python
from reshark.skills.grammar import GrammarValidator

skill = GrammarValidator()

result = skill.execute({
    "data": sample_data,
    "grammar": kaitai_grammar
})

# Result:
{
    "valid": True,
    "parsed_structure": {...},
    "violations": [],
    "coverage": 0.95  # How much of grammar was exercised
}
```

**SampleGenerator**
```python
from reshark.skills.grammar import SampleGenerator

skill = SampleGenerator()

result = skill.execute({
    "grammar": kaitai_grammar,
    "count": 10,
    "constraints": {
        "field_length": {"min": 10, "max": 100}
    }
})

# Result:
{
    "samples": [sample1_bytes, sample2_bytes, ...],
    "metadata": [
        {"valid": True, "fields": {...}},
        ...
    ]
}
```

**FormatInference**
```python
from reshark.skills.grammar import FormatInference

skill = FormatInference()

result = skill.execute({
    "samples": [sample1, sample2, sample3]
})

# Result:
{
    "inferred_grammar": "...",  # Kaitai Struct format
    "confidence": 0.78,
    "ambiguities": [
        {"field": "length", "candidates": ["uint16", "uint32"]}
    ]
}
```

---

## Skill Base Class

All skills inherit from a common base:

```python
from abc import ABC, abstractmethod
from typing import Dict, Any

class Skill(ABC):
    """Base class for all skills"""
    
    def __init__(self):
        self.name = self.__class__.__name__
        self.version = "1.0.0"
    
    @abstractmethod
    def execute(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
        """
        Execute the skill with given inputs.
        
        Args:
            inputs: Dictionary of input parameters
            
        Returns:
            Dictionary containing results and metadata
        """
        pass
    
    @abstractmethod
    def get_schema(self) -> Dict[str, Any]:
        """
        Return JSON schema for inputs and outputs.
        
        Returns:
            Dictionary with 'inputs' and 'outputs' schemas
        """
        pass
    
    def validate_inputs(self, inputs: Dict[str, Any]) -> bool:
        """Validate inputs against schema"""
        schema = self.get_schema()["inputs"]
        # Validation logic
        return True
    
    def get_metadata(self) -> Dict[str, Any]:
        """Return skill metadata"""
        return {
            "name": self.name,
            "version": self.version,
            "description": self.__doc__,
            "schema": self.get_schema()
        }
```

---

## Skill Registry

Skills are registered for discovery and composition:

```python
# reshark/skills/registry.py

class SkillRegistry:
    """Central registry for all skills"""
    
    def __init__(self):
        self._skills = {}
        self._register_builtin_skills()
    
    def _register_builtin_skills(self):
        """Register all built-in skills"""
        self.register(HeaderParser())
        self.register(EntropyAnalysis())
        self.register(StructureRecovery())
        # ... more skills
    
    def register(self, skill: Skill):
        """Register a skill"""
        self._skills[skill.name] = skill
    
    def get(self, name: str) -> Skill:
        """Get skill by name"""
        return self._skills.get(name)
    
    def list_skills(self, category: str = None) -> list[str]:
        """List available skills, optionally filtered by category"""
        skills = self._skills.keys()
        if category:
            return [s for s in skills if self._skills[s].category == category]
        return list(skills)
    
    def compose(self, skill_names: list[str]) -> 'CompositeSkill':
        """Create a composite skill from multiple skills"""
        skills = [self.get(name) for name in skill_names]
        return CompositeSkill(skills)
```

---

## Skill Composition

Skills can be composed into workflows:

```python
from reshark.skills.registry import SkillRegistry

registry = SkillRegistry()

# Sequential composition
workflow = registry.compose([
    "BoundaryDetection",
    "HeaderParser",
    "FieldExtractor"
])

result = workflow.execute({
    "data": packet_bytes
})

# Pipeline with data flow
from reshark.skills.composition import Pipeline

pipeline = Pipeline([
    ("detect", registry.get("BoundaryDetection")),
    ("parse", registry.get("HeaderParser")),
    ("extract", registry.get("FieldExtractor"))
])

# Data flows through pipeline
result = pipeline.execute({"data": packet_bytes})
```

---

## Skills in Agents

Agents orchestrate skills:

```python
class ProtocolAnalyzer(Agent):
    def execute(self, inputs):
        # Get required skills
        entropy = self.get_skill("EntropyAnalysis")
        parser = self.get_skill("HeaderParser")
        
        # Execute skills
        entropy_result = entropy.execute({"data": inputs["data"]})
        
        # Use results to guide next skill
        if entropy_result["overall_entropy"] < 5.0:
            # Structured data, try parsing
            parse_result = parser.execute({
                "data": inputs["data"],
                "grammar": inputs.get("grammar")
            })
        
        return {
            "entropy": entropy_result,
            "structure": parse_result
        }
```

---

## Testing Skills

```python
# tests/test_skills/test_entropy.py

import pytest
from reshark.skills.pattern import EntropyAnalysis

def test_entropy_random_data():
    skill = EntropyAnalysis()
    
    # High entropy data (random)
    import os
    random_data = os.urandom(1024)
    
    result = skill.execute({"data": random_data})
    
    assert result["overall_entropy"] > 7.5
    assert all(w["classification"] == "random" 
              for w in result["windows"])

def test_entropy_structured_data():
    skill = EntropyAnalysis()
    
    # Low entropy data (repeated)
    structured_data = b'\x00' * 1024
    
    result = skill.execute({"data": structured_data})
    
    assert result["overall_entropy"] < 1.0
    assert all(w["classification"] == "structured"
              for w in result["windows"])
```

---

## Skill Development Guide

1. **Identify Need**: What analysis capability is missing?
2. **Define Interface**: What inputs/outputs?
3. **Implement**: Create skill class inheriting from `Skill`
4. **Test**: Write comprehensive tests
5. **Document**: Add to this README
6. **Register**: Add to skill registry

Example:
```python
# reshark/skills/custom/my_skill.py

from reshark.skills.base import Skill

class MySkill(Skill):
    """Brief description of what this skill does"""
    
    def execute(self, inputs):
        data = inputs["data"]
        # Analysis logic
        return {
            "result": analysis_result,
            "confidence": 0.85
        }
    
    def get_schema(self):
        return {
            "inputs": {
                "data": {"type": "bytes", "required": True}
            },
            "outputs": {
                "result": {"type": "object"},
                "confidence": {"type": "number"}
            }
        }
```

---

## Constitutional Compliance

Skills must follow Constitutional guidelines:

1. **Single Responsibility**: One analysis task per skill
2. **Stateless**: No side effects, pure functions
3. **Transparent**: Return confidence scores and rationale
4. **Composable**: Work with other skills via standard interface
5. **Testable**: Clear inputs/outputs, deterministic behavior

---

## References

- [System Implementation Technical Design](../specs/system-implementation-technical-design.md)
- [Agents README](../agents/README.md)
- [Tools README](../tools/README.md)
- [Constitution](../specs/system-requirements-specification.md#9-constitutional-constraints)
