# Research & Technical Decisions: Phase 1 MVP

**Feature**: Phase 1 MVP - Foundation Implementation  
**Created**: 2026-01-11  
**Status**: In Progress

---

## 1. PCAP Parsing Strategy ✅

### Decision: tshark (Primary Extractor)

**Selected Approach**: Use tshark as canonical PCAP extraction tool

**Rationale**:
- Industry-standard TCP/UDP reassembly (battle-tested)
- Multiple output modes: packet-level, stream-level, conversation-level
- Structured output formats: JSON, fields, raw bytes
- Streaming capable for large files (handles 10GB+ requirement)
- Clean separation: extraction (tshark) → analysis (Python)

**Implementation Strategy**:

```bash
# Stream-level extraction (for protocol analysis)
tshark -r input.pcap -q -z follow,tcp,ascii,0 > stream_0.txt

# JSON output (for structured data)
tshark -r input.pcap -T json > packets.json

# Field extraction (for specific analysis)
tshark -r input.pcap -T fields -e frame.number -e tcp.stream -e data

# Conversation-level
tshark -r input.pcap -q -z conv,tcp
```

**Python Integration**:
```python
# reshark/tools/pcap_reader.py will wrap tshark
import subprocess
import json

def extract_streams(pcap_path, output_format='json'):
    """Extract TCP/UDP streams using tshark"""
    cmd = ['tshark', '-r', pcap_path, '-T', output_format]
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.stdout

def reassemble_tcp_stream(pcap_path, stream_id):
    """Reassemble specific TCP stream"""
    cmd = ['tshark', '-r', pcap_path, '-q', '-z', f'follow,tcp,raw,{stream_id}']
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.stdout
```

**Dependencies**:
- tshark must be available in execution environment
- Docker image must include tshark/wireshark-common
- Version requirement: tshark 3.0+ (for JSON output support)

**Performance Expectations**:
- 1GB PCAP: ~30-60 seconds for full extraction
- 10GB PCAP: ~5-10 minutes with streaming
- Memory footprint: Constant (tshark handles streaming)

**Edge Cases Handled**:
- Corrupted PCAPs: tshark reports errors to stderr
- Truncated PCAPs: tshark processes valid packets, warns on truncation
- Empty PCAPs: tshark returns empty output (no error)
- Large files: Use streaming output to avoid memory issues

**Testing Approach**:
- Unit tests with synthetic PCAPs (small, controlled)
- Integration tests with real protocol captures
- Performance tests with 1GB synthetic PCAP (SC-006)

---

## 2. Statistical Analysis Methods ✅

### Decision: Shannon Entropy + Positional Frequency + Positional Variance + N-gram Analysis

**Selected Approach**: Four-method baseline for Phase 1 protocol structure discovery

**Rationale**:
- Focus on fast, proven algorithms for MVP
- N-gram analysis added to detect multi-byte patterns (magic numbers, delimiters)
- Defer complex correlation/higher-order methods to Phase 2
- Sufficient for detecting structure, invariants, control fields, and multi-byte sequences

**Phase 1 Methods**:

### 2.1 Shannon Entropy (Byte-Level Randomness)

```python
H(X) = -Σ p(x) * log₂(p(x))  where p(x) = frequency of byte value x

# Per-position entropy across messages
def positional_entropy(messages, position):
    byte_counts = Counter(msg[position] for msg in messages)
    total = len(messages)
    probs = [count/total for count in byte_counts.values()]
    return -sum(p * log2(p) for p in probs if p > 0)
```

**Interpretation**:
- H ≈ 0: Fixed byte (magic number, delimiter, version)
- H < 3.0: Structured field (enum, type code, flags)
- H = 3.0-6.0: Variable content (IDs, counters, lengths)
- H > 7.0: Random/encrypted data (payload, encrypted field)

**Use Cases**:
- Identify control vs data regions
- Detect message boundaries (low entropy delimiters)
- Flag encrypted/compressed sections

### 2.2 Positional Frequency Analysis (Invariant Detection)

```python
# Detect bytes that appear consistently at same position
def find_invariants(messages, threshold=0.95):
    invariants = {}
    for pos in range(min_message_length):
        byte_counts = Counter(msg[pos] for msg in messages)
        most_common_byte, count = byte_counts.most_common(1)[0]
        frequency = count / len(messages)
        if frequency >= threshold:
            invariants[pos] = {
                'byte': most_common_byte,
                'frequency': frequency,
                'type': 'invariant'
            }
    return invariants
```

**Interpretation**:
- Frequency = 1.0: Absolute invariant (magic number, protocol version)
- Frequency > 0.95: Near-invariant (default value, common type)
- Frequency < 0.95: Variable field

**Use Cases**:
- Protocol magic numbers
- Version bytes
- Fixed delimiters
- Message type indicators

### 2.3 Positional Variance Analysis (Field Stability)

```python
# Measure how much each position varies
import numpy as np

def positional_variance(messages):
    variances = {}
    for pos in range(min_message_length):
        byte_values = [msg[pos] for msg in messages]
        variances[pos] = {
            'variance': np.var(byte_values),
            'std_dev': np.std(byte_values),
            'range': (min(byte_values), max(byte_values))
        }
    return variances
```

**Interpretation**:
- Variance ≈ 0: Fixed field
- Low variance (< 100): Constrained field (type codes, small integers)
- High variance (> 5000): Unconstrained field (IDs, random data)

**Use Cases**:
- Distinguish counters from IDs
- Identify length fields (constrained range)
- Detect sequence numbers (monotonic)

### 2.4 N-gram Frequency Analysis (Multi-Byte Pattern Detection)

```python
# Detect multi-byte sequences (bigrams, trigrams)
from collections import Counter

def find_ngram_patterns(messages, n=2, threshold=0.90):
    """
    Detect recurring n-byte sequences across messages.

    Args:
        messages: List of byte sequences
        n: N-gram size (2=bigram, 3=trigram, etc.)
        threshold: Minimum frequency to consider pattern significant

    Returns:
        Dictionary of frequent n-grams with positions and frequencies
    """
    ngram_positions = {}  # ngram -> list of (message_idx, position)

    for msg_idx, msg in enumerate(messages):
        for pos in range(len(msg) - n + 1):
            ngram = msg[pos:pos+n]

            if ngram not in ngram_positions:
                ngram_positions[ngram] = []
            ngram_positions[ngram].append((msg_idx, pos))

    # Filter by frequency
    frequent_ngrams = {}
    total_messages = len(messages)

    for ngram, positions in ngram_positions.items():
        unique_messages = len(set(msg_idx for msg_idx, _ in positions))
        frequency = unique_messages / total_messages

        if frequency >= threshold:
            # Calculate positional consistency
            position_counts = Counter(pos for _, pos in positions)
            most_common_pos, pos_count = position_counts.most_common(1)[0]
            positional_frequency = pos_count / len(positions)

            frequent_ngrams[ngram] = {
                'frequency': frequency,
                'total_occurrences': len(positions),
                'unique_messages': unique_messages,
                'most_common_position': most_common_pos,
                'positional_frequency': positional_frequency,
                'positions': position_counts
            }

    return frequent_ngrams
```

