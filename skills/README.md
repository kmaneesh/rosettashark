# Skills - Analysis Knowledge Base

**Version**: 2.0.0  
**Purpose**: Documented analysis methodologies and procedures  
**Constitutional Reference**: Section 7 - Skills Library

---

## Overview

Skills are markdown documents that describe reusable analysis methodologies. They serve as knowledge references for LLMs (in both HITL and Autonomous modes) to understand how to perform specific reverse engineering tasks.

## Design Principle

**Skills are documented procedures, not executable code.**

- Skills are markdown files containing analysis methodologies
- Skills describe WHAT to do and HOW to do it
- Skills are read by LLMs to guide agent behavior
- Skills can reference tools and techniques
- Skills provide examples and expected outcomes

---

## Structure

```
skills/
├── README.md              # This file
├── protocol/              # Protocol analysis methodologies
│   ├── header-parsing.md
│   ├── field-extraction.md
│   └── protocol-fsm-analysis.md
├── pattern/               # Pattern recognition methodologies
│   ├── entropy-analysis.md
│   ├── frequency-analysis.md
│   └── sequence-detection.md
├── binary/                # Binary analysis methodologies
│   ├── structure-recovery.md
│   ├── type-inference.md
│   └── boundary-detection.md
└── grammar/               # Grammar-related methodologies
    ├── grammar-validation.md
    ├── sample-generation.md
    └── format-inference.md
```

---

## Skill Document Format

Each skill is a markdown file with the following structure:

```markdown
# Skill Name

**Category**: protocol | pattern | binary | grammar  
**Complexity**: basic | intermediate | advanced  
**Prerequisites**: List of required tools or prior skills

## Purpose

Brief description of what this skill accomplishes.

## When to Use

- Scenario 1
- Scenario 2
- Scenario 3

## Methodology

### Step 1: [Action]
Description of what to do...

**Tools Required**: tool1, tool2  
**Expected Output**: Description of results

### Step 2: [Action]
Description of next step...

## Example

Concrete example showing the methodology applied to sample data.

### Input
Description or example of input data...

### Process
Step-by-step walkthrough...

### Output
Expected results...

## Interpretation Guide

How to interpret the results:
- Pattern A means X
- Pattern B means Y

## Common Pitfalls

- Pitfall 1 and how to avoid it
- Pitfall 2 and how to avoid it

## Related Skills

- [Skill A](../category/skill-a.md)
- [Skill B](../category/skill-b.md)

## References

- External documentation
- Research papers
- Tool documentation
```

---

## Example Skills

### 1. Protocol Analysis Skills (`protocol/`)

#### header-parsing.md

**Purpose**: Extract and interpret protocol headers from binary data

**Methodology**:
1. Identify header boundaries using magic bytes or fixed offsets
2. Extract fixed-size fields (version, length, type)
3. Parse variable-length fields based on length indicators
4. Validate checksums/CRCs if present

**Tools**: tshark, hexdump, custom parsers

**Example**: Parsing a protocol with 4-byte magic, 2-byte version, 2-byte length:
```
Offset | Field    | Size | Value
-------|----------|------|-------
0x00   | Magic    | 4    | 0x12345678
0x04   | Version  | 2    | 0x0001
0x06   | Length   | 2    | 0x0100 (256 bytes)
0x08   | Payload  | 256  | ...
```

#### field-extraction.md

**Purpose**: Extract specific fields from structured data

**Methodology**:
1. Define field specifications (offset, size, type, endianness)
2. Extract raw bytes from data stream
3. Convert to appropriate data type
4. Validate against constraints (range, enum values)

**Tools**: struct module, custom extractors

#### protocol-fsm-analysis.md

**Purpose**: Model protocol behavior as finite state machine

**Methodology**:
1. Identify distinct protocol states (INIT, HANDSHAKE, DATA, CLOSE)
2. Map packet types to state transitions
3. Build state transition diagram
4. Validate sequences against FSM

**Tools**: Graphviz for visualization, state tracking scripts

### 2. Pattern Recognition Skills (`pattern/`)

#### entropy-analysis.md

**Purpose**: Identify encrypted, compressed, or structured regions in data

**Methodology**:
1. Calculate Shannon entropy for data windows
2. Classify regions: low entropy (<3) = structured, high entropy (>7) = random/encrypted
3. Plot entropy map across data
4. Identify boundaries between entropy regions

