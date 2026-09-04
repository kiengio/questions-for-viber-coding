# Contributing

Thanks for considering a contribution to `questions-for-viber-coding`.

## Sharing an idea

Ideas, feedback, and "what if it also did X" suggestions are welcome in
[GitHub Discussions](../../discussions) — use the **Ideas** category. Discussions is the right
place for anything exploratory or not yet fully formed. You don't need to write a formal proposal;
a short description of the problem you hit or the improvement you'd like is enough.

## Reporting a bug

If the skill behaves in a way that contradicts `SKILL.md` (for example: it opened more than one
question group in a single run, reused a retired ID, answered a question itself, or read far more
files than the stated tier budgets), please open an [Issue](../../issues) describing:

- What you asked the skill to do.
- What happened, ideally with the relevant `DESIGN-QUESTIONS.md` excerpt.
- What you expected instead, with a reference to the specific rule in `SKILL.md` it violated.

## Proposing a change (Pull Requests)

Before opening a PR that changes behavior (not just typo/doc fixes), please open a Discussion or
Issue first so the change can be discussed. This skill's design went through several rounds of
review to keep its scope narrow and its state model predictable, so behavioral changes should be
weighed against that intent rather than added ad hoc.

When you do open a PR:

1. Keep `SKILL.md` internally consistent — if you change a rule, check the "Procedure",
   "Combined State File Contract", "Pitfalls", and "Verification" sections for anything that now
   contradicts it.
2. If you change the state-file contract (the five sections in `SKILL.md` / `templates/DESIGN-QUESTIONS.md`),
   update both files together, plus the matching explanation in `README.md`.
3. Do not expand scope silently. This skill intentionally does not answer questions, does not
   implement code, and does not manage a multi-file state. Proposals to add those belong in a
   Discussion first, not directly in a PR.
4. Describe how you tested your change (e.g., ran it against an empty project and a project with
   existing code, as described in `SKILL.md`'s "Verification" section).

## Code of conduct

Be respectful and assume good faith. Disagreements about design should focus on the stated scope
and goals of the skill, not on the people discussing it.