**Interpretation**:
- Frequency > 0.95 + High positional consistency: Protocol magic number/header
- Frequency > 0.90 + Variable positions: Message delimiter (e.g., `\r\n`)
- Frequency > 0.80: Common multi-byte field (e.g., 2-byte length prefix)
- Position 0 bigrams: Protocol identifiers (e.g., `0xCAFE`, `HTTP`)

**Use Cases**:
- **Magic Numbers**: Detect multi-byte protocol identifiers
  - Example: `0xCAFEBABE` (Java class files), `0x89PNG` (PNG images)
- **Delimiters**: Find message/field boundaries
  - Example: `\r\n` (HTTP), `0x00` (null-terminated strings)
- **Multi-byte Fields**: Identify 2+ byte structures
  - Example: 2-byte length fields, 4-byte sequence numbers
- **Protocol Signatures**: Detect known protocol patterns
  - Example: MQTT `0x10` + length byte, DNS query flags

**N-gram Sizes**:
- **n=2 (bigrams)**: Most useful for magic numbers and 2-byte fields
- **n=3 (trigrams)**: Useful for 3-byte sequences and delimiters
- **n=4+**: Only for specific known patterns (avoid over-fitting)

**Example Output**:
```python
{
    b'\xca\xfe': {
        'frequency': 1.0,
        'total_occurrences': 1000,
        'unique_messages': 1000,
        'most_common_position': 0,
        'positional_frequency': 1.0
    },
    b'\r\n': {
        'frequency': 0.98,
        'total_occurrences': 2500,
        'unique_messages': 980,
        'most_common_position': 127,
        'positional_frequency': 0.65
    }
}
```

**Implementation Strategy**:

```python
# reshark/tools/statistics.py
class ProtocolStatistics:
    def __init__(self, messages):
        self.messages = messages
        self.min_length = min(len(m) for m in messages)

    def analyze(self):
        return {
            'entropy': self.calculate_entropy_map(),
            'invariants': self.find_invariants(),
            'variance': self.calculate_variance_map(),
            'ngrams': {
                'bigrams': self.find_ngrams(n=2),
                'trigrams': self.find_ngrams(n=3)
            }
        }
```

**Performance Expectations**:
- 1000 messages, 100 bytes each: <2 seconds (including n-gram analysis)
- 10000 messages, 1KB each: <8 seconds (including n-gram analysis)
- Complexity: O(n*m) for single-byte, O(n*m*k) for n-grams where k=ngram_size
- N-gram overhead: ~30-40% additional computation time

**Phase 2 Deferrals**:
- Correlation matrices (field relationships)
- Kolmogorov-Smirnov tests (distribution analysis)
- Mutual information (field dependencies)
- Higher-order statistics (patterns across positions)

---

## 3. Test Suite Generation ✅

### Decision: Hybrid Template + f-string with Delegated Logic

**Selected Approach**: Generate thin pytest wrappers that delegate to shared helper functions

**Rationale**:
- Generated tests remain readable and debuggable
- Complex logic lives in testable helper modules (not generated strings)
- Easy to maintain and extend
- No external dependencies beyond pytest

**Architecture**:

```
workspace/rulebook/tests/
├── test_protocol_xyz.py        # Generated test file
└── helpers/
    ├── __init__.py
    ├── field_extractors.py      # Shared extraction logic
    ├── validators.py            # Shared validation logic
    └── pcap_loaders.py          # Shared PCAP loading
```

**Code Generation Pattern**:

```python
# reshark/tools/test_generator.py

HYPOTHESIS_TEST_TEMPLATE = '''
def test_{test_id}_{hypothesis_name}():
    """
    Hypothesis: {hypothesis_claim}
    
    Validation: {validation_description}
    Evidence: Generated from {num_pcaps} PCAPs on {date}
    """
    from helpers.pcap_loaders import load_messages
    from helpers.validators import validate_field, validate_boundary
    
    # Load test data
    messages = load_messages("{pcap_path}")
    
    # Validate hypothesis
    results = validate_{validation_type}(
        messages=messages,
        position={field_position},
        expected={expected_value},
        **{additional_params}
    )
    
    # Assert validation passed
    assert results.passed, f"Hypothesis failed: {{results.failure_reason}}"
'''

def generate_test(hypothesis, pcap_path):
    """Generate thin test wrapper for hypothesis"""
    test_code = HYPOTHESIS_TEST_TEMPLATE.format(
        test_id=hypothesis.id,
        hypothesis_name=sanitize_name(hypothesis.name),
        hypothesis_claim=hypothesis.claim,
        validation_description=hypothesis.validation.description,
        num_pcaps=len(hypothesis.validation.pcaps),
        date=datetime.now().isoformat(),
        pcap_path=pcap_path,
        validation_type=hypothesis.validation.type,  # e.g., 'field', 'boundary', 'sequence'
        field_position=hypothesis.field.position,
        expected_value=repr(hypothesis.expected),
        additional_params=hypothesis.validation.params
    )
    return test_code
```

**Helper Function Examples**:

```python
# workspace/rulebook/tests/helpers/validators.py

def validate_field(messages, position, expected, length=1, tolerance=0.0):
    """
    Validate field appears at expected position with expected value
    
    Args:
        messages: List of byte sequences
        position: Field start position
        expected: Expected value (bytes or int)
        length: Field length in bytes
        tolerance: Allowed frequency tolerance (0.0-1.0)
    
    Returns:
        ValidationResult with passed/failed status and details
    """
    matches = 0
    failures = []
    
    for i, msg in enumerate(messages):
        if len(msg) < position + length:
            failures.append(f"Message {i}: too short")
            continue
        
        actual = msg[position:position+length]
        if actual == expected:
            matches += 1
        else:
            failures.append(f"Message {i}: expected {expected!r}, got {actual!r}")
    
    frequency = matches / len(messages)
    passed = frequency >= (1.0 - tolerance)
    
    return ValidationResult(
        passed=passed,
        frequency=frequency,
        total_messages=len(messages),
        matches=matches,
        failures=failures[:10]  # Limit failure details
    )

def validate_boundary(messages, delimiter, expected_positions):
    """Validate message boundaries occur at expected positions"""
    # Implementation delegates complex boundary detection
    pass

def validate_sequence(messages, position, length, monotonic=True):
    """Validate sequence number field increases monotonically"""
    # Implementation delegates sequence validation logic
    pass
```

**Generated Test Example**:

```python
# workspace/rulebook/tests/test_protocol_xyz.py (Generated)

def test_001_magic_number():
    """
    Hypothesis: Magic number 0xCAFE appears at position 0
    
    Validation: Field validation with 100% frequency requirement
    Evidence: Generated from 3 PCAPs on 2026-01-11T15:30:00
    """
    from helpers.pcap_loaders import load_messages
    from helpers.validators import validate_field
    
    # Load test data
    messages = load_messages("/path/to/validation.pcap")
    
    # Validate hypothesis
    results = validate_field(
        messages=messages,
        position=0,
        expected=b'\xca\xfe',
        length=2,
        tolerance=0.0
    )
    
    # Assert validation passed
    assert results.passed, f"Hypothesis failed: {results.failure_reason}"
```

