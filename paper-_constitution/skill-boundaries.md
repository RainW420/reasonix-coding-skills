# Paper Skill Boundaries

This document defines control authority, inputs, outputs, and forbidden behavior for the paper skill system.

## Universal Contract

Every paper skill receives:

- `Paper Passport`
- task spec
- available artifacts
- requested mode, if any

Every skill returns:

- primary artifact, if any
- passport patch, if state changed
- verification notes
- unresolved gaps
- checkpoint recommendation

Reject any backend output that only returns prose when it should update the passport.

## Authority Levels

| Role | Skills | Authority |
|------|--------|-----------|
| Router | paper-superpowers, paper-task-modes | choose workflow and required skills |
| Mainline | paper-brainstorm, paper-plan, paper-write | own research design, structure, claims, and final prose |
| Gates | paper-verify, paper-debug, paper-review | judge or diagnose; do not silently rewrite |
| Workers | paper-research, paper-polish, paper-citation, paper-figure, paper-response, paper-finalize, paper-read, paper-ppt | perform bounded specialist tasks |
| Horizontal | paper-ponytail, paper-provenance | constrain all other skills |

## Adapter Rules

External repositories are backend capabilities only. They must be wrapped by adapters under `paper-_adapters`.

1. A backend cannot bypass `paper-superpowers`.
2. A backend must accept Paper Passport plus task spec.
3. A backend must return artifact plus passport patch plus verification notes.
4. Completion claims must pass `paper-verify`.
5. Large references must be lazy-loaded.

## Forbidden Behavior

- Research workers must not bind evidence to claims.
- Review workers must not rewrite manuscript text.
- Polish workers must not change claims, data, figures, or citations.
- Citation workers must not invent bibliographic records.
- Figure workers must not fabricate data.
- Finalize must not proceed with critical review findings unless a forced checkpoint approves it.
- Any skill seeing `MATERIAL GAP` or `NEEDS CONFIRMATION` above threshold must escalate.

## Local Paths

- Skills: `/home/rainw/.reasonix/skills/paper-*/SKILL.md`
- Constitution: `/home/rainw/.reasonix/skills/paper-_constitution/`
- References: `/home/rainw/.reasonix/skills/paper-_references/`
- Adapters: `/home/rainw/.reasonix/skills/paper-_adapters/`
- Academic base repo: `/home/rainw/paper-base-repos/academic-research-skills/`
- Nature base repo: `/home/rainw/paper-base-repos/nature-skills/`

If a base repo is missing, report the missing path and continue with the local constitution rather than inventing imported rules.

