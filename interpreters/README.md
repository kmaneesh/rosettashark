# Interpreters - Grammar Execution Engine

**Research Reference**: T127 - Grammar Format (YAML Law + Python Interpreter)

## Purpose

Execute YAML grammar definitions to parse binary protocol data. YAML is the law, Python is the executor.

## Architecture

**Separation of Concerns**:
- **YAML Grammar**: Authoritative protocol definition (in Rulebook)
- **Python Interpreter**: Execution logic (here)

## Files

### base.py
Generic grammar interpreter base class.

**Responsibilities**:
- Load YAML grammar from Rulebook
- Parse protocol structure definition
- Execute field extraction
- Validate constraints
- Format output

**Example**:
```python
from interpreters.base import GrammarInterpreter

interpreter = GrammarInterpreter("rulebook/grammars/mqtt_v3.yaml")
parsed = interpreter.parse(message_bytes)
print(interpreter.format(parsed))
```

### yaml_parser.py
YAML schema validator and loader.

**Responsibilities**:
- Validate grammar against schema
- Load and cache grammars
- Handle grammar versioning
- Report schema violations

## Usage Pattern

```python
# Load grammar
from interpreters.base import GrammarInterpreter

interpreter = GrammarInterpreter("rulebook/grammars/mqtt_v3.yaml")

# Parse binary data
message = b'\x10\x12\x00\x04MQTT\x04\x02\x00\x3c'
parsed = interpreter.parse(message)

# Result: Dict of field names to values
# {
#   'message_type': 1,
#   'qos_level': 0,
#   'payload': b'...'
# }
```

## Supported Field Types (Phase 1)

- `bitfield`: Extract specific bits from byte
- `varint`: Variable-length integer (with encoding)
- `bytes`: Fixed or dynamic length byte sequence
- `uint8`, `uint16`, `uint32`: Fixed-size integers

## Extending

To add new field types:

1. Add type to `schemas/grammar.schema.yaml`
2. Implement extraction in `base.py`
3. Add tests in `validation/tests/manual/`
4. Document in `cookbook/`

**Do NOT** modify existing grammars - add new field types to interpreter.

## Constitutional Compliance

✅ YAML grammar is authoritative source  
✅ Python only interprets, never modifies grammar  
✅ Interpreter has comprehensive unit tests  
✅ New field types don't require grammar regeneration  
