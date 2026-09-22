---
name: spec-check
description: >-
  Review a project's SPECIFICATION.md for internal consistency, conciseness, and implementation blind spots, and report findings in chat.
  Use when the captain invokes /spec-check (e.g. "/spec-check <project>", "/spec-check" from a project directory) or asks to review or sanity-check a project specification before splitting it into crews.
  This is the pre-flight check before /create-crews: it judges whether the specification is ready to hand to that skill, including whether every package output is pinned exactly enough to write an exact reconciliation anchor.
  It only reads and reports; it never edits, rewrites, or suggests inline changes to SPECIFICATION.md or any other project file.
user-invocable: true
metadata:
  internal: true
---

# spec-check

Read a project's `SPECIFICATION.md`, judge whether it is internally consistent, concise, and complete enough for an agent to implement end to end, and report the findings in chat.
This is analysis only.
Never create, edit, move, or delete `SPECIFICATION.md` or any other project file while running this skill; there is no branch, no commit, no PR.
The human reads the chat feedback and makes any changes themselves.

## Invocation and lookup

- `/spec-check <project>` resolves `<project>` against `projects/` and reads `projects/<project>/SPECIFICATION.md`.
- Plain `/spec-check` with no argument reads `SPECIFICATION.md` from the current working directory.
- Resolve an omitted project the same way `/create-crews` does: an explicit referent, a clear follow-up, or one confident match against the registry and work under way.
  Ask one concise question when the project is ambiguous or no project plausibly matches.
- If the resolved `SPECIFICATION.md` is absent or empty, say so plainly in chat and stop; do not invent feedback about a specification that is not there.
- Reading the project's other files to check consistency and implementability is allowed; writing to any of them is not.

## Review procedure

Read the whole specification, and enough of the project's existing code and structure to judge implementability, before writing feedback.
Produce specific, actionable findings, each pointing at the concrete passage, section, or missing value rather than giving generic advice.
Cover at least these dimensions:

1. **Required sections.**
   The specification must contain a problem statement, a solution description, and a brief implementation plan.
   Report each one that is missing entirely, and each one that is present but too thin to act on (for example a solution description with no plan, or a plan that only restates the goal).

2. **Consistency.**
   Look for internal contradictions, conflicting requirements, terminology that drifts between sections, and a solution description that does not match its own implementation plan.

3. **Conciseness.**
   Flag redundancy, filler, vague language, and passages that restate an earlier point without adding anything a builder needs.
   Ambiguity that a builder could read two different ways is a conciseness finding even when the prose is short.

4. **End-to-end implementability.**
   Judge whether an agent could build the whole thing from the specification plus the repository alone, with no further questions.
   Flag undefined data contracts or interfaces, unspecified error and edge-case behavior, missing non-functional requirements (performance, security, scale, compatibility), and unstated assumptions or external dependencies.

5. **`/create-crews` readiness.**
   Judge whether the specification exposes clear independent seams a decomposition could split along: distinct services, layers, endpoints, data contracts, or deliverable surfaces.
   See [`../create-crews/SKILL.md`](../create-crews/SKILL.md) for what that skill does with a specification and what its reconciliation anchor requires.
   The sharpest blind spot here is precision: flag every output left as "roughly", a range, or "should", because that phrasing blocks writing an exact `Expect` value.
   Name the specific passage and what exact value it is missing.

## Closing verdict

End the chat feedback with one clear verdict:

- **Ready to hand to `/create-crews`** when the required sections are present and substantive, no consistency or implementability gaps were found, and outputs are pinned precisely enough for exact reconciliation anchors.
- Otherwise, a **prioritized list of the specific gaps to fix first**, each one pointing at the concrete missing value or ambiguous passage found during the review, not generic advice to "add more detail" or "be more specific".