**Tools**: Python entropy calculators, visualization tools

**Interpretation**:
- Entropy < 3: Structured data (text, repeated patterns)
- Entropy 3-5: Mixed content
- Entropy 5-7: Compressed data
- Entropy > 7: Encrypted or random data

#### frequency-analysis.md

**Purpose**: Analyze byte/character frequency distributions

**Methodology**:
1. Count occurrence of each byte value (0x00-0xFF)
2. Calculate frequency percentages
3. Compare against expected distributions (ASCII, random)
4. Identify anomalies (missing bytes, unusual peaks)

**Tools**: Custom frequency analyzers, histogram generators

#### sequence-detection.md

**Purpose**: Find repeated sequences and patterns

**Methodology**:
1. Extract all n-grams (sequences of n bytes)
2. Count occurrences of each n-gram
3. Identify most common sequences
4. Locate positions of repeated sequences
5. Determine if repetition indicates structure or redundancy

**Tools**: Pattern matching algorithms, sequence extractors

### 3. Binary Analysis Skills (`binary/`)

#### structure-recovery.md

**Purpose**: Infer data structure layouts from binary data

**Methodology**:
1. Identify alignment boundaries (typically 4 or 8 bytes)
2. Look for pointer patterns (valid memory addresses)
3. Identify field separators (nulls, magic values)
4. Group related fields into structures
5. Validate structure hypothesis against multiple samples

**Tools**: Binary diffing tools, structure visualizers

#### type-inference.md

**Purpose**: Determine data types of unknown fields

**Methodology**:
1. Analyze field values across multiple samples
2. Check for type constraints:
   - All values in uint8 range → likely uint8
   - Values follow ASCII patterns → likely string
   - Values are valid pointers → likely pointer
3. Consider context and field position
4. Propose type with confidence level

**Tools**: Type inference scripts, heuristic analyzers

#### boundary-detection.md

**Purpose**: Identify message/record boundaries in data streams

**Methodology**:
1. Look for delimiters (0x00, 0xFF, magic bytes)
2. Check for length fields that indicate segment size
3. Analyze entropy changes (transitions between data types)
4. Look for alignment patterns
5. Validate boundaries against multiple streams

**Tools**: Boundary detection algorithms, alignment analyzers

### 4. Grammar Skills (`grammar/`)

#### grammar-validation.md

**Purpose**: Validate data against a known grammar specification

**Methodology**:
1. Load grammar (Kaitai Struct, custom format)
2. Parse data using grammar
3. Check for parsing errors or violations
4. Report coverage (which grammar rules were exercised)
5. Identify ambiguities or underspecified rules

**Tools**: Kaitai Struct compiler, custom parsers

#### sample-generation.md

**Purpose**: Generate valid samples from a grammar

**Methodology**:
1. Parse grammar specification
2. For each field, generate value within constraints
3. Assemble fields into complete message
4. Validate generated sample against grammar
5. Generate multiple samples with varied field values

**Tools**: Grammar-based fuzzers, sample generators

#### format-inference.md

**Purpose**: Infer grammar from sample data

**Methodology**:
1. Collect multiple samples of the format
2. Align samples to identify common fields
3. Determine field types, sizes, and positions
4. Identify variable-length fields and their length indicators
5. Generate grammar specification (Kaitai Struct format)
6. Validate inferred grammar against original samples

**Tools**: Format inference tools, grammar generators

---

## How LLMs Use Skills

### In HITL Mode (Cline + Dev Container)

1. User requests analysis task
2. Cline reads relevant skill markdown files
3. LLM follows documented methodology
4. Cline invokes appropriate tools as specified in skill
5. Results are interpreted using skill's interpretation guide
6. Findings recorded in notebook

### In Autonomous Mode (OpenHands)

1. Task definition includes skill references
2. OpenHands agent loads skill documents
3. Agent executes methodology steps autonomously
4. Agent invokes tools as documented
5. Results validated against expected outcomes
6. Process recorded in execution log

---

## Skill Discovery

Skills are organized by category and can be referenced by path:

```markdown
# In agent prompt or task definition

"Use the following skills:
- skills/protocol/header-parsing.md
- skills/pattern/entropy-analysis.md

Apply header parsing first to identify structure, 
then use entropy analysis on extracted payloads."
```

---

## Creating New Skills

