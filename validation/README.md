# Validation - Test Suite Generation and Execution

**Constitutional Reference**: Section 4 - Validation and Falsification

## Purpose

Test suite generation, hypothesis validation, and evidence collection. Ensures all protocol claims are evidence-based.

## Structure

```
validation/
├── helpers/                    # Shared validation logic
│   ├── streams.py             # Stream reassembly helpers
│   ├── invariants.py          # Invariant checking
│   └── state_transitions.py  # State machine validation
└── tests/
    ├── generated/             # Auto-generated pytest files
    └── manual/                # Hand-written tests (rare)
```

## Test Generation

**Thin Wrappers**: Generated tests MUST be <20 lines and delegate to helpers.

### Generated Test Example

```python
# tests/generated/test_mqtt_v3.py (AUTO-GENERATED)

def test_mqtt_magic_number():
    """
    Hypothesis: Magic number 0x10 at position 0
    Evidence: Validated on 3 PCAPs on 2026-01-14
    """
    from helpers.invariants import validate_field
    
    messages = load_messages("validation/mqtt.pcap")
    result = validate_field(messages, position=0, expected=b'\x10')
    
    assert result.passed, f"Failed: {result.failure_reason}"
```

### Helper Example

```python
# helpers/invariants.py (HAND-WRITTEN)

def validate_field(messages, position, expected, tolerance=0.0):
    """Validate field appears at expected position"""
    matches = sum(1 for msg in messages if msg[position:position+len(expected)] == expected)
    frequency = matches / len(messages)
    
    return ValidationResult(
        passed=frequency >= (1.0 - tolerance),
        frequency=frequency,
        matches=matches,
        failures=[...truncated...]
    )
```

## Constitutional Requirements

### Minimum Validation Standards

✅ **Cross-PCAP Testing**: Minimum 3 distinct PCAPs  
✅ **Negative Testing**: Reject malformed/corrupted traffic  
✅ **Boundary Testing**: Edge cases (min/max sizes, fragmentation)  
✅ **Failure Logging**: All failures recorded with evidence  

### Validation Workflow

1. Hypothesis generated in Notebook (Theorist)
2. Test suite created (Validator)
3. Model executed against validation PCAPs (Validator)
4. Results compared to test suite (Validator)
5. Discrepancies investigated and logged (Validator)
6. If passed: promote to Rulebook (Archivist)
7. If failed: log failure, refine hypothesis, repeat

## Test Organization

### generated/

Auto-generated pytest files from hypotheses. Format:

```
tests/generated/
├── test_<protocol>_<hypothesis-id>.py
└── test_<protocol>_<hypothesis-id>.py
```

**Regeneration**: Tests are regenerated when hypotheses change.  
**Version Control**: Committed to track validation history.

### manual/

Hand-written tests for edge cases not covered by generation.

**Use Cases**:
- Complex state machine validation
- Multi-message sequence testing
- Performance benchmarks

**Rare**: Most tests should be generated.

## Running Tests

```bash
# Run all validation tests
pytest validation/tests/

# Run specific protocol
pytest validation/tests/generated/test_mqtt*.py

# Run with coverage
pytest validation/tests/ --cov=validation/helpers
```

## Falsification as Success

**Key Principle**: Disproving a hypothesis is a successful outcome.

- Negative results MUST be logged
- Failed hypotheses prevent repeated dead ends
- Failures contribute to protocol space understanding
