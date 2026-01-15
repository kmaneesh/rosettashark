# Notebook - Session-Scoped Working Memory

**Constitutional Reference**: Section 3.1 - Notebook (Short-Term Working Memory)

## Purpose

Transient storage for hypotheses, intermediate statistics, and experimental results. Ephemeral and session-scoped.

## Structure

```
notebook/
└── sessions/
    └── YYYY-MM-DD/                    # Date directory (UTC)
        └── YYYY-MM-DDTHH-MM-SSZ__<uuid>/  # Session directory
            ├── manifest.json          # Session metadata
            ├── hypotheses.json        # Generated hypotheses
            ├── byte_stats.csv         # Statistical observations
            ├── failures.log           # Validation failures
            └── artifacts/             # PCAP outputs, intermediate data
```

## Session Naming Convention

**Format**: `YYYY-MM-DDTHH-MM-SSZ__<uuid4>`

**Example**: `2026-01-14T10-32-41Z__a9f3c891-4b2e-4d3f-9a12-3c5d8e9f1234`

**Rules**:
- Timestamp: ISO 8601 format, UTC timezone
- UUID: Standard UUID4 format (lowercase with hyphens)
- Separator: Double underscore `__`
- No user-specified names allowed
- No custom formats allowed

## Session Lifecycle

### States

1. **active**: Session directory exists, no completion marker
2. **completed**: Contains `.completed` marker file
3. **archived**: Moved to archive location (Phase 2)

### manifest.json Format

```json
{
  "session_id": "a9f3c891-4b2e-4d3f-9a12-3c5d8e9f1234",
  "created_at": "2026-01-14T10:32:41Z",
  "pcap_paths": ["input1.pcap", "input2.pcap"],
  "status": "active",
  "user": "researcher",
  "protocol_hint": "mqtt_v3"
}
```

## Allowed Contents

✅ Active hypotheses under investigation  
✅ Statistical measurements (entropy, frequency, variance)  
✅ Partial field boundaries and candidate control bytes  
✅ Validation attempts and outcomes  
✅ Agent reasoning traces and decision logs  

## Forbidden Contents

❌ Validated protocol grammars (→ Rulebook)  
❌ Similarity patterns or embeddings (→ Patternbook)  
❌ Long-term historical data (must be archived or promoted)  

## Retention Policy

**Session Cleanup** (manual in Phase 1):
- Review sessions older than 30 days
- Archive completed sessions
- Delete failed/abandoned sessions

**Promotion**:
- Validated hypotheses → Rulebook (via Archivist)
- Session marked as `.completed`
- Evidence trail preserved in Rulebook metadata

## Creating Sessions

```python
from reshark.utils.session import SessionManager

manager = SessionManager(workspace_root)
session_id = manager.create_session(pcap_paths=["test.pcap"])
# Returns: "a9f3c891-4b2e-4d3f-9a12-3c5d8e9f1234"

session_path = manager.get_session_path(session_id)
# Returns: Path("notebook/sessions/2026-01-14/2026-01-14T10-32-41Z__a9f3c891-.../")
```
