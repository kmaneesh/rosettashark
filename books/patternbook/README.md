# Patternbook - Similarity and Analogy Memory

**Constitutional Reference**: Section 3.3 - Patternbook (Similarity and Analogy Memory)

**Phase**: Phase 2 only (not implemented in Phase 1 MVP)

## Purpose

Non-authoritative storage for protocol similarities, analogies, and exploration hints. This is ADVISORY ONLY - never ground truth.

## Structure

```
patternbook/
└── embeddings/          # Vector database (Chroma/Qdrant/FAISS)
    ├── protocol_behaviors.db
    ├── field_patterns.db
    └── state_transitions.db
```

## Authority Level

⚠️ **CRITICAL**: Patternbook entries are advisory only and MUST NOT be treated as ground truth.

Any pattern-derived hypothesis MUST be validated before promotion to Rulebook.

## Allowed Contents (Phase 2)

✅ Vector embeddings of protocol behaviors  
✅ Similarity scores between protocol features  
✅ Clustering results and protocol family groupings  
✅ Suggested analogies for new protocol analysis  

## Forbidden Contents

❌ Protocol grammars or specifications (→ Rulebook)  
❌ Claims of protocol equivalence without validation  
❌ Raw PCAP data or byte sequences (processed only)  

## Usage Pattern (Phase 2)

```python
# Query similar protocols
similar = patternbook.find_similar(new_protocol_features, k=5)

# Generate hypothesis from similarity (ADVISORY)
hypothesis = theorist.generate_from_similarity(similar)

# MUST validate before trust
validation_result = validator.validate(hypothesis, pcaps)

if validation_result.passed:
    archivist.promote_to_rulebook(hypothesis)
```

## Phase 1 Status

**NOT IMPLEMENTED** in Phase 1 MVP. Directory structure exists for future use.

Phase 1 focuses on:
- Manual hypothesis generation (Theorist)
- Evidence-based validation (Validator)
- Explicit promotion gates (Archivist)