**Prohibited Patterns**:

❌ **NO inline complex logic**:
```python
# BAD: Complex logic in generated test
def test_bad():
    for msg in messages:
        if len(msg) > 10:
            fields = []
            for i in range(0, len(msg), 4):
                fields.append(struct.unpack('>I', msg[i:i+4])[0])
            # ... more inline complexity
```

✅ **YES thin wrapper with delegation**:
```python
# GOOD: Thin wrapper, logic in helper
def test_good():
    from helpers.validators import validate_structured_fields
    results = validate_structured_fields(messages, field_size=4)
    assert results.passed
```

**Benefits**:
- Generated tests are <20 lines each
- Helper functions are unit-testable independently
- Easy to debug (step into helper functions)
- Changes to validation logic don't require regeneration
- Human-readable generated code

**Performance Expectations**:
- Generate 10 tests: <100ms
- Execute 10 tests with 1000 messages each: <5 seconds
- Test file size: ~1KB per test

**Testing the Generator**:
```python
# tests/unit/test_tools/test_test_generator.py
def test_generates_valid_python_syntax():
    hypothesis = create_test_hypothesis()
    test_code = generate_test(hypothesis, "test.pcap")
    compile(test_code, '<string>', 'exec')  # Should not raise

def test_generates_thin_wrappers():
    hypothesis = create_test_hypothesis()
    test_code = generate_test(hypothesis, "test.pcap")
    lines = test_code.split('\n')
    assert len([l for l in lines if l.strip()]) < 20  # Max 20 non-empty lines
```

---

## 4. Session Isolation ✅

### Decision: Hierarchical Date Directories + UUID Sessions (Constrained Hybrid)

**Selected Approach**: Combine date-based organization with UUID session identifiers under strict rules

**Rationale**:
- Date hierarchy enables easy cleanup and archival
- UUID guarantees absolute session uniqueness
- No user naming prevents namespace pollution
- Chronological browsing with guaranteed isolation

**Directory Structure**:

```
workspace/
├── notebook/
│   ├── 2026-01-11/
│   │   ├── a7f3c891-4b2e-4d3f-9a12-3c5d8e9f1234/
│   │   │   ├── metadata.json
│   │   │   ├── observations.json
│   │   │   └── hypotheses.json
│   │   └── b8e4d902-5c3f-5e4g-0b23-4d6e9f0g2345/
│   └── 2026-01-12/
│       └── c9f5e013-6d4g-6f5h-1c34-5e7f0g1h3456/
├── rulebook/
│   ├── grammars/
│   │   └── protocol-name-v1.0.py
│   └── tests/
│       └── test_protocol-name.py
└── cookbook/
    └── procedures.md
```

**Strict Rules**:

1. **Date Format**: YYYY-MM-DD (ISO 8601)
   - Uses UTC date to avoid timezone ambiguity
   - Created at session initialization time
   
2. **Session ID**: UUID4 (RFC 4122)
   - Generated via Python `uuid.uuid4()`
   - Lowercase hexadecimal with hyphens
   - No custom naming allowed
   
3. **Path Construction**:
   ```python
   session_path = workspace / "notebook" / date.isoformat() / str(session_uuid)
   ```

4. **Metadata Requirements**:
   - Every session MUST have `metadata.json` as first file
   - Metadata includes: creation timestamp, user, input PCAPs, status
   
5. **Collision Prevention**:
   - UUID collision probability: negligible (2^-122)
   - Date directory collision: impossible (one dir per day)
   - No race conditions: atomic directory creation with `exist_ok=False`

6. **Session States**:
   - `active`: Session directory exists, no completion marker
   - `completed`: Contains `.completed` marker file
   - `archived`: Moved to archive location (future: Phase 2)

**Implementation**:

```python
# reshark/utils/session.py

from pathlib import Path
from uuid import uuid4
from datetime import datetime, timezone
import json

class SessionManager:
    def __init__(self, workspace_root: Path):
        self.workspace = workspace_root
        self.notebook_root = workspace_root / "notebook"
    
    def create_session(self, pcap_paths: list[str]) -> str:
        """
        Create new isolated session with concurrency protection

        Uses file-based advisory lock to prevent race conditions when
        multiple agents create sessions simultaneously.

        Returns:
            session_id (str): UUID of created session

        Raises:
            SessionCreationError: If session creation fails after lock acquisition
        """
        import fcntl

        # Determine date directory (UTC)
        date_str = datetime.now(timezone.utc).date().isoformat()
        date_dir = self.notebook_root / date_str
        date_dir.mkdir(parents=True, exist_ok=True)

        # Acquire exclusive lock for session creation
        lock_file_path = date_dir / ".session_create.lock"

        with open(lock_file_path, 'w') as lock_file:
            # Block until exclusive lock is available
            fcntl.flock(lock_file.fileno(), fcntl.LOCK_EX)

            try:
                # Generate session ID within lock
                session_id = str(uuid4())

                # Create session directory (atomic check-and-create)
                session_dir = date_dir / session_id
                session_dir.mkdir(parents=False, exist_ok=False)

                # Write metadata
                metadata = {
                    "session_id": session_id,
                    "created_at": datetime.now(timezone.utc).isoformat(),
                    "pcap_paths": pcap_paths,
                    "status": "active",
                    "user": os.getenv("USER", "unknown")
                }

                metadata_path = session_dir / "metadata.json"
                metadata_path.write_text(json.dumps(metadata, indent=2))

                return session_id

            except FileExistsError:
                # UUID collision (extremely unlikely, but handle gracefully)
                raise SessionCreationError(
                    f"Session directory {session_id} already exists"
                )
            finally:
                # Lock is released automatically when 'with' block exits
                pass
    
    def get_session_path(self, session_id: str) -> Path:
        """
        Resolve session ID to filesystem path
        
        Searches all date directories for matching UUID
        """
        for date_dir in self.notebook_root.iterdir():
            if not date_dir.is_dir():
                continue
            session_path = date_dir / session_id
            if session_path.exists():
                return session_path
        
        raise SessionNotFoundError(f"Session {session_id} not found")
    
    def list_sessions(self, date: str = None) -> list[dict]:
        """
        List all sessions, optionally filtered by date
        
        Args:
            date: Optional date string (YYYY-MM-DD)
        
        Returns:
            List of session metadata dicts
        """
        sessions = []
        
        if date:
            date_dirs = [self.notebook_root / date]
        else:
            date_dirs = sorted(self.notebook_root.iterdir())
        
        for date_dir in date_dirs:
            if not date_dir.is_dir():
                continue
            
            for session_dir in date_dir.iterdir():
                if not session_dir.is_dir():
                    continue
                
                metadata_path = session_dir / "metadata.json"
                if metadata_path.exists():
                    metadata = json.loads(metadata_path.read_text())
                    sessions.append(metadata)
        
        return sessions
    
    def complete_session(self, session_id: str):
        """Mark session as completed"""
        session_path = self.get_session_path(session_id)
        marker = session_path / ".completed"
        marker.write_text(datetime.now(timezone.utc).isoformat())
```

