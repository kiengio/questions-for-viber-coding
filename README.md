# Questions for Vibe Coding

A Claude Code skill for people who have a product idea but do not know how to turn it into a concrete specification for an AI implementation agent.

The skill keeps the clarification process deliberately small and staged. It first inspects the target repository, identifies the current roadmap stage, and then opens only the next appropriate group of design questions. As the project progresses and answers are recorded, it unlocks later groups without flooding the user with every possible decision at the beginning.

## The problem this solves

When a nontechnical person asks an AI to build an app, the biggest failure is often not the AI’s coding ability. It is that important product decisions were never made explicit. A single giant questionnaire is also a poor solution: it overwhelms the product owner, asks questions that depend on decisions not yet made, and creates redundant answers.

This skill provides a controlled alternative:

1. Inspect the project cheaply and then selectively.
2. Determine whether the project is in inception, discovery, design, build, test, or operation.
3. Ask a small, non-overlapping set of questions for that stage.
4. Store the current answers and question lifecycle in one durable Markdown file.
5. Reopen dependent questions when a previous decision changes.

The result is a growing design record that another implementation agent can consume.

## What this skill does not do

This skill does not:

- Answer the design questions.
- Suggest answers, examples, or preferred solutions.
- Use leading wording to push the user toward a decision.
- Implement, refactor, or debug the target project.
- Create tickets, tasks, roadmaps, or planning files outside the combined state file.
- Split state across multiple files.
- Guess an answer that the user has not explicitly supplied.
- Open later question groups early “just in case.”
- Decide whether the user, another AI, or a team member answers a question.
- Delete history or reuse a question ID.

You remain responsible for making the product decisions. You may answer directly, discuss them with one AI, ask multiple AIs and synthesize their responses, or use any other method you prefer.

## Installation

Claude Code discovers a project skill when it is placed at:

```text
<target-project>/.claude/skills/questions-for-viber-coding/SKILL.md
```

To install it for one project:

1. Copy this repository’s `SKILL.md` into the path above.
2. Optionally copy `README.md` and `templates/` too if you want local documentation or a starting template. They are not required for Claude Code to load the skill.
3. Start Claude Code from the target project (or its repository root).
4. Confirm that `/questions-for-viber-coding` appears as an available skill, then invoke it.

To install it for all projects for one user, copy the same skill folder to:

```text
~/.claude/skills/questions-for-viber-coding/SKILL.md
```

On Windows, the equivalent user directory is commonly:

```text
%USERPROFILE%\.claude\skills\questions-for-viber-coding\SKILL.md
```

The exact Claude Code installation method may vary by version. The important contract is the directory layout: a skill folder containing a root `SKILL.md` under `.claude/skills/` or the user-level Claude skills directory. If you keep this repository outside the target project, use Claude Code’s supported extra-directory option or copy the skill into the project as described above.

The skill is marked for explicit invocation because it writes and updates the project’s design state. Claude should not run it unexpectedly during an unrelated coding task.

## How to invoke it

From the target project, type:

```text
/questions-for-viber-coding
```

A useful first message is:

```text
Scan this project and run the next allowed design-question phase. Use the default DESIGN-QUESTIONS.md at the project root. Do not implement code or answer the questions.
```

You may override the state-file name explicitly:

```text
/questions-for-viber-coding

Use `docs/product-decisions.md` as the one combined state file. Scan the project, then generate only the next allowed question group.
```

For an answer-only update, invoke the skill and be explicit that no rescan is wanted:

```text
/questions-for-viber-coding

Do not rescan. Record this user-provided answer for DQ-FND-001:
[the answer]
```

The skill must still validate the state file first. If the file is corrupt, it stops instead of replacing it.

## The one-file state model

By default, all runtime state is kept in the target project’s root file:

```text
DESIGN-QUESTIONS.md
```

The file has five sections:

- Section A, Status Metadata: current stage, confidence, scan timestamp, fingerprint, file counts, and the one group currently allowed to open.
- Section B, Project Profile Summary: a short current description of the project, stack, locked decisions, and remaining ambiguities. This section is replaced on a meaningful rescan.
- Section C, Current Question Ledger: a flat list of questions with stable IDs, group/stage, reason, dependencies, and lifecycle status. Retired questions remain here.
- Section D, Current Answers: the authoritative current answer mapping keyed by question ID. When an answer changes, the old value is replaced, not appended.
- Section E, Change History: an append-only archive of changes. It is not an active specification and must never be treated as one by an implementation agent.

The template at `templates/DESIGN-QUESTIONS.md` shows the expected shape. You can copy it into a target project, but normally the skill creates the skeleton itself when the state file is absent.

## Typical session walkthrough

The following example shows the intended rhythm. The exact question wording depends on the project and its current evidence; the skill must not provide answer suggestions.

### 1. Start with an empty or nearly empty project

Suppose a new project directory contains only a README and no meaningful source code. Invoke the skill:

```text
/questions-for-viber-coding

Scan the project and generate the next questions.
```

The skill performs the cheap compass scan, concludes that the project is in `inception`, and skips the deeper file-reading tiers. It creates `DESIGN-QUESTIONS.md` and opens only the `foundation` group, with a small number of questions such as:

```text
DQ-FND-001 — What outcome should this product achieve?
Why we are asking: This determines how later requirements and acceptance boundaries will be evaluated.

DQ-FND-002 — Who is the primary user or beneficiary?
Why we are asking: This determines whose workflow and constraints later questions must describe.
```

Those are questions, not recommendations. The user decides the answers outside the skill.

### 2. Record answers

After thinking through the decisions, the user supplies explicit answers:

```text
/questions-for-viber-coding

Record these answers without rescanning:
- DQ-FND-001: [user’s current decision]
- DQ-FND-002: [user’s current decision]
```

The skill validates the IDs, writes the current values into Section D, and appends answer-change events to Section E. It does not invent values for unanswered questions.

### 3. Run again as the project progresses

Later, the user asks an implementation agent to create an initial project structure or adds real code and documentation. The user invokes the skill again:

```text
/questions-for-viber-coding

Scan for meaningful changes and generate only the next allowed question group.
```

The skill compares the stored fingerprint with the current repository. If meaningful files changed, it performs the bounded hotspot scan and updates the profile. If the foundation group meets its opening threshold, the skill may open `scope-and-users`. It opens at most one new group in this run and adds no more than seven new questions.

On an unchanged repository, it skips the deeper rescan and does not create duplicate questions. Existing IDs remain stable.

### 4. Change an earlier answer

Suppose the user later changes the answer to `DQ-FND-001`, and `DQ-FUN-003` depends on it. The user says:

```text
/questions-for-viber-coding

Do not rescan. Replace the current answer for DQ-FND-001 with:
[user’s revised decision]
```

The skill:

1. Records the old value and new value in Section E.
2. Fully replaces the value for `DQ-FND-001` in Section D.
3. Finds every ledger question depending on `DQ-FND-001`.
4. Reopens any dependent question that was locked in, adding a review flag and a history entry.
5. Marks a dependent question no longer applicable if the new decision makes it meaningless; it never deletes the entry.

Section D contains only the revised current answer. The old answer appears only in the archive Section E.

### 5. Hand the result to an implementation agent

When enough questions are locked in for the next implementation slice, give the target repository and `DESIGN-QUESTIONS.md` to the separate implementation agent. The implementation agent should use:

- Section A to understand the current stage and allowed scope.
- Section C to understand each question’s wording, lifecycle, and dependencies.
- Section D as the only authoritative source for current answers.
- Section E only if historical context is intentionally requested; it is never an active requirement.

The implementation agent should not implement questions that are still `not yet opened` or `currently open`, and should not treat a historical value in Section E as current.

## Question group progression

The built-in progression is:

1. Foundation
2. Scope and users
3. Functional requirements
4. Data and boundaries
5. Quality and operations
6. Release and evolution

