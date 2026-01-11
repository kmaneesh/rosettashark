# Scopes - Agent Mandates

**Constitutional Reference**: Section 5 - Scopes (Agent Authority)

## Purpose

Scopes define bounded agent roles with specific analysis authority. Each scope has a declared domain and may not exceed its mandate.

## Scope Definitions

### observer.md
Mandate: Statistical analysis of PCAP data
- Authority: Entropy calculation, frequency analysis, invariant detection
- Reads: PCAPs (via tools)
- Writes: Notebook (observations only)
- Forbidden: Hypothesis generation, grammar creation

### theorist.md
Mandate: Hypothesis generation from observations
- Authority: Pattern inference, boundary detection, field structure proposals
- Reads: Notebook (Observer results)
- Writes: Notebook (hypotheses only)
- Forbidden: Validation, Rulebook promotion

### validator.md
Mandate: Cross-PCAP hypothesis testing
- Authority: Test suite generation, validation execution, negative testing
- Reads: Notebook (hypotheses), PCAPs (via tools)
- Writes: Notebook (validation results)
- Forbidden: Hypothesis modification, Rulebook promotion

### archivist.md (Phase 1)
Mandate: Memory management and promotion
- Authority: Rulebook promotion, session archival, grammar generation
- Reads: Notebook (validated hypotheses), Rulebook (existing grammars)
- Writes: Rulebook (promoted grammars), Notebook (archival marks)
- Forbidden: Hypothesis generation, validation execution

## Creating New Scopes

1. Define mandate in `<scope-name>.md`
2. Specify authority boundaries
3. Document read/write permissions
4. List forbidden operations
5. Add to orchestration graph

## Constitutional Requirements

- Each scope MUST enforce its declared authority
- Scopes MAY read from any memory system
- Scopes MAY write only to Notebook (unless promotion authority)
- Only Archivist has Rulebook write authority (Phase 1)
