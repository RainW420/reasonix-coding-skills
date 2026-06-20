# Paper Passport Schema

Paper Passport is the shared state contract for the paper skill system. Every paper skill accepts the current passport plus a task spec, and returns either no state change or a passport patch. A caller must reject patches that violate this schema or the write permissions below.

## Core Schema

```yaml
research_question: string
target_journal: string
paper_type: enum(research|article|review|short|letter)
paper_status: enum(ideation|planning|drafting|polishing|review|revision|final)

claims:
  - id: string
    text: string
    section: string
    evidence: [string]
    source_type: enum(user_data|source_claim|inference|material_gap|needs_confirmation)
    status: enum(supported|weak|unverified|contradicted)

evidence_candidates:
  - id: string
    type: enum(citation|experiment|user_provided|figure|calculation)
    ref: string
    proposed_for: [string]
    support_level: enum(direct|partial|background)

evidence_map:
  - id: string
    type: enum(citation|experiment|user_provided|figure|calculation)
    ref: string
    supports: [string]

citation_registry:
  - key: string
    bibtex: string
    support_level: enum(direct|partial|background)
    used_in: [string]

figure_contracts:
  - id: string
    conclusion: string
    archetype: string
    data_source: enum(user_provided|generated|public_dataset)
    status: enum(planned|drafted|polished|final)

section_status:
  introduction: enum(outlined|drafted|polished|verified|final)
  methods: enum(outlined|drafted|polished|verified|final)
  results: enum(outlined|drafted|polished|verified|final)
  discussion: enum(outlined|drafted|polished|verified|final)
  abstract: enum(outlined|drafted|polished|verified|final)

term_glossary:
  - term: string
    zh: string
    defined_in: string

known_gaps:
  - id: string
    description: string
    severity: enum(minor|moderate|critical)
    status: enum(open|mitigated|accepted|resolved)
    owner: enum(user|paper-write|paper-research|paper-figure|paper-citation|paper-polish|paper-debug|paper-verify)

review_findings:
  - id: string
    dimension: string
    score: int(0-100)
    severity: enum(minor|moderate|critical)
    comment: string

revision_history:
  - timestamp: string
    trigger: string
    action: string
    affected_sections: [string]
```

## Write Permissions

| Field group | Append only | Read/write | Bind authority |
|-------------|-------------|------------|----------------|
| claims | - | paper-write | - |
| evidence_candidates | paper-research | - | - |
| evidence_map | - | - | paper-write only |
| section_status | - | paper-write | - |
| citation_registry | paper-research, paper-citation | - | - |
| figure_contracts | - | paper-figure | - |
| review_findings | - | paper-review | - |
| term_glossary | - | paper-polish, paper-write | - |
| known_gaps | paper-debug | paper-verify | - |
| revision_history | all paper skills | paper-write, paper-response | - |

## State Flows

### Evidence Binding

1. `paper-research` and `paper-citation` add candidate evidence.
2. `paper-write` decides whether a candidate supports a claim.
3. Only `paper-write` moves an item into `evidence_map`.
4. `paper-verify` may mark a claim weak or unverified, but it must not rewrite the claim text.

### Gap Handling

1. Any skill may report a gap in its output notes.
2. `paper-debug` appends a gap when root cause analysis confirms it.
3. `paper-verify` updates `status` as open, mitigated, accepted, or resolved.
4. Critical open gaps force a checkpoint escalation.

### Review To Revision

1. `paper-review` writes `review_findings` only.
2. Critical findings block `paper-finalize`, not revision work.
3. `paper-response` produces a `revision_plan` and `proposed_patch`.
4. `paper-write` revision mode or the main session merges the final text.