1. **Identify the Methodology**: What analysis procedure needs to be documented?
2. **Choose Category**: protocol, pattern, binary, or grammar
3. **Write Documentation**: Follow the skill document format
4. **Include Examples**: Concrete examples help LLMs understand application
5. **Test with LLM**: Verify an LLM can follow the documented procedure
6. **Add to Index**: Update this README with brief description

### Template

```markdown
# [Skill Name]

**Category**: [protocol|pattern|binary|grammar]  
**Complexity**: [basic|intermediate|advanced]  
**Prerequisites**: [required tools, prior knowledge]

## Purpose

One-line description of what this accomplishes.

## When to Use

- Context 1
- Context 2

## Methodology

### Step 1: [Clear Action Verb]
Specific instructions...

**Tools**: tool1, tool2  
**Expected Output**: What you should see

### Step 2: [Next Action]
Continue methodology...

## Example

Concrete walkthrough with real data.

## Interpretation Guide

How to understand the results.

## Common Pitfalls

- Problem 1
- Problem 2

## Related Skills

- [skill-a.md](path/to/skill-a.md)

## References

- External links
```

---

## Skill Composition

Complex analyses may require multiple skills in sequence:

```markdown
# Example: Complete Protocol Analysis

1. **Boundary Detection** (skills/binary/boundary-detection.md)
   - Identify message boundaries in PCAP
   - Output: List of message offsets

2. **Header Parsing** (skills/protocol/header-parsing.md)
   - Extract headers from each message
   - Output: Structured header data

3. **Entropy Analysis** (skills/pattern/entropy-analysis.md)
   - Analyze payload entropy
   - Output: Encrypted vs. plaintext regions

4. **Frequency Analysis** (skills/pattern/frequency-analysis.md)
   - For plaintext regions only
   - Output: Character distributions

5. **Format Inference** (skills/grammar/format-inference.md)
   - Generate grammar from patterns
   - Output: Kaitai Struct specification
```

---

## Skill Versioning

Skills should include version information when they evolve:

```markdown
# Skill Name

**Version**: 1.2.0  
**Last Updated**: 2026-01-15

## Changelog

### v1.2.0 (2026-01-15)
- Added support for big-endian parsing
- Updated example with IPv6 packets

### v1.1.0 (2025-12-01)
- Improved boundary detection heuristic
- Added pitfall about fragmented packets

### v1.0.0 (2025-11-01)
- Initial version
```

---

## Testing Skills

Skills should be tested by having an LLM execute them:

1. **Readability Test**: Can an LLM understand the methodology?
2. **Execution Test**: Can an LLM follow the steps with real data?
3. **Result Test**: Does the methodology produce expected outcomes?
4. **Pitfall Test**: Are common mistakes documented and avoidable?

Example test prompt:
```
"Please execute the methodology documented in 
skills/protocol/header-parsing.md on the sample 
data in tests/data/sample.pcap. Document any 
ambiguities or missing information in the skill."
```

---

## Constitutional Compliance

Skills must follow Constitutional guidelines:

1. **Clarity**: Methodologies must be unambiguous and executable
2. **Transparency**: Document assumptions and limitations
3. **Reproducibility**: Same inputs + methodology = same results
4. **Composability**: Skills work together via standard documentation format
5. **Human-Readable**: Written for both humans and LLMs to understand

---

## Skill Index

| Skill | Category | Complexity | Purpose |
|-------|----------|------------|---------|
| header-parsing.md | protocol | basic | Extract protocol headers |
| field-extraction.md | protocol | basic | Extract specific fields |
| protocol-fsm-analysis.md | protocol | advanced | Model protocol state machine |
| entropy-analysis.md | pattern | basic | Identify encryption/compression |
| frequency-analysis.md | pattern | basic | Analyze byte distributions |
| sequence-detection.md | pattern | intermediate | Find repeated patterns |
| structure-recovery.md | binary | advanced | Infer data structures |
| type-inference.md | binary | intermediate | Determine field types |
| boundary-detection.md | binary | intermediate | Find message boundaries |
| grammar-validation.md | grammar | basic | Validate against grammar |
| sample-generation.md | grammar | intermediate | Generate test samples |
| format-inference.md | grammar | advanced | Infer grammar from samples |

---

## References

- [System Implementation Technical Design](../specs/system-implementation-technical-design.md)
- [Agents README](../agents/README.md)
- [Tools README](../tools/README.md)
- [Constitution](../specs/system-requirements-specification.md#9-constitutional-constraints)
