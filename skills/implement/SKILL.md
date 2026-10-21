---
name: implement
description: "Implement a feature from a feature document created by the newfeature skill."
argument-hint: "[feature-document-path]"
disable-model-invocation: true
---

Implement the work described in the feature document at `$0`. If no path was given, ask for it.

The feature document is the spec. Read the whole file before starting, and follow the rules in its **Generation** section.

Before changing anything, record the current `HEAD` commit (`git rev-parse HEAD`) as the base commit for the review.

Call the Skill tool with "tdd" where possible, at the seams listed in the **Seams** section of the feature document. Those seams are pre-agreed.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Once done, call the Skill tool with "feature-review", passing the base commit and the feature document path.

Never commit. Leave all changes uncommitted for the user to review.
