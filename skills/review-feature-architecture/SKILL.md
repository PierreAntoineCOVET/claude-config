---
name: review-feature-architecture
description: Review the Technical section of a feature document (created by the newfeature skill) for deepening opportunities before any code is written, present them as a visual HTML report, then grill through whichever one you pick. Only invoke when the user explicitly asks for an architecture review of a feature document, or gives their go for the architecture review phase of newfeature.
argument-hint: "[feature-document-path]"
---

# Review Feature Architecture

Review the design proposed in a feature document, not the existing codebase. Surface architectural friction in the proposal and suggest **deepening opportunities**: changes that turn shallow modules into deep ones before they are built. The aim is testability and AI-navigability.

The feature document is at `$0`. If no path was given, ask for it.

This skill is _informed_ by the project's domain model and built on a shared design vocabulary:

- Call the Skill tool with "codebase-design" for the architecture vocabulary (**module**, **interface**, **depth**, **seam**, **adapter**, **leverage**, **locality**) and its principles (the deletion test, "the interface is the test surface", "one adapter = hypothetical seam, two = real"). Use these terms exactly in every suggestion, and don't drift into "component," "service," "API," or "boundary."
- The domain language in `GLOSSARY.md` gives names to good seams; ADRs in `docs/adr/` record decisions this skill should not re-litigate.

## Process

### 1. Explore

Read the whole feature document. The **Functional** section is settled: don't question it, use it to judge whether the design serves it. The review targets the **Technical** section: Sequence diagram, File structure, Seams, Assumptions and Tests.

Read the project's domain glossary (`GLOSSARY.md`) and any ADRs in the area the feature touches.

Then spawn a sub-agent to read only the existing code the feature touches: the updated and removed files in **File structure**, and the unchanged classes in the **Sequence diagram**. Don't scan the rest of the codebase. Note where the proposal creates friction:

- Where would understanding one concept require bouncing between many small new modules?
- Which proposed modules are **shallow**, with an interface nearly as complex as the implementation?
- Where does the proposal extract pure functions just for testability, while the real bugs would hide in how they're called (no **locality**)?
- Where would new modules leak across their seams, or couple tightly to existing ones?
- Are the seams in **Seams** placed where behavior can be observed without reaching inside? Are there more seams than needed?
- Does the proposal fit how the existing code is shaped, or fight it?

Apply the **deletion test** to every proposed module you suspect is shallow: would removing it from the design concentrate complexity, or just move it? A "yes, concentrates" is the signal you want.

### 2. Present candidates as an HTML report

Write a self-contained HTML file to the OS temp directory so nothing lands in the repo. Resolve the temp dir from `$TMPDIR`, falling back to `/tmp` (or `%TEMP%` on Windows), and write to `<tmpdir>/feature-architecture-review-<timestamp>.html` so each run gets a fresh file. Open it for the user (`xdg-open <path>` on Linux, `open <path>` on macOS, `start <path>` on Windows) and tell them the absolute path.

The report uses **Tailwind via CDN** for layout and styling, and **Mermaid via CDN** for diagrams where a graph/flow/sequence reliably communicates the structure. Mix Mermaid with hand-crafted CSS/SVG visuals. Each candidate gets a **before/after visualisation**: before is the design as proposed in the document, after is the suggested change. Be visual.

For each candidate, render a card with:

- **Files**: which proposed and existing files/modules are involved
- **Problem**: why the proposed design would cause friction
- **Solution**: plain English description of what would change in the design
- **Benefits**: explained in terms of locality and leverage, and how tests would improve
- **Before / After diagram**: side-by-side, custom-drawn, illustrating the shallowness and the deepening
- **Recommendation strength**: one of `Strong`, `Worth exploring`, `Speculative`, rendered as a badge

End the report with a **Top recommendation** section: which candidate you'd tackle first and why. If the proposal is already sound, say so: an empty or short report is a valid outcome.

**Use GLOSSARY.md vocabulary for the domain, and the `/codebase-design` vocabulary for the architecture.**

**ADR conflicts**: if a candidate contradicts an existing ADR, only surface it when the friction is real enough to warrant revisiting the ADR. Mark it clearly in the card.

See [HTML-REPORT.md](HTML-REPORT.md) for the full HTML scaffold, diagram patterns, and styling guidance.

Do NOT propose interfaces yet. After the file is written, ask the user: "Which of these would you like to explore?"

### 3. Grilling loop

Once the user picks a candidate, call the Skill tool with "grilling" to walk the decision tree with them: constraints, dependencies, the shape of the deepened module, what sits behind the seam, what tests survive.

To explore alternative interfaces for the deepened module, call the Skill tool with "codebase-design" and use its design-it-twice parallel sub-agent pattern.

### 4. Update the feature document

When a decision settles, propose the exact edits to the **Technical** section (Sequence diagram, File structure, Seams, Assumptions, Tests) and write them only after the user approves. Keep the document's diagram rules. Never edit the **Functional** section; if a decision would change it, stop and tell the user.

Call the Skill tool with "domain-modeling" for new terms and ADRs: propose each `GLOSSARY.md` term or ADR, and write it only after the user approves. If the user rejects a candidate with a load-bearing reason, offer an ADR so future reviews don't re-suggest it.
