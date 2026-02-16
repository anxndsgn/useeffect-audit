# useEffect Audit Checklist

Use this checklist to classify each `useEffect` or `useLayoutEffect` and choose the refactor pattern.

## 1) Which trigger actually caused this logic?

Pick exactly one:
- `Interaction-driven`: caused by a user action (click, submit, input).
- `Rendering-driven sync`: caused by component appearing/updating and must sync with an external system.
- `Render data`: pure computation from props/state for UI output.

Decision:
- Interaction-driven -> move to the event handler.
- Render data -> derive during render.
- Rendering-driven sync -> keep an Effect and continue below.

## 2) Keep only true synchronization Effects

Keep an Effect only if it synchronizes with something outside React.

Examples:
- Subscribing/unsubscribing to external events or sockets
- Connecting/disconnecting to services
- Running imperative DOM or third-party widget APIs
- Starting/stopping timers
- Synchronizing network activity based on visibility

If not external sync, remove the Effect.

## 3) Validate Effect lifecycle semantics (for kept Effects)

Model each Effect as one independent start/stop process:
- Setup starts synchronization.
- Cleanup stops synchronization and fully undoes setup.
- Setup -> cleanup -> setup must be safe in development.
- Split unrelated sync processes into separate Effects.

Smell:
- One Effect mixes connection setup, logging, and derived state updates.

Fix:
- Keep sync in the Effect; move event logic out; derive UI data in render.

## 4) Fix dependencies by changing code, not by suppressing lint

Rule:
- Dependency arrays must include every reactive value read by Effect code.
- Do not suppress `react-hooks/exhaustive-deps`.

When dependencies feel "too reactive", refactor:
- Move interaction-specific logic into event handlers.
- Move non-reactive constants outside the component.
- Create objects/functions inside the Effect (or extract primitive deps).
- Use state updater functions when reading state only to compute next state.
- Split Effects by purpose to reduce accidental dependencies.

## 5) Use `useEffectEvent` only for non-reactive Effect logic

Use `useEffectEvent` when Effect code must read latest props/state without making the Effect re-synchronize for those reads.

Allowed pattern:
- Keep synchronization reactive to true sync keys.
- Move non-reactive reads (for example analytics/logging payload details) into an Effect Event called from the Effect.

Constraints:
- Do not use Effect Events to hide legitimate dependencies.
- Do not pass Effect Events to other components or hooks.
- Do not add Effect Events to dependency arrays.

## 6) Common remove/refactor patterns

### A) Derived data in Effect

Smell:
- Effect sets state like `setFullName(firstName + " " + lastName)`.

Fix:
- Replace mirrored state with render-time expressions.

Before:
```tsx
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(firstName + ' ' + lastName);
}, [firstName, lastName]);
```

After:
```tsx
const fullName = firstName + ' ' + lastName;
```

### B) Event logic in Effect

Smell:
- Effect runs after `submitted === true` to send a request or show a notification.

Fix:
- Execute directly in click/submit handler.

Before:
```tsx
const [submitted, setSubmitted] = useState(false);
useEffect(() => {
  if (submitted) {
    post('/api/checkout');
    showToast('Order placed');
  }
}, [submitted]);
```

After:
```tsx
async function handleSubmit() {
  await post('/api/checkout');
  showToast('Order placed');
}
```

### C) Reset-all state on identity change

Smell:
- Effect clears local state when `userId` or `itemId` changes.

Fix:
- Prefer keyed remount (`key={identity}`) for full subtree reset.

### D) Partial sync on prop change

Smell:
- Effect runs on `items` change and calls `setSelection(null)`.

Fix:
- Store stable ids; derive selected object in render.
- If needed, guard updates during render with previous-value tracking.

### E) Chain of Effects

Smell:
- Effect A sets state; effect B reacts and sets more state; effect C emits callback.

Fix:
- Collapse into render-time derivation and event-time batching.

### F) Child notifying parent via Effect

Smell:
- Child Effect calls `onChange(value)` when local state updates.

Fix:
- Call `onChange(nextValue)` in the same child event handler.
- Or make child controlled by parent.

## 7) Verification checklist

For each kept/refactored Effect, verify:
- No behavior change for user-visible flows.
- No stale reads (especially after dependency changes).
- Cleanup runs correctly on dependency change and unmount.
- Development setup/cleanup replay is safe.
- No `exhaustive-deps` suppressions were added.

## Mini snippet: keep sync reactive, move non-reactive reads to `useEffectEvent`

Before:
```tsx
useEffect(() => {
  const conn = createConnection(serverUrl, roomId);
  conn.on('connected', () => {
    showNotification('Connected!', theme);
  });
  conn.connect();
  return () => conn.disconnect();
}, [roomId, theme]);
```

After:
```tsx
const onConnected = useEffectEvent(() => {
  showNotification('Connected!', theme);
});

useEffect(() => {
  const conn = createConnection(serverUrl, roomId);
  conn.on('connected', onConnected);
  conn.connect();
  return () => conn.disconnect();
}, [roomId]);
```

## Refactor Output Template

Use this per Effect:
1. File + component
2. Original Effect intent
3. Classification: Keep/Remove
4. Trigger type: Rendering-driven sync or Interaction-driven event
5. Chosen pattern from this checklist
6. Dependency rationale
7. Risk notes
8. Verification steps
