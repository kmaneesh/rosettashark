# Tools - Scripts and Utilities

**Version**: 2.0.0  
**Purpose**: Executable scripts and utility wrappers  
**Constitutional Reference**: Section 6 - Tools and Capabilities

---

## Overview

Tools are thin wrappers around external programs and reusable utility scripts. They provide a consistent interface for analysis operations across both HITL and Autonomous modes.

## Design Principle

**Tools are plumbing, not brains.**

- Tools execute external programs (tshark, scapy, etc.)
- Tools capture and return output
- Tools handle errors and timeouts
- Tools DO NOT interpret results (that's for Agents/Skills)
- Tools are stateless and idempotent

---

## Structure

```
tools/
├── scripts/                # Executable scripts
│   ├── pcap_parser.py
│   ├── binary_analyzer.py
│   └── format_converter.py
├── __init__.py
├── pcap_tools.py          # PCAP-specific wrappers
├── binary_tools.py        # Binary analysis wrappers
└── statistics.py          # Statistical utilities
```

---

## Tool Categories

### 1. PCAP Tools (`pcap_tools.py`)

Wrappers for PCAP analysis programs.

**TsharkWrapper**
```python
from reshark.tools.pcap_tools import TsharkWrapper

tshark = TsharkWrapper()

# Extract streams
streams = tshark.extract_streams(
    pcap_path="data/sample.pcap",
    filter_expr="tcp.port == 443"
)

# Reassemble TCP stream
stream_data = tshark.reassemble_tcp_stream(
    pcap_path="data/sample.pcap",
    stream_id=0
)

# Extract specific fields
fields = tshark.extract_fields(
    pcap_path="data/sample.pcap",
    fields=["ip.src", "ip.dst", "tcp.port"]
)
```

**ScapyWrapper**
```python
from reshark.tools.pcap_tools import ScapyWrapper

scapy = ScapyWrapper()

# Read PCAP
packets = scapy.read_pcap("data/sample.pcap")

# Filter packets
filtered = scapy.filter_packets(packets, lambda p: p.haslayer("TCP"))

# Extract payload
payloads = scapy.extract_payloads(packets)
```

### 2. Binary Tools (`binary_tools.py`)

Binary data analysis utilities.

**HexDumpWrapper**
```python
from reshark.tools.binary_tools import HexDumpWrapper

hexdump = HexDumpWrapper()

# Get hex representation
hex_output = hexdump.format_hex(
    data=binary_data,
    format="canonical"  # or "one-byte", "two-byte"
)
```

**BinutilsWrapper**
```python
from reshark.tools.binary_tools import BinutilsWrapper

binutils = BinutilsWrapper()

# Examine binary structure
info = binutils.file_info("data/binary_sample")

# Extract strings
strings = binutils.extract_strings(
    filepath="data/binary_sample",
    min_length=4
)
```

### 3. Statistical Tools (`statistics.py`)

Statistical analysis utilities.

**EntropyCalculator**
```python
from reshark.tools.statistics import EntropyCalculator

entropy = EntropyCalculator()

# Shannon entropy
ent = entropy.shannon_entropy(data_bytes)

# Per-position entropy map
entropy_map = entropy.position_entropy(
    streams=[stream1, stream2, stream3]
)
```

**FrequencyAnalyzer**
```python
from reshark.tools.statistics import FrequencyAnalyzer

freq = FrequencyAnalyzer()

# Byte frequency distribution
dist = freq.byte_frequency(data_bytes)

# N-gram analysis
ngrams = freq.ngram_frequency(data_bytes, n=2)
```

### 4. Scripts (`scripts/`)

Standalone executable scripts for specific tasks.

**pcap_parser.py**
```bash
# Parse PCAP and output JSON
python tools/scripts/pcap_parser.py \
  --input data/sample.pcap \
  --output parsed.json \
  --format streams
```

**binary_analyzer.py**
```bash
# Analyze binary file structure
python tools/scripts/binary_analyzer.py \
  --input data/binary_sample \
  --output analysis.json
```

**format_converter.py**
```bash
# Convert between formats
python tools/scripts/format_converter.py \
  --input data/sample.pcap \
  --output data/sample.pcapng \
  --from pcap \
  --to pcapng
```

---

## Tool Base Class

All tool wrappers should follow this pattern:

```python
from abc import ABC, abstractmethod
from typing import Dict, Any
import subprocess

class Tool(ABC):
    """Base class for tool wrappers"""
    
    def __init__(self, executable_path: str = None):
        self.executable = executable_path or self._default_executable()
        self._verify_installation()
    
    @abstractmethod
    def _default_executable(self) -> str:
        """Return default executable name"""
        pass
    
    def _verify_installation(self) -> bool:
        """Verify tool is installed"""
        try:
            subprocess.run([self.executable, "--version"],
                         capture_output=True, check=True)
            return True
        except (subprocess.CalledProcessError, FileNotFoundError):
            raise RuntimeError(f"{self.executable} not found")
    
    def run(self, params: Dict[str, Any]) -> Dict[str, Any]:
        """Execute tool with parameters"""
        pass
```

---

## Error Handling

Tools should handle errors gracefully:

```python
class TsharkWrapper:
    def extract_streams(self, pcap_path, filter_expr=None):
        try:
            result = subprocess.run(
                cmd,
                capture_output=True,
                text=True,
                timeout=300,  # 5 minute timeout
                check=True
            )
            return self._parse_output(result.stdout)
            
        except subprocess.TimeoutExpired:
            return {"error": "timeout", "duration": 300}
        except subprocess.CalledProcessError as e:
            return {"error": "execution_failed", "stderr": e.stderr}
        except FileNotFoundError:
            return {"error": "pcap_not_found", "path": pcap_path}
```

---

## Tool Registry

Tools are registered for easy discovery:

```python
# reshark/tools/registry.py

class ToolsRegistry:
    """Central registry for all tools"""
    
    def __init__(self):
        self._tools = {}
        self._register_builtin_tools()
    
    def _register_builtin_tools(self):
        self.register("tshark", TsharkWrapper())
        self.register("scapy", ScapyWrapper())
        self.register("entropy", EntropyCalculator())
    
    def register(self, name: str, tool: Tool):
        """Register a tool"""
        self._tools[name] = tool
    
    def get(self, name: str) -> Tool:
        """Get tool by name"""
        return self._tools.get(name)
```

---

## Usage in Agents

Agents invoke tools through the registry:

```python
class PatternAnalyzer(Agent):
    def execute(self, inputs):
        # Invoke tshark tool
        streams = self.invoke_tool("tshark", {
            "pcap_path": inputs["data_path"],
            "filter": "tcp"
        })
        
        # Invoke entropy calculator
        entropy = self.invoke_tool("entropy", {
            "data": streams["raw_bytes"]
        })
        
        return {"streams": streams, "entropy": entropy}
```

---

## Testing Tools

```python
# tests/test_tools/test_tshark.py

import pytest
from reshark.tools.pcap_tools import TsharkWrapper

def test_tshark_extract_streams():
    tshark = TsharkWrapper()
    result = tshark.extract_streams("tests/data/test.pcap")
    
    assert result["status"] == "success"
    assert len(result["streams"]) > 0

def test_tshark_timeout():
    tshark = TsharkWrapper()
    result = tshark.extract_streams(
        "tests/data/huge.pcap",
        timeout=1  # Force timeout
    )
    
    assert result["error"] == "timeout"
```

---

## Constitutional Compliance

Tools must follow Constitutional guidelines:

1. **No Interpretation**: Return raw/structured data, no analysis
2. **Transparent**: Log all invocations
3. **Isolation**: Run in sandboxed environment
4. **Read-Only**: Never modify input data
5. **Audit Trail**: Record all tool executions

---

## Adding New Tools

1. Create wrapper class inheriting from `Tool`
2. Implement required methods
3. Add to registry
4. Write tests
5. Document in this README

Example:
```python
# reshark/tools/custom_tool.py

from reshark.tools.base import Tool

class MyTool(Tool):
    def _default_executable(self):
        return "mytool"
    
    def run(self, params):
        # Implementation
        return results
```

---

## References

- [System Implementation Technical Design](../specs/system-implementation-technical-design.md)
- [Agents README](../agents/README.md)
- [Skills README](../skills/README.md)

```python
read_pcap(path) -> list[bytes]
write_pcap(path, messages) -> None
merge_pcaps(input_paths, output_path) -> None
filter_pcap(input_path, output_path, filter_expr) -> None
```

## Error Handling

All tools MUST handle:
- Missing tool binaries (raise ToolNotFoundError)
- Corrupted input files (raise CorruptedFileError)
- Timeouts (raise ToolTimeoutError)
- Non-zero exit codes (raise ToolExecutionError)

## Tool Requirements

Document tool dependencies in `docker/requirements.txt`:

```
# System tools (installed via apt)
tshark >= 3.0
zeek >= 5.0  # Phase 2
```

## Adding New Tools

1. Create `<tool>.py` wrapper
2. Document functions and error handling
3. Add to Docker image
4. Add integration tests
5. Update this README

## Constitutional Compliance

✅ Tools execute in sandboxed environment (Docker)  
✅ All tool invocations logged  
✅ Tool outputs stored in Notebook (not Rulebook)  
✅ Tools NEVER make protocol claims  
