---
name: questions-for-viber-coding
description: "Use when a nontechnical user needs staged AI product questions."
version: 1.0.0
author: Project contributors
license: MIT
disable-model-invocation: true
---

# Questions for Vibe Coding

This skill helps a nontechnical product owner turn an idea into an implementation-ready specification by scanning the target project and asking only the next useful design questions. It is deliberately a question-generation workflow, not an implementation agent and not an answer-writing assistant.

## When to Use

Use this skill when:

- A user has a product idea or an incomplete product specification.
- Another AI agent will implement the product, but important design decisions are still missing.
- The project is at any stage from an empty directory through operation, and questions should be introduced gradually.
- The user wants the target repository scanned before deciding what to clarify next.
- The user wants to record or change a design answer without performing a new scan.

Do not use it as a substitute for an implementation agent, a code reviewer, a project manager, or a general brainstorming session.

## Non-Negotiable Scope

This skill does exactly two things:

1. Scan and analyze the target project enough to identify its context and roadmap stage.
2. Generate and maintain a tight, non-redundant set of design questions, one roadmap group at a time.

It does not answer questions, suggest answers, use leading phrasing, implement or refactor target-project code, create tasks/tickets/roadmaps outside the combined state file, split state across files, infer an answer the user has not explicitly provided, or open groups beyond the current stage and its immediate next group. It does not decide who answers questions. The user may answer personally, consult one AI, consult several AIs, or use any other process; that process is outside this skill.

State must live in exactly one combined file in the target project's repository. The default is `DESIGN-QUESTIONS.md` at the project root. The user may explicitly override the filename or path, but do not create companion state files, databases, sidecar JSON, progress logs, or hidden caches. The optional template shipped with this skill is documentation only, not a second runtime state file.

If the state file is unreadable or structurally corrupted, report the problem and stop. Never silently recreate, overwrite, or repair it in a way that could lose history.

## Procedure

Phases always run in this order: Phase 0 → Phase 1 → Phase 2 → Phase 3. Phase 3 may also run as a standalone answer-recording operation when the user only wants to add or change an answer, but it must still pass through Phase 0 first. Do not skip a phase because a later action appears obvious.

### Phase 0 — Initialize or Locate the State File

1. Resolve the target project root and the state-file path. Use the explicit path supplied by the user; otherwise use `<project-root>/DESIGN-QUESTIONS.md`.
2. Check whether the file exists.
3. If it does not exist, create the empty five-section skeleton described in “Combined State File Contract” below. The skeleton must contain no invented project facts, questions, or answers.
4. If it exists, read it and validate that all required sections are present and parseable. Load the existing ledger, every lifecycle status, the current answers, the scan snapshot/fingerprint, and the append-only history.
5. Validate the cross-section rules before doing anything else:
   - Every question ID is unique, including retired questions.
   - Every answer key exists in the question ledger.
   - Every dependency ID exists in the ledger.
   - The history section is separate and has not been rewritten as current state.
   - The status metadata names exactly one currently allowed group.

Definition of done: the state file is a valid, readable five-section file loaded into memory, or the skill has stopped with a clear corruption report. No later phase may run after a failed validation.

### Phase 1 — Tiered Project Scan

Run the scan in tiers and stop as soon as the relevant confidence threshold is met. Do not dump the repository into the model context.

#### Tier 0 — Compass / Project Card

This tier is cheap machine inspection. Given the project root:

