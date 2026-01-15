# Orchestration - Workflow Automation

**Phase**: Phase 2 (LangGraph-based autonomous orchestration)

**Phase 1 Status**: Manual orchestration via scripts (see `scripts/`)

## Purpose

Define and execute multi-scope workflows using LangGraph for autonomous protocol analysis.

## Files (Phase 2)

### graph.py
LangGraph workflow definition.

**Nodes**: Observer → Theorist → Validator → Archivist  
**Edges**: Conditional transitions based on state  
**State**: Shared memory and session context  

### state.py
Shared state model for workflow execution.

**State Fields**:
- `session_id`: Current session UUID
- `pcap_paths`: Input PCAPs
- `observations`: Observer results
- `hypotheses`: Theorist proposals
- `validation_results`: Validator outcomes
- `promoted_grammars`: Archivist outputs

### transitions.py
State transition logic and conditions.

**Transitions**:
- Observer complete → Theorist
- Theorist complete → Validator
- Validator passed → Archivist
- Validator failed → Theorist (refinement loop)
- Archivist complete → End

## Phase 1 Manual Orchestration

Phase 1 uses scripts in `scripts/` directory:

```bash
# Manual workflow (Phase 1)
python scripts/run_session.py --pcap input.pcap

# Steps:
# 1. Create session
# 2. Run Observer
# 3. Run Theorist
# 4. Run Validator (with 3 PCAPs)
# 5. Review results
# 6. Run Archivist (if passed)
```

## Phase 2 Autonomous Workflow

```python
# Autonomous workflow (Phase 2 - future)
from orchestration.graph import create_workflow
from orchestration.state import WorkflowState

workflow = create_workflow()
initial_state = WorkflowState(
    session_id=session_id,
    pcap_paths=["input.pcap"]
)

result = workflow.run(initial_state)
# Automatically: Observer → Theorist → Validator → Archivist
```

## Human-in-the-Loop (Phase 2)

Even in Phase 2, critical decisions require human approval:

- Hypothesis acceptance before validation
- Rulebook promotion confirmation
- Grammar deprecation decisions

## Constitutional Compliance

✅ Scopes respect authority boundaries  
✅ Memory boundaries enforced at transitions  
✅ Evidence trails maintained across workflow  
✅ Validation gates enforced (3 PCAP minimum)  
