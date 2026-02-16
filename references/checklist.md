# useEffect Audit Checklist

Use this checklist to classify each effect and choose a refactor pattern.

## 1) Is this synchronizing with an external system?

Keep the effect if the answer is yes.

Examples:
- Subscribing/unsubscribing to external events or sockets
- Running imperative DOM or third-party widget APIs
- Starting/stopping timers
- Synchronizing network activity based on component visibility

If kept:
- Ensure dependency list matches used values.
- Ensure cleanup is correct.

## 2) Is this deriving render data from props/state?

Remove the effect and derive during render.

Smell:
- Effect sets state like `setFullName(firstName + " " + lastName)`.

Fix:
- Replace mirrored state with render-time expressions.

## 3) Is this caused by a specific user action?

Move logic into the event handler.

Smell:
- Effect runs after `submitted === true` to send a request or show a notification.

Fix:
- Execute side effect directly in click/submit handler where the intent is explicit.

## 4) Is this resetting all local state when identity changes?

Prefer keyed remount instead of effect-based reset.

Smell:
- Effect clears local state when `userId`/`itemId` changes.

Fix:
- Split component and pass `key={identity}` to reset subtree state automatically.

## 5) Is this adjusting only part of local state on prop change?

Prefer deriving from stable ids or render-time guards, not effect-based sync.

Smell:
- Effect runs on `items` change and calls `setSelection(null)`.

Fix:
- Store selected id; derive selected object in render.
- If needed, guard updates during render with previous-value tracking pattern.

## 6) Is this part of a chain of effects?

Collapse chain into render-time derivation plus event-time updates.

Smell:
- Effect A sets state; effect B reacts and sets more state; effect C emits parent callback.

Fix:
- Compute derivable values in render.
- Batch state updates in the original event handler.

## 7) Is this notifying parent about local state changes?

Prefer parent updates in the same event that changes child state.

Smell:
- Child effect calls `onChange(value)` whenever local `value` updates.

Fix:
- Call `onChange(nextValue)` inside child event handler.
- Or fully control child from parent when appropriate.

## Refactor Output Template

Use this per effect:

1. File + component
2. Original effect intent
3. Keep/Remove decision
4. Chosen pattern from this checklist
5. Risk notes
6. Verification steps
