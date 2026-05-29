# English Output Templates

Use this file when `lang=en`. The main `SKILL.md` defines workflow rules; this file defines reusable wording blocks.

## Per-turn Footer

```markdown
**Current Phase:** ...
**Next Step:** ...
**What I Need From You:** ...
```

## Phase Confirmation Block (CONTEXT_SCAN / SPEC / SPLIT / IMPACT_SCAN / PLAN / REVIEW)

```markdown
-------------------
📌 Current Stage: [CONTEXT_SCAN | SPEC | SPLIT | IMPACT_SCAN | PLAN | REVIEW]

Please confirm whether the following is correct:
1. ...
2. ...

Reply with:
- "Confirm" / "OK" / "Continue" -> move to next phase
- Or provide revisions -> update this phase and reconfirm

Note: if you only send a short confirmation word, it is treated as phase approval.
-------------------
```

## Final Gate Before DOC

```markdown
-------------------
📌 About to enter DOC (generate final system analysis from template)

Confirmed phases: CONTEXT_SCAN / SPEC / SPLIT / IMPACT_SCAN / PLAN
Template to use: {template_path}
Planned outputs: {single_or_multi_docs}

Please confirm whether to generate final documents now:
- Reply "Confirm generation" / "Start drafting" -> enter DOC
- Reply with revisions -> go back to the relevant phase
-------------------
```

## Resume Prompt Block

```markdown
I found existing analysis history in `{artifact_root}`:
- Current state: {current_state}
- Last updated: {last_updated}

Please choose:
1. Resume history (resume)
2. Start over (restart)
```

## Artifact Dir Confirmation Block (default path only)

```markdown
-------------------
📁 Confirm artifact output directory (system-derived, not explicitly set by you)

- Parent dir of PRD/project (parent_dir): {parent_dir} (source: {parent_dir_source})
- Default base dir (base_dir): {base_dir}
- This session's artifact dir (artifact_root): {artifact_root}

Please confirm whether intermediate artifacts should be written under the path above:
- "Confirm" / "OK" / "Continue" -> keep this default directory
- Provide a new path -> use your directory instead (full replacement, no skill-name suffix appended)

Note: nothing is created or written until you confirm.
-------------------
```

## Phase-jump Entry Validation Block

```markdown
You asked to start from `{start_phase}`.

Entry validation result:
- Missing upstream artifacts: {missing_artifacts}
- Can continue directly: {can_continue}

Choose next step:
1. Provide missing artifacts and continue
2. Fall back to `{fallback_phase}`
3. Continue with your summary as temporary baseline
```

## Manual Edit Detected Block

```markdown
I detected manual edits in `{artifact_path}` (content fingerprint differs from the recorded `content_sha256` baseline).

Choose how to proceed:
1. adopt: continue with your edited version
2. merge: merge it with the current draft
3. regenerate: regenerate this phase from latest inputs

Note: if you reply only "OK/Confirm/Continue" without a strategy, default to adopt.
```

## Requirement Correction Rollback Block

```markdown
Detected requirement correction signal: {rollback_trigger}
Recommended rollback phase: {rollback_to_phase}
Impacted artifacts: {stale_artifacts}

Please choose:
1. Roll back and regenerate (recommended)
2. Continue with partial edits only (risk must be acknowledged)
```

## Artifact Naming

- `spec.en.md`
- `split.en.md`
- `context.en.md`
- `impact.en.md`
- `plan.en.md`
- `review.en.md`
- `prd-to-design.state.en.json`

## Stage Skeleton Templates

- SPEC: `templates/spec.md` (`en` section)
- SPLIT: `templates/split.md` (`en` section)
- CONTEXT_SCAN: `templates/context-scan.md` (`en` section)
- IMPACT_SCAN: `templates/impact-scan.md` (`en` section)
- PLAN: `templates/plan.md` (`en` section)
- DOC: `templates/doc.md` (`en` section)
- REVIEW: `templates/review.md` (`en` section)
