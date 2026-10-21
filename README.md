# claude-config

My own versions of Claude Code skills, copied from [mattpocock/skills](https://github.com/mattpocock/skills).

## Skills

`codebase-design`, `diagnosing-bugs`, `domain-modeling`, `feature-review` (upstream `code-review`), `grill-me`, `grill-with-docs`, `grilling`, `implement`, `improve-codebase-architecture`, `newfeature` (mine), `prototype`, `review-feature-architecture` (mine), `tdd`, `wizard`

Installed as symlinks from `~/.claude/skills/<name>` to `skills/<name>`, so edits here apply immediately.

## TODO

- [x] **Remove the commit step from `implement`**: `skills/implement/SKILL.md` still ends with "Commit your work to the current branch".
- [ ] **Initialize the git repository**: `git init` has not been run yet.
- [x] **Install the skills**: Claude Code won't see them until they are linked or copied into `~/.claude/skills/`.
- [x] **`setup-matt-pocock-skills` is not included**: `code-review` expects the issue tracker config it creates, so without it, it will ask where the spec is or tell you to run setup.
- [x] **Resolve the `code-review` name clash**: it keeps its upstream name, which clashes with the built-in `/code-review` skill once installed.

## Plan

Target workflow:

1. `/newfeature` creates the feature document
2. `grilling` explores the idea as far as possible
3. Write the **Functional** part of the document
4. `domain-modeling` checks the Functional part is clear and consistent with the glossary and ADRs
5. Draft the **Technical** part of the document
6. Architecture review of the Technical part (not the codebase)
7. `/implement` uses the document as spec, with `tdd`
8. `feature-review` reviews the uncommitted changes against the document

### Decisions

- [x] **Code review**: the repo's `code-review`, renamed `feature-review` to avoid the clash with the built-in `/code-review`.
- [x] **`newfeature`**: becomes a skill (`skills/newfeature/SKILL.md`). Commands are merged into skills and support the same `$0`/`$1` arguments; a skill can also hold the Mermaid diagram rules in a separate file loaded on demand. Keep `disable-model-invocation: true` so it only runs when typed.
- [x] **Architecture review**: a new skill; `improve-codebase-architecture` stays as-is.
- [x] **Seams**: a new `### Seams` subsection under Technical, before Tests.
- [x] **Chaining**: `newfeature` calls each phase's skill itself, asking before each phase. It calls `grilling` directly (`grill-me` is user-only), and `review-feature-architecture` is model-invocable so it can launch it. `implement` stays manual (`disable-model-invocation: true`): at the end, `newfeature` tells you to run `/implement <path>`.
- [x] **Glossary and ADRs**: during feature mode, each `GLOSSARY.md` term or ADR is proposed and written only after approval.
- [x] **Architecture review skill name**: `review-feature-architecture`.

### 0. Groundwork

- [x] 1. Convert `~/.claude/commands/newfeature.md` into `skills/newfeature/SKILL.md` in this repo, then remove the old command (a skill takes precedence over a command with the same name anyway).
- [x] 2. Resolve the `code-review` name clash: renamed `feature-review`.
- [x] 3. Install by symlinking each skill folder into `~/.claude/skills/` so edits here apply immediately.

### 1. `newfeature`: the orchestrator

- [x] 4. Remove its own implementation step; the final step hands off to `/implement <path>` instead.
- [x] 5. Allow `domain-modeling` to write `GLOSSARY.md` and ADRs during feature mode (or require approval for each write); feature mode currently only allows editing the feature file.
- [x] 6. Make the phases explicit (grill, write Functional, domain review, write Technical, architecture review, confirm, `/implement`), each naming the skill it calls and what it writes back to the document.
- [x] 7. Add a seams entry to the template ("Seams" under Technical, or redefine **Tests** as "seams + tests"), since `tdd` won't write tests until seams are agreed.

### 2. Grilling

- [x] 8. `grilling` already works on an idea and waits for confirmation before acting; no change needed.
- [x] 9. Use `grill-me` for the idea phase, not `grill-with-docs` (which runs `domain-modeling` at the same time). Optionally tell `grilling` that its settled decisions feed the Functional section.

### 3. `domain-modeling`

- [x] 10. Add a "review a feature document" mode for the Functional part: check each term against `GLOSSARY.md` and ADRs, invent edge-case scenarios for every business rule, compare the described flows with the code, report contradictions, then update the glossary or offer ADRs.

### 4. Technical draft

- [x] 11. Have `newfeature` load `codebase-design` while drafting Technical, so the architecture review speaks the same vocabulary. No change to `codebase-design` itself.

### 5. Architecture review of the document

- [x] 12. Create a new skill (rather than changing `improve-codebase-architecture`, whose Explore step scans code and git history) that:
  - takes the feature document path
  - reads the Technical section and only the existing code it touches
  - applies the `codebase-design` principles (deletion test, shallow vs deep modules, seam placement)
  - produces the same kind of HTML report, with candidates about the proposed design (reusing `HTML-REPORT.md`)
  - runs `grilling` on the picked candidate
  - writes accepted decisions back into the Technical section

### 6. `implement`

- [x] 13. Remove the commit line; replace it with "leave all changes uncommitted".
- [x] 14. Take the feature document path as argument, use it as the spec and follow its Generation section.
- [x] 15. Pass the document's seams to `tdd` as already agreed; add one line to `tdd`: "seams listed in the spec count as confirmed".
- [x] 16. Pass the base commit (HEAD before implementation) and the document path to the code review.

### 7. Code review

- [x] 17. If using the repo's `code-review`: diff the working tree against the base commit (`git diff <base>` plus untracked files) instead of `git diff <fixed-point>...HEAD`, which is empty when nothing is committed.
- [x] 18. Remove its issue tracker dependency (the `/setup-matt-pocock-skills` line) and use the feature document as the spec source.
- [x] 19. ~~If using the built-in `/code-review` instead, nothing changes; it already reviews uncommitted changes, but there is no check against the spec.~~ Not applicable: the repo's review was chosen.
