# Tools - External Tool Wrappers

**Constitutional Reference**: Section 6 - Tools and Capabilities

## Purpose

Thin wrappers around external analysis tools. NO LOGIC HERE - just invocation and output capture.

## Design Principle

**Tools are plumbing, not brains.**

- Tools execute external programs (tshark, zeek, etc.)
- Tools capture and return output
- Tools handle errors and timeouts
- Tools DO NOT interpret results (that's for Scopes)

## Files

### tshark.py
Wrapper for tshark PCAP analysis.

**Functions**:
```python
extract_streams(pcap_path, output_format='json') -> str
reassemble_tcp_stream(pcap_path, stream_id) -> bytes
extract_fields(pcap_path, fields) -> list[dict]
get_conversations(pcap_path, protocol='tcp') -> list[dict]
```

**Example**:
```python
from tools.tshark import extract_streams

# Extract TCP streams as JSON
streams_json = extract_streams("input.pcap", output_format="json")

# Reassemble specific stream
stream_0 = reassemble_tcp_stream("input.pcap", stream_id=0)
```

### zeek.py (Phase 2)
Wrapper for Zeek protocol analyzer.

**Status**: Not implemented in Phase 1

### pcap_io.py
PCAP file operations (read, write, merge, filter).

**Functions**:
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
