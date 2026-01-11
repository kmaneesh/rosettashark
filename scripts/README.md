# Scripts - Manual Workflow Automation

**Phase**: Phase 1 (Manual orchestration)

## Purpose

Command-line scripts for manual workflow execution. These implement Phase 1 manual orchestration before LangGraph automation in Phase 2.

## Scripts

### run_session.py
Execute complete analysis workflow for a protocol.

**Usage**:
```bash
python scripts/run_session.py --pcap input.pcap --validation-pcaps val1.pcap val2.pcap val3.pcap
```

**Workflow**:
1. Create new session (generate UUID)
2. Run Observer scope (statistical analysis)
3. Display observations, wait for review
4. Run Theorist scope (hypothesis generation)
5. Display hypotheses, wait for review
6. Run Validator scope (3 PCAP validation)
7. Display validation results
8. If passed: prompt for Archivist promotion
9. If failed: log failures, suggest refinement

**Human Review Checkpoints**:
- After Observer: Review statistics before hypothesis generation
- After Theorist: Review hypotheses before validation
- After Validator: Review results before promotion

### validate_rulebook.py
Validate all grammars in Rulebook against their test suites.

**Usage**:
```bash
# Validate all grammars
python scripts/validate_rulebook.py

# Validate specific grammar
python scripts/validate_rulebook.py --grammar mqtt_v3
```

**Checks**:
- Grammar conforms to schema
- All validation tests pass
- Evidence files exist
- 3 PCAP minimum requirement met

### cleanup_sessions.py
Clean up old Notebook sessions.

**Usage**:
```bash
# List sessions older than 30 days
python scripts/cleanup_sessions.py --dry-run --days 30

# Archive completed sessions
python scripts/cleanup_sessions.py --archive --days 30

# Delete failed sessions
python scripts/cleanup_sessions.py --delete-failed
```

**Safety**:
- Dry-run mode shows what would be deleted
- Prompts for confirmation before deletion
- Preserves sessions with promoted grammars

## Development Workflow

### Quick Test
```bash
# Generate synthetic protocol
python scripts/generate_synthetic.py --output test.pcap

# Analyze it
python scripts/run_session.py --pcap test.pcap
```

### Full Validation
```bash
# Run on real protocol
python scripts/run_session.py \
  --pcap mqtt_capture.pcap \
  --validation-pcaps mqtt_test1.pcap mqtt_test2.pcap mqtt_test3.pcap

# Validate Rulebook integrity
python scripts/validate_rulebook.py
```

## Phase 2 Migration

These scripts will be replaced by `orchestration/graph.py` in Phase 2, but remain as:
- Debugging tools
- Step-by-step execution for troubleshooting
- Educational examples of workflow

## Constitutional Compliance

✅ All scripts respect scope boundaries  
✅ Human review checkpoints enforced  
✅ Minimum 3 PCAP requirement checked  
✅ Evidence trails maintained  