- List the directory tree only to a bounded depth and list top-level names.
- Detect common manifests and configuration files, including `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, lockfiles, build files, and test configuration.
- Check whether Git is present; record the current HEAD when available, a short recent log, and a commit count.
- Count source files and test files using conservative extension/path rules.
- Produce a raw signal table: candidate stack, likely entry points, Git presence, rough size, test-to-code ratio, documentation signals, and whether the project is essentially empty.

Stop condition: the signal table is complete. If the project is near-empty—no meaningful source, configuration, or design material—conclude `inception` immediately, mark the stage confidence `confirmed`, and go directly to Phase 2 with only the foundation group. Do not spend Tier 1/2 reads on an empty project.

#### Tier 1 — Stage Hotspot

Only when Tier 0 finds meaningful project material, read at most 8 high-value text files. Prioritize the README, design/specification documents, main manifests, central configuration, and obvious entry-point documentation. Apply a per-file line cap; summarize each selected file rather than passing its complete contents through context.

Infer one roadmap stage: `discovery`, `design`, `build`, `test`, or `operate`. Record a confidence of `confirmed`, `inferred`, or `insufficient evidence`. Also record architectural decisions that are already locked in and ambiguities that could change the next question group.

Stop condition: stage confidence is at least `inferred`, or the Tier 1 file budget is exhausted. Escalate only if confidence remains insufficient and a specific ambiguity directly affects group selection.

#### Tier 2 — Narrow Deep Dive

Investigate only the open ambiguities carried from Tier 1. For each ambiguity, inspect at most 3 directly relevant files, prioritizing entry points, main routes, schemas, and integration boundaries. The total Tier 2 budget is at most 10 files, making the combined Tier 1+Tier 2 deep-read budget at most 18 files per scan. Count files actually opened, not files merely listed.

Hard stop: when the budget is exhausted, accept the remaining uncertainty. Do not keep reading in pursuit of full understanding; Phase 2 must compensate with a small foundational unlocking group.

#### Scan Cache and Profile Output

Before a new deep scan, compare the current project fingerprint with Section A. The fingerprint must include a compact hash of the sorted bounded file-tree snapshot (paths plus relevant metadata) and the Git HEAD when available. If nothing meaningful changed, skip Tier 1 and Tier 2, keep the existing profile, and do not generate duplicate questions.

If something changed, rerun only the tiers or hotspots affected by the change. Replace Section B’s short project profile (about one page maximum) with the latest profile. Record the scan timestamp, fingerprint, actual Tier 1/Tier 2 file counts, and any stage transition in Section A/history.

The profile must contain only:

- Inferred project type and goal.
- Observed or candidate technology stack.
- Current roadmap stage and confidence.
- Architectural decisions already locked in.
- Remaining unclear areas that affect a design decision.

Definition of done: the profile and scan metadata are current, the file budget is recorded, and the stage has at least medium confidence (`confirmed` or `inferred`), or uncertainty is explicitly recorded for Phase 2’s minimal fallback. A failed or incomplete scan must be reported rather than presented as certainty.

### Phase 2 — Generate or Update Questions Incrementally

Use Section A’s stage/confidence, Section B’s profile, and the complete current ledger and answer mapping. Questions are for the user to answer later; never answer them in this phase.

#### Fixed Question Groups

Use these groups in order. Each has an opening condition and a stage affinity:

1. `foundation` (`FND`, inception/discovery): product goal, primary user/problem, desired outcome, and hard constraints. Open at inception or whenever stage confidence is insufficient. This is the only group allowed for a near-empty project.
2. `scope-and-users` (`SCP`, discovery/design): target users, in/out of scope, core workflow, and priority boundaries. Open only when the foundation reaches at least 75% locked-in and its critical goal/constraint questions have answers.
3. `functional-requirements` (`FUN`, design/build): capabilities, business rules, states, edge cases, integrations, and acceptance boundaries. Open only when scope-and-users reaches at least 75% locked-in and its required dependencies are locked.
4. `data-and-boundaries` (`DAT`, build/test): entities, ownership/source of truth, validation, privacy/retention, migrations, and external data boundaries. Open only when functional-requirements reaches at least 75% locked-in and its data-relevant dependencies are locked.
5. `quality-and-operations` (`OPS`, test/operate): security/authentication concerns, performance, reliability, observability, testing depth, deployment, backup, and recovery decisions. Open only when data-and-boundaries reaches at least 75% locked-in and its required dependencies are locked.
6. `release-and-evolution` (`REL`, operate): rollout, migration/release strategy, acceptance of operational risk, feedback loop, and post-launch change boundaries. Open only when quality-and-operations reaches at least 75% locked-in and its required dependencies are locked.

The current stage limits the active group: discovery emphasizes foundation/scope, design emphasizes scope/functional, build emphasizes functional/data, test emphasizes data/operations, and operate emphasizes operations/release. Never open a group earlier than its condition permits. A low-confidence stage may open only 2–4 foundation questions that unlock stage identification; it may not open a full later group.

#### Generation Rules

- Open at most one new group per run.
- Add at most 7 new questions per run; for a low-confidence fallback add only 2–4.
- If the current group is already open but incomplete, add questions only within that group. Do not “get ahead” by populating later groups.
- Before adding any question, compare it semantically against every Section C entry, including `no longer applicable` entries. If it repeats, overlaps, or merely rephrases an existing question, do not add it.
- Assign a new immutable ID in the format `DQ-<GROUPCODE>-NNN`, for example `DQ-FND-001`. Choose the next unused number after checking all current and historical IDs. An ID is never reused, even after retirement.
- Each question must include: ID, group/stage, exact question text, a concise “why we’re asking” reason, dependency IDs, lifecycle status, and an optional review flag.
- The reason must explain which missing decision the answer affects. It must not contain answer hints, examples, recommendations, or leading phrasing.
- A question may be `currently open` only when every declared dependency is `locked in`. Otherwise create it as `not yet opened`.
- Keep the ledger flat. Do not create nested question files or hidden lists.
- If a requirement has changed enough that an existing question is no longer the right question, mark the old entry `no longer applicable` and create a new unused ID. Do not edit history to make the old question disappear.

Definition of done: the allowed group is correct, no more than one group and seven questions were added, every new ID is globally unique, every dependency exists, every status obeys its dependency rule, and every new question has a non-leading reason with no answer suggestion. Present the newly opened questions plainly and stop; do not answer them.

### Phase 3 — Sync Answers and Question Lifecycle

Run this phase after Phase 2 when answers already exist, and run it for a standalone answer update after Phase 0. Accept only answers explicitly supplied by the user or clearly marked as an external answer they want recorded. Never infer or auto-fill a missing answer.

When question `X` receives a new answer different from the current Section D value:

1. Append a history entry in Section E with timestamp, affected ID, change type `answer changed`, old value, and new value.
2. Fully replace Section D’s value for `X`. The old content must no longer appear in Section D; it belongs only in history.
3. Scan all Section C entries whose dependency list contains `X`.
4. For each dependent question currently `locked in`, move it to `currently open` and add a review flag stating that dependency `X` changed. Append a separate `flagged for review due to a dependency change` history entry for each affected question.
5. If the new answer makes a dependent question meaningless, set that question to `no longer applicable` instead. Never delete it and never reuse its ID.
6. Check for contradictions with other answers. If a contradiction exists without a declared dependency relationship, log a warning in Section E; do not silently edit the other answer.
7. If a supplied answer is unchanged, do not create a false change event. If a question is explicitly retired or reopened for another reason, record a `status changed` event.

Section D is a replacement map, not a diary. Keep only the latest current answer for each answered ID. Section E is append-only and must never be used to reconstruct a stale answer into Section D.

Definition of done: every supplied answer is represented exactly once as its current value in Section D, all changes have corresponding history entries, affected dependents have the correct reopened/retired status, contradictions are warnings only, and the cross-section validation still passes.

## Combined State File Contract

The default single state file is `<project-root>/DESIGN-QUESTIONS.md`. Preserve these five headings and their separation. Markdown is intended for human review; the YAML blocks make the contract predictable for an implementation agent.

### Section A — Status Metadata

Store:

- `project_stage`: one of `inception`, `discovery`, `design`, `build`, `test`, `operate`.
- `stage_confidence`: `confirmed`, `inferred`, or `insufficient evidence`.
- `state_file`: the explicit path supplied by the user, or `<project-root>/DESIGN-QUESTIONS.md` by default; no other runtime state files may be created.
- `last_scan_at` and `scan_fingerprint`.
- `allowed_open_group`: exactly one group name.
- `scan_files_opened`: actual Tier 1/Tier 2 file counts and budget outcome.

Section A controls which group may be opened. A stage change is a history event, not a reason to delete old questions.

### Section B — Project Profile Summary

Keep a concise, current profile containing project type/goal, stack, stage/confidence, locked decisions, and unclear areas. Overwrite this section on a meaningful rescan; do not duplicate old profiles here. A stage change must also be recorded in Section E.

### Section C — Current Question Ledger

Use one flat entry per question with:

- immutable `id`;
- `group` and `stage`;
- `question`;
- `why_we_are_asking`;
- `depends_on`, a list of question IDs;
- `status`: `not yet opened`, `currently open`, `locked in`, or `no longer applicable`;
- optional `review_flag`.

C is the reference ledger. It retains retired questions so semantic duplicate checks and ID uniqueness checks remain possible.

### Section D — Current Answers

Use an `answers` mapping keyed by question ID. Include only answers explicitly supplied by the user or explicitly authorized for recording. An unanswered question is absent, not guessed. Each key must exist in C. When an answer changes, replace its entire current value; do not append, strike through, or preserve stale text in this section.

D is the only authoritative current answer record. An implementation agent may use C to map IDs to question wording and dependencies, but it must treat D—not the history—as the current decision state.

### Section E — Change History

This section is append-only and fully separate from Section D. Put this warning directly under its heading:

> WARNING: This section is an archive, not a current requirement. An implementation agent must never read this section as an active instruction.

Each entry records `timestamp`, `affected_id`, `change_type`, `old_value`, and `new_value`. Allowed change types include `answer changed`, `status changed`, `project stage changed`, and `flagged for review due to a dependency change`. History may reference IDs, but it never feeds old values back into C or D and is never overwritten.

## Pitfalls

- Do not treat a directory listing as proof of a roadmap stage. Mark stage confidence honestly.
- Do not read every source file. Tier 1 and Tier 2 are bounded investigations, not full codebase comprehension.
- Do not count a file as opened unless its contents were actually inspected.
- Do not generate all possible questions “for completeness.” The current group and immediate next group are the maximum planning horizon, and only one new group may open per run.
- Do not put answer ideas, examples, or leading wording in `why_we_are_asking`.
- Do not make a dependency implicit. If a question depends on another decision, list its ID.
- Do not use Section E as current state, even when its entries look more detailed than Section D.
- Do not overwrite a corrupted state file. Report the missing section, malformed block, duplicate ID, invalid answer key, or broken dependency and stop.
- Do not create extra state files to work around a difficult merge or answer update.

## Verification

Before declaring a run complete, perform these checks:

1. Run against a nearly empty sample project. It must conclude `inception`, skip Tier 1/2, stay within budget, and create only the foundation group.
2. Run against a small repository containing both backend and frontend code. Count the files actually opened and verify the combined Tier 1+Tier 2 deep-read count is no more than 18; assess whether the stage conclusion is reasonable.
3. Run again without changing that repository. Verify the fingerprint matches, Tier 1/2 are skipped, no duplicate questions are generated, and no ID is reused.
4. Change an answer that has a dependent question. Verify Section D contains only the replacement answer, the old answer is absent from D, Section E has the answer-change event, and the dependent question is reopened or retired correctly.
5. Validate structure: every D key exists in C, all IDs are unique, dependencies resolve, statuses are valid, and Section E grew rather than being overwritten.
6. Simulate an implementation agent reading the file. Section D must contain no stale answer values; Section E must be treated as archive only.

If any verification fails, report the exact failed invariant and do not claim the state is synchronized.