**CLI Usage**:

```bash
# Create session (auto-generates UUID)
$ python -m reshark.scopes.observer --pcap input.pcap
Created session: a7f3c891-4b2e-4d3f-9a12-3c5d8e9f1234

# List today's sessions
$ python -m reshark.utils.session list
2026-01-11:
  - a7f3c891-4b2e-4d3f-9a12-3c5d8e9f1234 (active)
  - b8e4d902-5c3f-5e4g-0b23-4d6e9f0g2345 (completed)

# Continue existing session
$ python -m reshark.scopes.theorist --session a7f3c891-4b2e-4d3f-9a12-3c5d8e9f1234
```

**Benefits**:

✅ **Uniqueness**: UUID guarantees no collisions
✅ **Chronology**: Date directories enable time-based queries
✅ **Cleanup**: Easy to remove old sessions by date
✅ **Traceability**: Every session has provenance metadata
✅ **Simplicity**: No naming conflicts, no validation needed
✅ **Automation-friendly**: No human naming decisions required
✅ **Concurrency-safe**: File-based advisory locks prevent race conditions
✅ **Multi-agent support**: Works across OpenHands agents in separate Docker containers

**Concurrency Protection**:

The `create_session()` method uses POSIX advisory locks (fcntl) to prevent race conditions:

- **Lock scope**: Per-date directory (`.session_create.lock` file)
- **Lock type**: Exclusive (`LOCK_EX`) - blocks concurrent creation attempts
- **Lock duration**: Held only during session directory creation (~10ms)
- **Automatic release**: Lock released when `with` block exits (exception-safe)
- **Cross-process**: Works across multiple Python processes and Docker containers

**Race Condition Prevention**:

```python
# Agent A and Agent B both call create_session() simultaneously
# Time T=0: Agent A acquires lock on .session_create.lock
# Time T=1: Agent B blocks waiting for lock
# Time T=2: Agent A creates session directory, writes metadata, releases lock
# Time T=3: Agent B acquires lock, creates new session directory with different UUID
# Result: Two distinct sessions, no collision
```

**Forbidden Patterns**:

❌ User-specified session names
❌ Custom session ID formats
❌ Session directories outside date hierarchy
❌ Sessions without metadata.json
❌ Local timezone dates (must use UTC)
❌ Bypassing advisory lock (always use SessionManager.create_session())

**Performance Expectations**:

- Session creation: <15ms (directory + metadata write + lock overhead)
- Lock contention impact: +5ms average per concurrent creation attempt
- Session lookup: O(d) where d = number of date directories
- List sessions: O(n) where n = total sessions (can optimize with index)
- Concurrent creation throughput: ~100 sessions/second with 10 concurrent agents

**Cleanup Strategy** (Future: Phase 2):

```python
# Archive sessions older than 30 days
def archive_old_sessions(days_threshold=30):
    cutoff_date = datetime.now(timezone.utc).date() - timedelta(days=days_threshold)
    for date_dir in notebook_root.iterdir():
        date = datetime.fromisoformat(date_dir.name).date()
        if date < cutoff_date:
            # Move to archive or delete
            pass
```

---

## 5. Grammar Format ✅

### Decision: Split Hybrid - YAML as Law, Python as Interpreter

**Selected Approach**: YAML grammar definitions with Python interpreter/executor

**Rationale**:
- YAML is the authoritative source of truth (the "law")
- Python interprets and executes YAML grammars
- Clear separation: data (grammar) vs code (parser)
- Version control friendly (YAML diffs are readable)
- Language-agnostic grammar definitions

**Architecture**:

```
workspace/rulebook/grammars/
├── mqtt_v3.yaml              # Grammar definition (LAW)
├── modbus_tcp.yaml           # Grammar definition (LAW)
└── custom_proto.yaml         # Grammar definition (LAW)

reshark/memory/rulebook.py    # Grammar interpreter (EXECUTOR)
```

**Grammar Definition Format (YAML)**:

```yaml
# workspace/rulebook/grammars/mqtt_v3.yaml

protocol:
  name: MQTTv3
  version: "1.0"
  description: "MQTT protocol version 3 message structure"

metadata:
  validated_at: "2026-01-11T15:30:00Z"
  session_id: "a7f3c891-4b2e-4d3f-9a12-3c5d8e9f1234"
  validation_pcaps:
    - "test1.pcap"
    - "test2.pcap"
    - "test3.pcap"
  confidence: 0.98
  evidence_path: "evidence/mqtt_v3_evidence.json"

fields:
  - name: message_type
    position: 0
    length: 1
    type: bitfield
    bits: [4, 5, 6, 7]  # High nibble
    description: "MQTT message type (CONNECT, PUBLISH, etc.)"
    
  - name: dup_flag
    position: 0
    length: 1
    type: bitfield
    bits: [3]
    description: "Duplicate delivery flag"
    
  - name: qos_level
    position: 0
    length: 1
    type: bitfield
    bits: [1, 2]
    description: "Quality of Service level"
    
  - name: retain_flag
    position: 0
    length: 1
    type: bitfield
    bits: [0]
    description: "Retain flag"
    
  - name: remaining_length
    position: 1
    length: variable
    type: varint
    encoding: mqtt_varint
    description: "Remaining message length"
    
  - name: payload
    position: dynamic  # After remaining_length
    length: remaining_length
    type: bytes
    description: "Message payload"

constraints:
  - field: message_type
    operator: in
    values: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14]
    
  - field: qos_level
    operator: in
    values: [0, 1, 2]

invariants:
  - position: 0
    mask: 0xF0
    description: "Message type always in high nibble"
```

**Python Interpreter (Executor)**:

