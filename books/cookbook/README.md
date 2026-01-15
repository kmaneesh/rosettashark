# Cookbook - Reusable Analysis Procedures

**Constitutional Reference**: Section 3.4 - Cookbook (Procedural Knowledge)

## Purpose

Reusable analysis procedures, statistical methods, and measurement techniques. This is NOT protocol-specific knowledge (which belongs in Rulebook).

## Procedures

### entropy_analysis.md
- Shannon entropy calculation
- Per-position entropy mapping
- Entropy threshold interpretation
- Use cases: Control vs data detection, encryption identification

### variance_analysis.md
- Positional variance calculation
- Standard deviation analysis
- Range analysis
- Use cases: Field type classification, counter detection

### alignment_strategies.md
- Message boundary detection
- Delimiter identification
- Length field discovery
- Frame alignment techniques

### state_machine_inference.md
- State transition detection
- Protocol flow analysis
- Session tracking
- Use cases: Connection-oriented protocols

### validation_patterns.md
- Test suite generation patterns
- Cross-PCAP validation strategies
- Negative test generation
- Evidence collection methods

## Adding New Procedures

1. Create `<procedure-name>.md` in cookbook/
2. Document: Purpose, Inputs, Algorithm, Outputs, Use Cases
3. Include code examples (implementation-agnostic)
4. Reference from scope mandates if used

## Constitutional Rules

**Allowed**:
- Analysis recipes and workflows
- Statistical method descriptions
- Tool usage templates
- Algorithm explanations

**Forbidden**:
- Protocol-specific grammars (→ Rulebook)
- Active hypotheses (→ Notebook)
- Tool outputs or measurements (→ Notebook)
- Vector embeddings (→ Patternbook)
