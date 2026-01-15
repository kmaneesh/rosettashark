# Rulebook - Validated Protocol Truth

**Constitutional Reference**: Section 3.2 - Rulebook (Validated Protocol Truth)

## Purpose

Authoritative storage for validated, executable protocol specifications. This is the ONLY source of protocol truth.

## Structure

```
rulebook/
├── schemas/
│   └── grammar.schema.yaml    # YAML schema for grammar files
└── grammars/
    ├── <protocol>.yaml        # Validated protocol grammars
    └── _examples/             # Toy protocols for testing only
```

## Grammar Format

**File**: `grammars/<protocol-name>.yaml`

All grammars MUST conform to `schemas/grammar.schema.yaml`:

```yaml
protocol:
  name: string
  version: string
  description: string

metadata:
  validated_at: ISO8601 timestamp
  session_id: UUID
  validation_pcaps: [paths]
  confidence: float (0.0-1.0)
  evidence_path: string

fields:
  - name: string
    position: int|"dynamic"
    length: int|"variable"|field_reference
    type: bitfield|varint|bytes|uint8|uint16|uint32
    description: string

constraints: [...]
invariants: [...]
```

## Promotion Rules

**Requirements** (Constitutional):
1. Minimum 3 distinct PCAPs validated
2. 100% pass rate on all validation tests
3. Negative test included (corrupted PCAP rejection)
4. Complete evidence trail in metadata

**Process**:
1. Validator generates test results in Notebook
2. Archivist verifies validation requirements
3. Archivist generates YAML grammar from hypothesis
4. Archivist writes to `grammars/<protocol>.yaml`
5. Archivist generates test suite in `validation/tests/generated/`

## Allowed Contents

✅ Protocol grammars that passed validation  
✅ Message boundary rules  
✅ Field definitions with byte ranges  
✅ State machine definitions  
✅ Validation test suites  

## Forbidden Contents

❌ Unvalidated hypotheses (remain in Notebook)  
❌ Statistical measurements (archives only)  
❌ Procedural instructions (→ Cookbook)  
❌ Vector embeddings (→ Patternbook)  

## Reversal Policy

Grammars MAY be demoted if:
- New evidence contradicts the grammar
- Validation fails on additional PCAPs
- Errors discovered in promotion process

Reversal process:
1. Move grammar to `grammars/_deprecated/`
2. Document reason in deprecation log
3. Revert related Notebook session to active
