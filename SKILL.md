---
name: useeffect-audit
description: Audit and refactor React components to remove unnecessary useEffect usage and fix Effect-related bugs such as derived-state syncing, event logic in Effects, chains of Effects, and parent-child notification loops. Use when reviewing React code with useEffect, debugging rerender loops, or modernizing components according to React's "You Might Not Need an Effect" guidance.
---

# useEffect Audit

## Overview

Run a structured review of each `useEffect` and decide whether to remove it, move logic to render or event handlers, or keep it for external synchronization only. Apply minimal, behavior-preserving refactors with explicit reasoning and test notes.

Primary reference: https://react.dev/learn/you-might-not-need-an-effect

## Audit Workflow

1. Find all effects in scope:
	- Scan for `useEffect(` and `useLayoutEffect(`.
	- Group by component and purpose.
2. Classify each effect using `references/checklist.md`:
	- Keep only if it synchronizes with an external system.
	- Otherwise move logic to render-time computation, keyed reset, or event handlers.
3. Refactor one effect at a time:
	- Preserve behavior.
	- Remove redundant state and redundant renders.
	- Keep dependencies and cleanup correct for effects that remain.
4. Report changes:
	- State the original smell.
	- State the chosen pattern and why.
	- List behavior risks and test cases.

## Decision Rules

- Keep an effect for external synchronization only:
	- Browser APIs, subscriptions, timers, imperative DOM/third-party widgets, or network sync caused by visibility.
- Remove an effect when it only transforms data for rendering:
	- Derive values directly during render.
- Remove an effect when it only reacts to a user action:
	- Move logic into the event handler that caused it.
- Replace prop-change reset effects with keyed remount when possible.
- Avoid chains of effects that only update state to trigger more effects.
- Prefer a single source of truth; avoid mirrored state.

## Output Format

For each audited effect, produce:

1. `file:line` and component name
2. Current effect intent (one sentence)
3. Classification: `Keep` or `Remove`
4. Refactor pattern applied (from `references/checklist.md`)
5. Code-level change summary
6. Verification notes (what to test)

## Guardrails

- Do not move logic out of effects if it changes semantics (for example, true external sync).
- Do not silence `exhaustive-deps` without explicit justification.
- Do not add `useMemo`/`useCallback` unless necessary for correctness or measured performance.
- Prefer the smallest safe refactor and verify with targeted tests.