A group opens only after its explicit prerequisite is met, normally at least 75% of the previous group’s questions being locked in plus any critical dependency answers. A single run can open only one new group. Low-confidence scans can open only a 2–4 question foundation subset to resolve the uncertainty.

The project stage limits how far ahead the skill may look:

- Discovery: foundation and scope.
- Design: scope and functional requirements.
- Build: functional requirements and data.
- Test: data and quality/operations.
- Operate: quality/operations and release/evolution.

This is intentional. A user who is still deciding what to build should not receive deployment and observability questionnaires.

## Scan limits and caching

Scanning is tiered:

- Tier 0 lists a bounded tree, top-level files, manifests, Git signals, and source/test counts.
- Tier 1 reads at most 8 high-value text files with per-file line limits.
- Tier 2 investigates only specific unresolved ambiguities, at most 3 files per ambiguity and at most 10 files total.

The combined Tier 1+Tier 2 deep-read budget is therefore at most 18 files. The skill must record the number of files actually opened. A near-empty project goes directly from Tier 0 to the foundation group.

On later runs, the skill compares a stored fingerprint containing a bounded file-tree snapshot hash and Git HEAD. If nothing meaningful changed, it skips Tier 1 and Tier 2. If only a subset changed, it investigates only the affected hotspot. This keeps the workflow predictable and prevents repeated full-project scans.

## FAQ and troubleshooting

### The state file is corrupted. What should I do?

Do not ask the skill to recreate it over the original. Make a backup first, inspect the reported structural problem, and restore a known-good version from version control or a manual backup if one exists. The skill is required to stop rather than risk losing the append-only history. After you restore or manually repair the file, invoke the skill again and let it validate the result.

### Why is there only one state file?

The single-file contract prevents answers, statuses, and history from drifting apart. Section C is the ID-keyed ledger, Section D is the current answer map, Section A controls what may open, and Section E is the archive. Do not create a separate answers file, roadmap file, scan cache, or “temporary” question list.

### Why did the skill ask only a few questions?

Incremental generation is the main feature. It opens no more than one new group per run and caps new questions at seven. A project with insufficient stage evidence receives only a 2–4 question foundation subset. Answer those questions and run the skill again when the project or decisions progress.

### Why did a question reopen?

A locked question may depend on an answer that changed. The skill reopens dependent questions so the user can review them. It records the reason in Section C and Section E instead of silently assuming the old conclusion remains valid.

### Why was a similar question not added?

Before adding a question, the skill compares it against every ledger entry, including retired entries. This avoids asking the same decision in different words and prevents a retired question from being accidentally recreated under a new prompt.

### Can I rename a question ID?

No. IDs are immutable and never reused. If the question itself is no longer appropriate, retire it as `no longer applicable` and create a new question with a new unused ID.

### Can the skill answer a question for me?

No. Answering is deliberately outside scope. You can ask another AI or a human for advice, then explicitly provide the decision you want recorded.

### How should an implementation agent consume the file?

Treat Section D as the current decision contract. Use Section C to map IDs to their question text and to see whether a decision is locked in. Use Section A for stage context. Never use Section E as an active instruction or as a source for resurrecting old answers.

### I ran the skill twice and nothing changed. Is it broken?

Not necessarily. If the repository fingerprint is unchanged, the skill should skip the deep scan and avoid duplicate questions. Check Section A’s fingerprint and scan counters. If you expected a new group, verify that the previous group’s threshold and dependency conditions have actually been met.

## Contributing

Ideas and feedback are welcome in [GitHub Discussions](../../discussions). Bug reports (behavior
that contradicts `SKILL.md`) go in [Issues](../../issues). See `CONTRIBUTING.md` before opening a
pull request that changes behavior.

## Files in this repository

- `SKILL.md` — the Claude Code workflow and non-negotiable rules.
- `README.md` — this beginner-friendly guide.
- `templates/DESIGN-QUESTIONS.md` — an optional empty state-file skeleton.
- `CONTRIBUTING.md` — how to share ideas, report bugs, or propose changes.
- `LICENSE` — MIT license.