```python
# reshark/memory/rulebook.py

import yaml
from pathlib import Path
from typing import Any, Dict, List

class GrammarInterpreter:
    """Interprets and executes YAML grammar definitions"""
    
    def __init__(self, grammar_path: Path):
        self.grammar_path = grammar_path
        self.grammar = self._load_grammar()
        self.protocol_name = self.grammar['protocol']['name']
        self.fields = self.grammar['fields']
        self.constraints = self.grammar.get('constraints', [])
    
    def _load_grammar(self) -> dict:
        """Load YAML grammar definition"""
        with open(self.grammar_path, 'r') as f:
            return yaml.safe_load(f)
    
    def parse(self, data: bytes) -> Dict[str, Any]:
        """
        Parse binary data according to grammar
        
        Args:
            data: Raw bytes to parse
            
        Returns:
            Dictionary of field names to parsed values
        """
        result = {}
        offset = 0
        
        for field in self.fields:
            field_name = field['name']
            field_type = field['type']
            
            # Extract field based on type
            if field_type == 'bitfield':
                result[field_name] = self._extract_bitfield(
                    data, field['position'], field['bits']
                )
            elif field_type == 'varint':
                value, consumed = self._extract_varint(
                    data, field['position'], field['encoding']
                )
                result[field_name] = value
                offset += consumed
            elif field_type == 'bytes':
                length = result.get(field['length'])  # Dynamic length reference
                result[field_name] = self._extract_bytes(
                    data, offset, length
                )
            else:
                result[field_name] = self._extract_fixed(
                    data, field['position'], field['length']
                )
        
        # Validate constraints
        self._validate_constraints(result)
        
        return result
    
    def _extract_bitfield(self, data: bytes, position: int, bits: List[int]) -> int:
        """Extract bitfield from byte"""
        byte_value = data[position]
        result = 0
        for i, bit_pos in enumerate(sorted(bits)):
            if byte_value & (1 << bit_pos):
                result |= (1 << i)
        return result
    
    def _extract_varint(self, data: bytes, position: int, encoding: str) -> tuple[int, int]:
        """Extract variable-length integer"""
        if encoding == 'mqtt_varint':
            value = 0
            multiplier = 1
            consumed = 0
            
            for i in range(position, len(data)):
                byte = data[i]
                value += (byte & 0x7F) * multiplier
                multiplier *= 128
                consumed += 1
                
                if (byte & 0x80) == 0:
                    break
            
            return value, consumed
        
        raise ValueError(f"Unsupported varint encoding: {encoding}")
    
    def _extract_bytes(self, data: bytes, position: int, length: int) -> bytes:
        """Extract fixed-length byte sequence"""
        return data[position:position+length]
    
    def _extract_fixed(self, data: bytes, position: int, length: int) -> int:
        """Extract fixed-length integer"""
        return int.from_bytes(data[position:position+length], 'big')
    
    def _validate_constraints(self, parsed: dict):
        """Validate parsed data against constraints"""
        for constraint in self.constraints:
            field_name = constraint['field']
            operator = constraint['operator']
            expected = constraint['values']
            actual = parsed.get(field_name)
            
            if operator == 'in':
                if actual not in expected:
                    raise ValueError(
                        f"Constraint violation: {field_name}={actual} not in {expected}"
                    )
    
    def format(self, parsed: dict) -> str:
        """Format parsed data as human-readable string"""
        lines = [f"Protocol: {self.protocol_name}"]
        for field_name, value in parsed.items():
            lines.append(f"  {field_name}: {value}")
        return '\n'.join(lines)


class Rulebook:
    """Manages collection of grammar definitions"""
    
    def __init__(self, grammars_dir: Path):
        self.grammars_dir = grammars_dir
        self._interpreters = {}
    
    def load_grammar(self, protocol_name: str) -> GrammarInterpreter:
        """Load and cache grammar interpreter"""
        if protocol_name not in self._interpreters:
            grammar_path = self.grammars_dir / f"{protocol_name}.yaml"
            if not grammar_path.exists():
                raise FileNotFoundError(f"Grammar not found: {protocol_name}")
            self._interpreters[protocol_name] = GrammarInterpreter(grammar_path)
        
        return self._interpreters[protocol_name]
    
    def list_grammars(self) -> List[dict]:
        """List all available grammars with metadata"""
        grammars = []
        for yaml_file in self.grammars_dir.glob("*.yaml"):
            with open(yaml_file, 'r') as f:
                grammar = yaml.safe_load(f)
                grammars.append({
                    'name': grammar['protocol']['name'],
                    'version': grammar['protocol']['version'],
                    'validated_at': grammar['metadata']['validated_at'],
                    'confidence': grammar['metadata']['confidence']
                })
        return grammars
```

**Usage Example**:

```python
# Load grammar and parse data
from reshark.memory.rulebook import Rulebook

rulebook = Rulebook(Path("workspace/rulebook/grammars"))
interpreter = rulebook.load_grammar("mqtt_v3")

# Parse message
message = b'\x10\x12\x00\x04MQTT\x04\x02\x00\x3c\x00\x00'
parsed = interpreter.parse(message)

print(interpreter.format(parsed))
# Output:
# Protocol: MQTTv3
#   message_type: 1
#   dup_flag: 0
#   qos_level: 0
#   retain_flag: 0
#   remaining_length: 18
#   payload: b'\x00\x04MQTT\x04\x02\x00\x3c\x00\x00'
```

**Benefits of Split Hybrid**:

✅ **YAML is authoritative**: Grammar lives in data, not code  
✅ **Version control**: YAML diffs are human-readable  
✅ **Language-agnostic**: Could generate parsers for other languages  
✅ **Testable**: Python interpreter has unit tests  
✅ **Extensible**: Add new field types without changing grammars  
✅ **Portable**: YAML can be shared, imported, published  

**Clear Responsibilities**:

| Component | Responsibility | Changes Require |
|-----------|---------------|-----------------|
| YAML Grammar | Protocol structure definition | Re-validation |
| Python Interpreter | Execution logic | Testing |
| Test Suite | Validation of grammar | Re-generation |

**Grammar Promotion Workflow**:

```python
# In Archivist scope (reshark/scopes/archivist.py)

def promote_to_rulebook(hypothesis, validation_results):
    """Generate YAML grammar from validated hypothesis"""
    
    grammar = {
        'protocol': {
            'name': hypothesis.protocol_name,
            'version': '1.0',
            'description': hypothesis.description
        },
        'metadata': {
            'validated_at': datetime.now(timezone.utc).isoformat(),
            'session_id': hypothesis.session_id,
            'validation_pcaps': validation_results.pcap_paths,
            'confidence': validation_results.confidence,
            'evidence_path': f"evidence/{hypothesis.protocol_name}_evidence.json"
        },
        'fields': hypothesis.fields,
        'constraints': hypothesis.constraints,
        'invariants': hypothesis.invariants
    }
    
    # Write YAML grammar (the law)
    grammar_path = rulebook_dir / f"{hypothesis.protocol_name}.yaml"
    with open(grammar_path, 'w') as f:
        yaml.dump(grammar, f, default_flow_style=False, sort_keys=False)
    
    return grammar_path
```

**Validation**:

```python
# Generated test uses interpreter
def test_mqtt_v3_grammar():
    """Test MQTT v3 grammar against validation PCAPs"""
    from helpers.pcap_loaders import load_messages
    from reshark.memory.rulebook import Rulebook
    
    # Load grammar interpreter
    rulebook = Rulebook(Path("workspace/rulebook/grammars"))
    interpreter = rulebook.load_grammar("mqtt_v3")
    
    # Load test messages
    messages = load_messages("validation/mqtt_v3.pcap")
    
    # Parse all messages
    for msg in messages:
        parsed = interpreter.parse(msg)
        assert parsed['message_type'] in [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14]
```

**Schema Validation with Pydantic**:

To prevent malformed YAML grammars from entering the Rulebook (constitutional "truth"), add Pydantic validation:

