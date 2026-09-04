---
name: questions-for-viber-coding
description: "Use when a nontechnical user needs staged AI product questions."
version: 1.2.0
author: Project contributors
license: MIT
disable-model-invocation: true
---

# Questions for Vibe Coding

This skill helps a nontechnical product owner turn an idea into an implementation-ready specification by scanning the target project and asking only the next useful design questions. It is deliberately a question-generation workflow, not an implementation agent — but for a project that is already mature when the skill first runs on it, its first initializing run also digs into what the project actually does today (its real code and configuration, not only its prose) and records the decisions it has already, explicitly made, so the state file starts as a true, concrete current picture instead of a generic template pretending a built project is still at square one.

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

1. Scan and analyze the target project enough to identify its context and roadmap stage — for an already-mature project, this includes reading enough of the real implementation to ground questions and answers in what actually exists, not only in what its documentation claims.
2. Generate and maintain a tight, non-redundant set of design questions — one roadmap group at a time in the normal case, or every group in a single pass during the one-time Bootstrap Mode described below.

It does not use leading phrasing, implement or refactor target-project code, create tasks/tickets/roadmaps outside the combined state file, split state across files, or open groups beyond what the current mode permits. It does not decide who answers an open question. The user may answer personally, consult one AI, consult several AIs, or use any other process; that process is outside this skill. It never answers a question and never records an answer unless that answer is either explicitly supplied by the user, or — only inside Bootstrap Mode — explicitly and unambiguously grounded in the target project's own materials (documentation or its actual implementation), with its source cited. It never guesses, extrapolates, or infers an answer from weak or indirect signals, in either mode.

State must live in exactly one combined file in the target project's repository. The default is `DESIGN-QUESTIONS.md` at the project root. The user may explicitly override the filename or path, but do not create companion state files, databases, sidecar JSON, progress logs, or hidden caches. The optional template shipped with this skill is documentation only, not a second runtime state file.

If the state file is unreadable or structurally corrupted, report the problem and stop. Never silently recreate, overwrite, or repair it in a way that could lose history.

## Bootstrap Mode for an Already-Mature Project

The normal staged workflow (one group per run, never more than 7 new questions, never an inferred answer) assumes a project grows into its decisions over time, discovered a little at a time. That assumption breaks the first time this skill runs against a project that is already past `inception` — every later group's opening condition depends on the group before it reaching 75% locked-in, and on a first run every group is empty, so the normal rules would never let any group open at all. Bootstrap Mode exists to resolve that one-time deadlock, and to make sure the initial ledger for a mature project reflects what it actually does, not a generic template — it does not loosen the skill's discipline going forward.

**Trigger.** Bootstrap Mode applies to exactly one run: the run in which Phase 0 creates the state file for the first time (it did not exist before this run) AND Tier 0/Tier 1 conclude the project's stage is something other than `inception`. If the state file already exists, or the project is genuinely at `inception`, run the normal staged workflow instead — never enter Bootstrap Mode on a later run, even if many groups are still open or unanswered.

Bootstrap Mode runs as two passes in sequence: a **Stage-Identification Pass** to establish the stage, profile, and an initial question set, followed by a **Deep Evidence Pass** to ground that question set in the project's real implementation before presenting it.

**Stage-Identification Pass — scan budget.** Because this pass must gather enough evidence to identify the stage and draft an initial question set, widen the Phase 1 budget for this pass only: Tier 1 up to 15 files, Tier 2 up to 25 files, combined cap 40 files. Still prioritize design/spec/decision documents and central configuration over incidental source files, and still record exactly how many files were actually opened. This wider budget applies only to this pass of the triggering run; every subsequent run (Bootstrap Mode does not recur) uses the normal 8/10/18 budget from Phase 1.