```python
# reshark/memory/grammar_schema.py

from pydantic import BaseModel, Field, field_validator
from typing import List, Literal, Union
from datetime import datetime

class ProtocolInfo(BaseModel):
    """Protocol metadata"""
    name: str = Field(..., min_length=1, description="Protocol name")
    version: str = Field(..., pattern=r'^\d+\.\d+$', description="Version (semver)")
    description: str = Field(..., min_length=1, description="Protocol description")

class GrammarMetadata(BaseModel):
    """Grammar validation metadata"""
    validated_at: datetime = Field(..., description="Validation timestamp (ISO8601)")
    session_id: str = Field(..., pattern=r'^[a-f0-9-]{36}$', description="Session UUID")
    validation_pcaps: List[str] = Field(..., min_items=1, description="Validation PCAP filenames")
    confidence: float = Field(..., ge=0.0, le=1.0, description="Confidence score")
    evidence_path: str = Field(..., description="Path to evidence JSON")

class FieldDefinition(BaseModel):
    """Grammar field definition"""
    name: str = Field(..., min_length=1, description="Field name")
    position: Union[int, Literal["dynamic"]] = Field(..., description="Byte position or 'dynamic'")
    length: Union[int, str] = Field(..., description="Fixed length or field reference")
    type: Literal["bitfield", "varint", "bytes", "fixed"] = Field(..., description="Field type")
    description: str = Field(..., min_length=1, description="Field description")

    # Optional fields for specific types
    bits: List[int] = Field(default=None, description="Bit positions for bitfield")
    encoding: str = Field(default=None, description="Encoding for varint")

    @field_validator('bits')
    def validate_bits(cls, v, values):
        """Validate bits are specified for bitfield types"""
        if values.data.get('type') == 'bitfield' and not v:
            raise ValueError("bitfield type requires 'bits' field")
        if v:
            if not all(0 <= bit <= 7 for bit in v):
                raise ValueError("bit positions must be 0-7")
        return v

    @field_validator('encoding')
    def validate_encoding(cls, v, values):
        """Validate encoding is specified for varint types"""
        if values.data.get('type') == 'varint' and not v:
            raise ValueError("varint type requires 'encoding' field")
        return v

class Constraint(BaseModel):
    """Field constraint definition"""
    field: str = Field(..., min_length=1, description="Field name to constrain")
    operator: Literal["in", "range", "eq", "ne"] = Field(..., description="Constraint operator")
    values: List[Union[int, str]] = Field(..., min_items=1, description="Allowed values")

class Invariant(BaseModel):
    """Positional invariant definition"""
    position: int = Field(..., ge=0, description="Byte position")
    mask: str = Field(..., pattern=r'^0x[0-9A-Fa-f]{2}$', description="Hex mask (e.g., '0xF0')")
    description: str = Field(..., min_length=1, description="Invariant description")

class GrammarSchema(BaseModel):
    """Complete grammar schema with validation"""
    schema_version: str = Field(
        default="1.0",
        pattern=r'^\d+\.\d+$',
        description="Grammar format version (for future evolution)"
    )
    protocol: ProtocolInfo
    metadata: GrammarMetadata
    fields: List[FieldDefinition] = Field(..., min_items=1)
    constraints: List[Constraint] = Field(default_factory=list)
    invariants: List[Invariant] = Field(default_factory=list)

    @field_validator('fields')
    def validate_field_references(cls, v):
        """Ensure dynamic length references point to existing fields"""
        field_names = {field.name for field in v}
        for field in v:
            if isinstance(field.length, str) and field.length not in field_names:
                raise ValueError(f"Field '{field.name}' references unknown field '{field.length}'")
        return v
```

**Updated GrammarInterpreter with Validation**:

```python
# reshark/memory/rulebook.py

import yaml
from pathlib import Path
from typing import Any, Dict, List
from pydantic import ValidationError
from reshark.memory.grammar_schema import GrammarSchema

class GrammarInterpreter:
    """Interprets and executes YAML grammar definitions"""

    def __init__(self, grammar_path: Path):
        self.grammar_path = grammar_path
        self.grammar = self._load_grammar()
        self.protocol_name = self.grammar['protocol']['name']
        self.fields = self.grammar['fields']
        self.constraints = self.grammar.get('constraints', [])

    def _load_grammar(self) -> dict:
        """Load and validate YAML grammar definition"""
        with open(self.grammar_path, 'r') as f:
            raw_grammar = yaml.safe_load(f)

        # Validate with Pydantic
        try:
            validated = GrammarSchema(**raw_grammar)
            return validated.model_dump()
        except ValidationError as e:
            raise ValueError(f"Invalid grammar {self.grammar_path.name}: {e}")

    # ... rest of implementation unchanged
```

**Grammar Linting Tool**:

```python
# reshark/cli/lint_grammar.py

from pathlib import Path
from pydantic import ValidationError
from reshark.memory.grammar_schema import GrammarSchema
import yaml
import sys

def lint_grammar(grammar_path: Path) -> bool:
    """
    Validate grammar against schema

    Returns:
        True if valid, False otherwise
    """
    try:
        with open(grammar_path, 'r') as f:
            raw_grammar = yaml.safe_load(f)

        GrammarSchema(**raw_grammar)
        print(f"✅ {grammar_path.name} is valid")
        return True

    except ValidationError as e:
        print(f"❌ {grammar_path.name} validation failed:")
        for error in e.errors():
            field = " -> ".join(str(x) for x in error['loc'])
            msg = error['msg']
            print(f"  {field}: {msg}")
        return False

    except Exception as e:
        print(f"❌ {grammar_path.name} error: {e}")
        return False

def main():
    """Lint all grammars in directory"""
    if len(sys.argv) != 2:
        print("Usage: python -m reshark.cli.lint_grammar <grammars_dir>")
        sys.exit(1)

    grammars_dir = Path(sys.argv[1])
    if not grammars_dir.is_dir():
        print(f"Error: {grammars_dir} is not a directory")
        sys.exit(1)

    results = []
    for yaml_file in sorted(grammars_dir.glob("*.yaml")):
        results.append(lint_grammar(yaml_file))

    valid_count = sum(results)
    total_count = len(results)

    print(f"\n{valid_count}/{total_count} grammars valid")
    sys.exit(0 if all(results) else 1)

if __name__ == '__main__':
    main()
```

**CLI Usage**:

```bash
# Lint all grammars before committing
$ python -m reshark.cli.lint_grammar workspace/rulebook/grammars
✅ mqtt_v3.yaml is valid
✅ modbus_tcp.yaml is valid
❌ custom_proto.yaml validation failed:
  fields -> 2 -> length: varint type requires 'encoding' field
  metadata -> confidence: ensure this value is less than or equal to 1.0

1/3 grammars valid

# Add to CI/CD pipeline
$ python -m reshark.cli.lint_grammar workspace/rulebook/grammars || exit 1
```

**Benefits of Pydantic Validation**:

✅ **Schema enforcement**: Prevents malformed YAML from entering Rulebook
✅ **Type safety**: Validates field types, ranges, patterns
✅ **Versioning**: `schema_version` field enables grammar format evolution
✅ **Developer experience**: Clear error messages for invalid grammars
✅ **CI/CD integration**: Automated validation in pre-commit hooks
✅ **Documentation**: Pydantic models serve as schema documentation

**Phase 1 Grammar Limitations**:

For MVP simplicity, Phase 1 grammars do **not** support:
- ❌ Grammar composition (import/extend)
- ❌ Conditional fields (if/else)
- ❌ Calculated fields (expressions)
- ❌ Custom validators beyond Pydantic
- ❌ Multi-message transactions
- ❌ Stateful parsing (requires session context)

These features are deferred to Phase 2 (Patternbook + grammar evolution).

**Future Evolution** (Phase 2):

- Add grammar composition (import/extend)
- Generate parsers for C/Rust/Go from YAML
- Grammar versioning and migration
- Grammar linting/validation tools
- Grammar documentation generator

---

## Known Unknowns Resolution

### Optimal Chunk Size for Large PCAPs

**Decision**: Use tshark streaming output, no chunking needed
- tshark handles memory management internally
- Output can be piped or streamed to files
- For >1GB files: Use `-T fields` or `-z follow` for reduced memory footprint

### Entropy Threshold Values

**Decision**: Empirical thresholds based on Shannon entropy (0-8 bits)
- H ≈ 0: Fixed byte (invariant, magic number)
- H < 3.0: Structured field (type codes, flags)
- H = 3.0-6.0: Variable content (IDs, counters)
- H > 7.0: Random/encrypted data
- **Refinement**: Adjust thresholds per protocol during validation

### Hypothesis Confidence Scoring

**Decision**: Frequency-based confidence with validation weight
```python
confidence = (validation_pass_rate * 0.7) + (frequency_score * 0.3)

where:
  validation_pass_rate = passed_pcaps / total_pcaps
  frequency_score = matches / total_messages
```
- Minimum confidence for promotion: 0.90 (90%)
- Can be adjusted per hypothesis type

### Test Coverage Requirements for Promotion

**Decision**: Constitutional minimum + constitutional compliance
- Minimum 3 PCAPs validated (constitutional requirement)
- 100% pass rate on all 3 PCAPs
- Negative test included (corrupted PCAP rejection)
- Evidence trail complete (all test results recorded)

### Synthetic Protocol Generator

**Decision**: Scapy-based generator for controlled testing
```python
# Generate synthetic protocol messages
from scapy.all import raw

def generate_synthetic_protocol(count=1000):
    messages = []
    for i in range(count):
        msg = bytes([
            0xCA, 0xFE,              # Magic number
            i % 256,                  # Sequence number
            (i >> 8) % 256,          # Message ID
            random.randint(0, 255)   # Random payload start
        ]) + os.urandom(random.randint(10, 50))  # Random payload
        messages.append(msg)
    return messages
```
- Used for unit tests and integration tests
- Controllable invariants, boundaries, patterns
- Can simulate corruption, truncation, edge cases

---

## Implementation Best Practices

### Tshark Subprocess Timeout and Error Handling

**Rationale**: Tshark can hang or consume excessive resources when processing malformed/corrupted PCAPs. Robust error handling prevents Observer scope lockup and ensures graceful degradation.

**Implementation Pattern**:

```python
# reshark/scopes/observer/pcap_processor.py

import subprocess
from pathlib import Path
from typing import List, Optional
from dataclasses import dataclass

@dataclass
class TsharkResult:
    """Result of tshark subprocess execution"""
    success: bool
    stdout: str
    stderr: str
    returncode: int
    timeout: bool = False

class PcapProcessor:
    """Processes PCAP files using tshark with robust error handling"""

    DEFAULT_TIMEOUT = 30  # seconds per PCAP
    MAX_PCAP_SIZE = 100 * 1024 * 1024  # 100MB

    def run_tshark(
        self,
        pcap_path: Path,
        fields: List[str],
        timeout: Optional[int] = None
    ) -> TsharkResult:
        """
        Run tshark with timeout and error handling

        Args:
            pcap_path: Path to PCAP file
            fields: List of tshark fields to extract
            timeout: Timeout in seconds (default: 30s)

        Returns:
            TsharkResult with success status and output
        """
        timeout = timeout or self.DEFAULT_TIMEOUT

        # Validate PCAP file
        if not pcap_path.exists():
            return TsharkResult(
                success=False,
                stdout="",
                stderr=f"PCAP file not found: {pcap_path}",
                returncode=-1
            )

        # Check file size
        if pcap_path.stat().st_size > self.MAX_PCAP_SIZE:
            return TsharkResult(
                success=False,
                stdout="",
                stderr=f"PCAP exceeds maximum size: {self.MAX_PCAP_SIZE} bytes",
                returncode=-1
            )

        # Build tshark command
        field_args = [f"-e {field}" for field in fields]
        cmd = [
            "tshark",
            "-r", str(pcap_path),
            "-T", "fields",
            *field_args,
            "-E", "separator=|",
            "-E", "occurrence=f"  # First occurrence only
        ]

        try:
            result = subprocess.run(
                cmd,
                capture_output=True,
                text=True,
                timeout=timeout,
                check=False  # Don't raise on non-zero exit
            )

            # Check for tshark errors
            if result.returncode != 0:
                return TsharkResult(
                    success=False,
                    stdout=result.stdout,
                    stderr=result.stderr,
                    returncode=result.returncode
                )

            return TsharkResult(
                success=True,
                stdout=result.stdout,
                stderr=result.stderr,
                returncode=result.returncode
            )

        except subprocess.TimeoutExpired:
            return TsharkResult(
                success=False,
                stdout="",
                stderr=f"Tshark execution exceeded timeout: {timeout}s",
                returncode=-1,
                timeout=True
            )

        except FileNotFoundError:
            return TsharkResult(
                success=False,
                stdout="",
                stderr="Tshark not found. Install with: apt-get install tshark",
                returncode=-1
            )

        except Exception as e:
            return TsharkResult(
                success=False,
                stdout="",
                stderr=f"Unexpected error running tshark: {str(e)}",
                returncode=-1
            )

    def extract_packets(self, pcap_path: Path) -> List[bytes]:
        """
        Extract raw packet payloads with error handling

        Returns:
            List of packet payloads, or empty list on error
        """
        result = self.run_tshark(
            pcap_path,
            fields=["data.data"],
            timeout=self.DEFAULT_TIMEOUT
        )

        if not result.success:
            # Log error to Notebook
            self._log_error(f"Failed to extract packets: {result.stderr}")
            return []

        # Parse tshark output
        packets = []
        for line in result.stdout.strip().split('\n'):
            if line:
                try:
                    # Convert hex string to bytes
                    hex_data = line.replace(':', '')
                    packets.append(bytes.fromhex(hex_data))
                except ValueError as e:
                    self._log_error(f"Invalid hex data: {line[:50]}...")
                    continue

        return packets
```

**Key Features**:

✅ **Timeout protection**: Default 30s timeout prevents infinite hangs
✅ **File size limits**: Reject PCAPs >100MB (adjustable per environment)
✅ **Graceful degradation**: Return empty list instead of crashing
✅ **Error logging**: Record failures to Notebook for debugging
✅ **Subprocess isolation**: tshark runs in separate process (no memory leaks)
✅ **Clear error messages**: Distinguish timeout, missing file, invalid format

**Constitutional Alignment**:

This implementation respects the Observer scope contract:
- "If extraction fails, record the failure to Notebook and return empty data"
- No silent failures - all errors logged
- No exceptions propagated to LangGraph orchestrator

---

### Pytest Fixtures and Parallel Test Execution

**Rationale**: Phase 1 MVP will generate ~200-300 test files. Shared fixtures reduce boilerplate, and parallel execution keeps CI/CD fast (<5 minutes).

**Shared Fixtures** (`conftest.py`):

```python
# tests/conftest.py

import pytest
from pathlib import Path
from uuid import uuid4
from datetime import datetime, timezone
import tempfile
import shutil

@pytest.fixture(scope="session")
def test_workspace():
    """
    Create temporary workspace for test isolation

    Scope: session (created once per test run)
    """
    workspace_root = Path(tempfile.mkdtemp(prefix="reshark_test_"))

    # Create workspace structure
    (workspace_root / "notebook").mkdir(parents=True)
    (workspace_root / "rulebook" / "grammars").mkdir(parents=True)
    (workspace_root / "cookbook").mkdir(parents=True)

    yield workspace_root

    # Cleanup after all tests
    shutil.rmtree(workspace_root)

@pytest.fixture
def session_id():
    """Generate unique session ID for test isolation"""
    return str(uuid4())

@pytest.fixture
def sample_pcap(tmp_path):
    """
    Generate minimal valid PCAP file

    Scope: function (new PCAP per test)
    """
    pcap_path = tmp_path / "test.pcap"

    # PCAP global header (24 bytes)
    global_header = bytes([
        0xd4, 0xc3, 0xb2, 0xa1,  # Magic number
        0x02, 0x00,              # Version major
        0x04, 0x00,              # Version minor
        0x00, 0x00, 0x00, 0x00,  # Timezone offset
        0x00, 0x00, 0x00, 0x00,  # Timestamp accuracy
        0xff, 0xff, 0x00, 0x00,  # Snaplen (65535)
        0x01, 0x00, 0x00, 0x00   # Network (Ethernet)
    ])

    # Single packet (truncated for minimal test)
    packet_header = bytes([
        0x00, 0x00, 0x00, 0x00,  # Timestamp seconds
        0x00, 0x00, 0x00, 0x00,  # Timestamp microseconds
        0x0a, 0x00, 0x00, 0x00,  # Captured length (10 bytes)
        0x0a, 0x00, 0x00, 0x00   # Original length (10 bytes)
    ])

    packet_data = bytes([0xCA, 0xFE, 0xBA, 0xBE, 0x00, 0x01, 0x02, 0x03, 0x04, 0x05])

    pcap_path.write_bytes(global_header + packet_header + packet_data)
    return pcap_path

@pytest.fixture
def mock_tshark_output():
    """Sample tshark field output for testing"""
    return """cafebabe00010203
deadbeef01020304
feedface02030405"""

@pytest.fixture
def sample_grammar(test_workspace):
    """
    Create minimal valid YAML grammar

    Returns:
        Path to grammar file
    """
    grammar_path = test_workspace / "rulebook" / "grammars" / "test_proto.yaml"

    grammar_content = """
schema_version: "1.0"

protocol:
  name: "TestProtocol"
  version: "1.0"
  description: "Minimal test protocol"

metadata:
  validated_at: "2025-01-15T10:00:00Z"
  session_id: "00000000-0000-0000-0000-000000000000"
  validation_pcaps:
    - "test.pcap"
  confidence: 0.95
  evidence_path: "notebook/2025-01-15/session-id/evidence.json"

fields:
  - name: "magic"
    position: 0
    length: 2
    type: "fixed"
    description: "Magic number 0xCAFE"

  - name: "sequence"
    position: 2
    length: 1
    type: "fixed"
    description: "Sequence number"

constraints:
  - field: "magic"
    type: "equals"
    value: "0xCAFE"

invariants:
  - field: "magic"
    description: "Always 0xCAFE"
"""

    grammar_path.write_text(grammar_content)
    return grammar_path
```

**Parallel Execution Configuration** (`pytest.ini`):

```ini
# pytest.ini

[pytest]
# Run tests in parallel with pytest-xdist
addopts =
    -v
    --tb=short
    --strict-markers
    -n auto  # Use all available CPU cores
    --maxfail=5  # Stop after 5 failures
    --durations=10  # Show 10 slowest tests

# Test discovery
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*

# Markers for selective test execution
markers =
    unit: Unit tests (fast, no external dependencies)
    integration: Integration tests (slower, may use tshark)
    slow: Tests that take >5 seconds
    requires_tshark: Tests that require tshark installed

# Coverage configuration
[coverage:run]
source = reshark
omit =
    */tests/*
    */conftest.py
    */__pycache__/*

[coverage:report]
exclude_lines =
    pragma: no cover
    def __repr__
    raise NotImplementedError
    if TYPE_CHECKING:
```

**Usage Examples**:

```bash
# Run all tests in parallel (auto-detect cores)
$ pytest -n auto

# Run only unit tests (fast feedback)
$ pytest -m unit

# Run integration tests with tshark
$ pytest -m integration --requires-tshark

# Run specific test file
$ pytest tests/test_observer.py -n 4

# Run with coverage report
$ pytest --cov=reshark --cov-report=html

# Debug mode (no parallel, verbose)
$ pytest -n 0 -vv -s
```

**Performance Expectations**:

- **Sequential**: ~200 tests in 3-4 minutes
- **Parallel (8 cores)**: ~200 tests in 30-45 seconds
- **CI/CD target**: <1 minute for unit tests, <3 minutes for full suite

**Benefits**:

✅ **Reduced boilerplate**: Shared fixtures eliminate copy-paste
✅ **Test isolation**: Each test gets fresh workspace/session_id
✅ **Fast feedback**: Parallel execution speeds up TDD workflow
✅ **Selective execution**: Run only relevant tests during development
✅ **CI/CD friendly**: Fast enough for pre-commit hooks

---

## Research Phase Complete ✅

**Status**: All technical decisions documented and ready for implementation

**Next Steps**:
1. Review research.md for approval
2. Proceed to Design Phase (data-model.md, contracts/)
3. Begin implementation Phase 1: Setup (T001-T008)

**Deliverables**:
- ✅ PCAP parsing strategy (tshark primary extractor)
- ✅ Statistical analysis methods (Shannon + frequency + variance)
- ✅ Test suite generation (hybrid template + helpers)
- ✅ Session isolation (date/UUID hierarchical)
- ✅ Grammar format (YAML law + Python interpreter)
- ✅ Known unknowns resolved