**Question generation in the Stage-Identification Pass.** Generate the full set of design questions needed to cover all six fixed groups in this one run, ignoring the normal "one group per run" and "at most 7 questions" caps and ignoring each group's 75%-locked opening condition. Still apply every other Phase 2 generation rule unchanged: no duplicate or overlapping questions, immutable `DQ-<GROUPCODE>-NNN` IDs, a non-leading `why_we_are_asking` for every question, and a flat ledger. Set every generated question's status to `currently open` by default; the Deep Evidence Pass and the auto-answering rule below may then lock some in.

**Auto-answering.** For each question, check whether the evidence gathered so far (across both passes) contains an explicit, unambiguous statement of that decision in the project's own materials — either its documentation (a locked architectural decision recorded in a decision log, a design doc, or equivalent) or its actual implementation (a concrete function, configuration value, constant, or test that pins the behavior down) — never a guess drawn from code shape, naming, or absence of a counter-example. If such an explicit statement exists:

- Record it as the question's answer in Section D immediately, in this same run, without waiting for the user.
- Set the question's status to `locked in`.
- Cite the source inline in the Section D value: file name plus the specific decision, function, constant, or config key (e.g. "per `DECISION_LOG.md`: ..." or "per `score_ranking.py`, `Score = ...`, config key `simulation.max_loss_per_order`").
- Log a `status changed` (not `answer changed`) history entry in Section E noting the answer was recorded from existing project documentation or implementation during bootstrap, with its source.

If no such explicit statement exists — including when the evidence only suggests, implies, or makes a guess plausible — leave the question `currently open` and record nothing in Section D for it. When genuinely unsure whether a statement is explicit enough to count, treat it as not explicit and leave the question open; auto-answering must never convert a plausible guess into a recorded answer.

**Deep Evidence Pass — grounding questions and answers in the real implementation.** A mature project deserves an initial ledger grounded in what its codebase actually does, not a paraphrase of its documentation. After the Stage-Identification Pass has drafted the question set and recorded any doc-based answers, run a second, deeper pass whose only job is to sharpen both the questions and their answers against real evidence:

- This pass has no fixed file ceiling, but every file it opens must be opened for a specific reason: it is the concrete implementation, configuration, constant/threshold definition, or test that a specific already-drafted question is really about. Never open a file speculatively or "just to look around." Prioritize actual source modules, configuration files (e.g. `config.yaml`), threshold/constant definitions, and behavior-pinning tests over more prose documentation — the Stage-Identification Pass already covered the prose.
- For every question whose group and dependency scope points at a specific mechanism (a scoring formula, a state machine, a safety threshold, a rate limit, a retention rule, a dedupe key, and so on), locate that mechanism's real implementation and rewrite the question's `question` text to name the concrete artifact it is really about — the function, config key, threshold, or file — instead of leaving it as a generic templated phrasing. A sharp question names something that only exists in this project; a question that could be pasted unchanged into any other project is a defect this pass exists to fix. This rewriting applies to every question in all six groups, whether or not it ends up answered.
- For every answer recorded from documentation in the Stage-Identification Pass, cross-check it against the actual implementation. If the implementation is more specific than, or differs from, the documentation, replace the answer with the implementation's exact figures, thresholds, config keys, and edge-case handling, and cite the specific file (and function/constant name) alongside any prior doc source. If code and documentation conflict, record both values in the answer, state plainly which one governs current runtime behavior and why, and leave the conflict flagged for the user to resolve explicitly — never silently pick one without saying so.
- A number, threshold, or mechanism only belongs in Section D when it was actually read from a real file in this project during this pass (or the Stage-Identification Pass). If a concrete figure cannot be found anywhere in the scanned evidence, do not invent or estimate one — keep the question `currently open` (or keep its prior doc-sourced answer if the implementation search found nothing more specific, saying plainly that no more specific figure was found) rather than guess.
- This pass does not add new questions and does not open a seventh group; it only sharpens the wording of the six groups' already-drafted questions and deepens their answers with implementation evidence.
- This pass applies only for a project whose Tier 0/Tier 1 scan found a real, non-trivial implementation to ground answers in (i.e. it is meaningfully past `inception`). A project that is barely past inception, with little or no implementation yet, has nothing for this pass to read — skip it and rely on the Stage-Identification Pass and Phase 2's foundation-only fallback instead.

This mirrors how a careful engineer actually onboards onto an existing system: read what it currently does before asking what it should do next.

**End of Bootstrap Mode.** Once both passes complete, the project has an initial full ledger — questions grounded in concrete project artifacts, and whatever answers were explicitly on record in documentation or implementation. Every run after that — including runs that only add newly discovered questions, or only record a user's answer — follows the normal staged Phase 2 workflow (one group per run, at most 7 new questions, an answer recorded only when the user explicitly supplies it). Bootstrap Mode does not recur, and does not retroactively re-open or re-scan groups it already covered.

## Procedure

Phases always run in this order: Phase 0 → Phase 1 → Phase 2 → Phase 3. Phase 3 may also run as a standalone answer-recording operation when the user only wants to add or change an answer, but it must still pass through Phase 0 first. Do not skip a phase because a later action appears obvious.

### Phase 0 — Initialize or Locate the State File

1. Resolve the target project root and the state-file path. Use the explicit path supplied by the user; otherwise use `<project-root>/DESIGN-QUESTIONS.md`.
2. Check whether the file exists.
3. If it does not exist, create the empty five-section skeleton described in “Combined State File Contract” below. The skeleton must contain no invented project facts, questions, or answers. Note whether this creation makes the current run eligible for Bootstrap Mode (see above) once Phase 1 confirms the stage.
4. If it exists, read it and validate that all required sections are present and parseable. Load the existing ledger, every lifecycle status, the current answers, the scan snapshot/fingerprint, and the append-only history.
5. Validate the cross-section rules before doing anything else:
   - Every question ID is unique, including retired questions.
   - Every answer key exists in the question ledger.
   - Every dependency ID exists in the ledger.
   - The history section is separate and has not been rewritten as current state.
   - The status metadata names exactly one currently allowed group.

Definition of done: the state file is a valid, readable five-section file loaded into memory, or the skill has stopped with a clear corruption report. No later phase may run after a failed validation.

### Phase 1 — Tiered Project Scan

Run the scan in tiers and stop as soon as the relevant confidence threshold is met. Do not dump the repository into the model context. If this run is eligible for Bootstrap Mode (state file just created, stage turns out not to be `inception`), use the widened Stage-Identification Pass budget described above instead of the normal Tier 1/Tier 2 caps, and follow it with the Deep Evidence Pass once Phase 2 has drafted the question set.

#### Tier 0 — Compass / Project Card

This tier is cheap machine inspection. Given the project root:

- List the directory tree only to a bounded depth and list top-level names.
- Detect common manifests and configuration files, including `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, lockfiles, build files, and test configuration.
- Check whether Git is present; record the current HEAD when available, a short recent log, and a commit count.
- Count source files and test files using conservative extension/path rules.
- Produce a raw signal table: candidate stack, likely entry points, Git presence, rough size, test-to-code ratio, documentation signals, and whether the project is essentially empty.

Stop condition: the signal table is complete. If the project is near-empty—no meaningful source, configuration, or design material—conclude `inception` immediately, mark the stage confidence `confirmed`, and go directly to Phase 2 with only the foundation group (Bootstrap Mode never triggers for an `inception` project). Do not spend Tier 1/2 reads on an empty project.

#### Tier 1 — Stage Hotspot

Only when Tier 0 finds meaningful project material, read high-value text files up to the tier's budget (8 files normally, 15 files under the Bootstrap Mode Stage-Identification Pass). Prioritize the README, design/specification documents, decision logs, main manifests, central configuration, and obvious entry-point documentation. Apply a per-file line cap; summarize each selected file rather than passing its complete contents through context — except when a passage is the explicit evidence for an auto-answer, in which case keep enough of it to cite accurately.

Infer one roadmap stage: `discovery`, `design`, `build`, `test`, or `operate`. Record a confidence of `confirmed`, `inferred`, or `insufficient evidence`. Also record architectural decisions that are already locked in and ambiguities that could change the next question group.

Stop condition: stage confidence is at least `inferred`, or the Tier 1 file budget is exhausted. Escalate only if confidence remains insufficient and a specific ambiguity directly affects group selection, or — under the Bootstrap Mode Stage-Identification Pass — if more files are needed to find explicit evidence for likely auto-answers, up to the widened budget.

#### Tier 2 — Narrow Deep Dive

Investigate only the open ambiguities carried from Tier 1. For each ambiguity, inspect directly relevant files, prioritizing entry points, main routes, schemas, and integration boundaries; up to 3 files per ambiguity normally, up to 5 files per ambiguity under the Bootstrap Mode Stage-Identification Pass. The total Tier 2 budget is at most 10 files normally (25 under that pass), making the combined Tier 1+Tier 2 deep-read budget at most 18 files per scan normally (40 under that pass). Count files actually opened, not files merely listed. The Bootstrap Mode Deep Evidence Pass runs after this tier and has its own targeted, uncapped budget as described above — it is not part of this 18/40-file count.

Hard stop: when the Tier 1/Tier 2 budget is exhausted, accept the remaining uncertainty for stage identification. Do not keep reading in pursuit of full understanding at this tier; Phase 2 must compensate with a small foundational unlocking group in the normal case, or hand off remaining uncertainty to the Deep Evidence Pass under Bootstrap Mode.

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

### Phase 2 — Generate or Update Questions

Use Section A’s stage/confidence, Section B’s profile, and the complete current ledger and answer mapping. In the normal case questions are for the user to answer later; never answer them in this phase. Under Bootstrap Mode only, apply the auto-answering rule described above, and run the Deep Evidence Pass to sharpen every question's wording and ground its answer before presenting the ledger.

#### Fixed Question Groups

Use these groups in order. Each has an opening condition and a stage affinity — these conditions gate the normal staged workflow; Bootstrap Mode opens all six groups in its one triggering run regardless of these conditions.

1. `foundation` (`FND`, inception/discovery): product goal, primary user/problem, desired outcome, and hard constraints. Open at inception or whenever stage confidence is insufficient. This is the only group allowed for a near-empty project.
2. `scope-and-users` (`SCP`, discovery/design): target users, in/out of scope, core workflow, and priority boundaries. Open only when the foundation reaches at least 75% locked-in and its critical goal/constraint questions have answers.
3. `functional-requirements` (`FUN`, design/build): capabilities, business rules, states, edge cases, integrations, and acceptance boundaries. Open only when scope-and-users reaches at least 75% locked-in and its required dependencies are locked.
4. `data-and-boundaries` (`DAT`, build/test): entities, ownership/source of truth, validation, privacy/retention, migrations, and external data boundaries. Open only when functional-requirements reaches at least 75% locked-in and its data-relevant dependencies are locked.
5. `quality-and-operations` (`OPS`, test/operate): security/authentication concerns, performance, reliability, observability, testing depth, deployment, backup, and recovery decisions. Open only when data-and-boundaries reaches at least 75% locked-in and its required dependencies are locked.
6. `release-and-evolution` (`REL`, operate): rollout, migration/release strategy, acceptance of operational risk, feedback loop, and post-launch change boundaries. Open only when quality-and-operations reaches at least 75% locked-in and its required dependencies are locked.

The current stage limits the active group in the normal case: discovery emphasizes foundation/scope, design emphasizes scope/functional, build emphasizes functional/data, test emphasizes data/operations, and operate emphasizes operations/release. Never open a group earlier than its condition permits outside Bootstrap Mode. A low-confidence stage may open only 2–4 foundation questions that unlock stage identification; it may not open a full later group.

#### Generation Rules

- Normal case: open at most one new group per run, and add at most 7 new questions per run (2–4 for a low-confidence fallback). Bootstrap Mode: cover all six groups in the one triggering run; these caps do not apply to that run.
- If the current group is already open but incomplete, add questions only within that group. Do not “get ahead” by populating later groups — except inside the single Bootstrap Mode run, whose entire purpose is to populate every group at once.
- Before adding any question, compare it semantically against every Section C entry, including `no longer applicable` entries. If it repeats, overlaps, or merely rephrases an existing question, do not add it.
- Every question's `question` text must be concrete to this project, not a generic template. Name the specific artifact it is about — a file, function, config key, workflow step, or user-facing behavior that actually exists — whenever the evidence gathered supports doing so. A question phrased so generically that it could be pasted unchanged into an unrelated project is under-specified; sharpen it with whatever concrete detail the scan already found, and let Bootstrap Mode's Deep Evidence Pass sharpen it further where a deeper mechanism is involved.
- Assign a new immutable ID in the format `DQ-<GROUPCODE>-NNN`, for example `DQ-FND-001`. Choose the next unused number after checking all current and historical IDs. An ID is never reused, even after retirement.
- Each question must include: ID, group/stage, exact question text, a concise “why we’re asking” reason, dependency IDs, lifecycle status, and an optional review flag.
- The reason must explain which missing decision the answer affects. It must not contain answer hints, examples, recommendations, or leading phrasing.
- A question may be `currently open` only when every declared dependency is `locked in` — except during the Bootstrap Mode run, where every newly generated question starts `currently open` (or `locked in` immediately, only via an explicit auto-answer) regardless of the normal dependency gate, since the whole ledger is being established at once.
- Keep the ledger flat. Do not create nested question files or hidden lists.
- If a requirement has changed enough that an existing question is no longer the right question, mark the old entry `no longer applicable` and create a new unused ID. Do not edit history to make the old question disappear.

Definition of done (normal case): the allowed group is correct, no more than one group and seven questions were added, every new ID is globally unique, every dependency exists, every status obeys its dependency rule, and every new question has a non-leading reason with no answer suggestion. Present the newly opened questions plainly and stop; do not answer them.

Definition of done (Bootstrap Mode run): all six groups are represented in Section C with a non-redundant question set covering each group's scope; every question's wording names a concrete project artifact wherever the evidence supports it, rather than reading as a generic template; every new ID is globally unique; every question generated has a non-leading reason; and every Section D entry added during this run cites an explicit source — documentation, implementation, or both — in the project's own materials, with implementation-level specifics (figures, config keys, thresholds) wherever the Deep Evidence Pass found them. Present the full ledger plus which questions were auto-answered, from what source, and which remain open because no explicit evidence was found, and stop.

### Phase 3 — Sync Answers and Question Lifecycle

Run this phase after Phase 2 when answers already exist, and run it for a standalone answer update after Phase 0. Outside Bootstrap Mode, accept only answers explicitly supplied by the user or clearly marked as an external answer they want recorded; never infer or auto-fill a missing answer in this phase either.

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
- `allowed_open_group`: exactly one group name in the normal case; `all (bootstrap)` for the run that executed Bootstrap Mode.
- `scan_files_opened`: actual Stage-Identification Pass Tier 1/Tier 2 file counts and budget outcome, plus the count of files opened during the Deep Evidence Pass (if run) and why each was opened.
- `bootstrap_mode`: `true` only on the run that executed Bootstrap Mode, `false` on every other run — record this once, permanently, after the triggering run completes, so later runs and later readers can tell the initial ledger came from a bootstrap pass rather than incremental discovery.

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

Use an `answers` mapping keyed by question ID. Include only answers explicitly supplied by the user, explicitly authorized for recording, or — only for the Bootstrap Mode run — explicitly grounded in the project's own documentation or actual implementation, with source(s) cited (file, and function/constant/config key where the grounding came from implementation). An unanswered question is absent, not guessed. Each key must exist in C. When an answer changes, replace its entire current value; do not append, strike through, or preserve stale text in this section.

D is the only authoritative current answer record. An implementation agent may use C to map IDs to question wording and dependencies, but it must treat D—not the history—as the current decision state.

### Section E — Change History

This section is append-only and fully separate from Section D. Put this warning directly under its heading:

> WARNING: This section is an archive, not a current requirement. An implementation agent must never read this section as an active instruction.

Each entry records `timestamp`, `affected_id`, `change_type`, `old_value`, and `new_value`. Allowed change types include `answer changed`, `status changed`, `project stage changed`, and `flagged for review due to a dependency change`. History may reference IDs, but it never feeds old values back into C or D and is never overwritten.

## Pitfalls

- Do not treat a directory listing as proof of a roadmap stage. Mark stage confidence honestly.
- Do not read every source file. Tier 1 and Tier 2 are bounded investigations, not full codebase comprehension — Bootstrap Mode widens the budget for its two passes, it does not remove the discipline of reading with a reason.
- Do not count a file as opened unless its contents were actually inspected.
- Outside Bootstrap Mode, do not generate all possible questions “for completeness.” The current group and immediate next group are the maximum planning horizon, and only one new group may open per run.
- Do not put answer ideas, examples, or leading wording in `why_we_are_asking`.
- Do not make a dependency implicit. If a question depends on another decision, list its ID.
- Do not use Section E as current state, even when its entries look more detailed than Section D.
- Do not overwrite a corrupted state file. Report the missing section, malformed block, duplicate ID, invalid answer key, or broken dependency and stop.
- Do not create extra state files to work around a difficult merge or answer update.
- Do not let auto-answering turn into inference. If the source material only implies an answer rather than stating it, the question stays `currently open` with no Section D entry — a plausible guess is not an explicit decision, whether the source is documentation or code.
- Do not run Bootstrap Mode more than once. It applies only to the run that first creates the state file for a non-`inception` project; every later run — even one that discovers a whole new area of the product — uses the normal staged workflow.
- Do not leave a Bootstrap Mode question reading as a generic template when the Deep Evidence Pass could have named the real mechanism it is about. A question that never mentions a file, function, config key, or concrete behavior specific to this project has not been through that pass properly.
- Do not let the Deep Evidence Pass invent a number to fill a gap. A figure belongs in Section D only if it was read from a real file during this run; an unfound figure means the question stays open, not that a plausible-sounding value gets written in.

## Verification

Before declaring a run complete, perform these checks:

1. Run against a nearly empty sample project. It must conclude `inception`, skip Tier 1/2, stay within budget, and create only the foundation group.
2. Run against a small repository containing both backend and frontend code. Count the files actually opened and verify the combined Tier 1+Tier 2 deep-read count is no more than 18; assess whether the stage conclusion is reasonable.
3. Run again without changing that repository. Verify the fingerprint matches, Tier 1/2 are skipped, no duplicate questions are generated, and no ID is reused.
4. Change an answer that has a dependent question. Verify Section D contains only the replacement answer, the old answer is absent from D, Section E has the answer-change event, and the dependent question is reopened or retired correctly.
5. Validate structure: every D key exists in C, all IDs are unique, dependencies resolve, statuses are valid, and Section E grew rather than being overwritten.
6. Simulate an implementation agent reading the file. Section D must contain no stale answer values; Section E must be treated as archive only.
7. Run against a mature sample project (already past `inception`, with no prior state file) with a handful of explicitly documented decisions, a few decisions that only exist in code (not in any doc), and several undocumented/unimplemented items. Verify Bootstrap Mode triggers both passes, all six groups appear in Section C, doc-only and code-only decisions both land in Section D (each with a cited source naming where it came from and `locked in` status), every undocumented/unimplemented item stays `currently open`, at least some questions name a concrete file/function/config key rather than reading as generic template text, and `bootstrap_mode: true` is recorded in Section A. Then run again unchanged: verify Bootstrap Mode does not re-trigger and the normal staged workflow applies.
8. In that same mature-sample-project run, introduce a deliberate conflict between a documentation claim and the actual code (e.g. a doc says one threshold, the code enforces a different one). Verify the resulting Section D answer states both values, says which one governs current runtime behavior, and flags the conflict rather than silently picking one.

If any verification fails, report the exact failed invariant and do not claim the state is synchronized.
