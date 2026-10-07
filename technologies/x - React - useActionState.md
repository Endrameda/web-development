---
tags:
  - react/hooks
status: seed
---

# React - useActionState

> A React hook that lets a form action update state directly from the result of an async submission, including its pending status - without manually wiring `useState` + `useTransition` yourself.

## When to reach for it

Reach for it the moment a `<form>`'s submission needs to drive state off an async result — a pending flag while it's in flight, a validation error if it fails, the new value if it succeeds — and you'd otherwise be hand-wiring `useState` + `useTransition` + your own try/catch to get the same three things. It's the natural fit for a Server Action in a Next.js/RSC app, where the `permalink` option even lets the form work before JavaScript has hydrated.

Skip it for async work with no form semantics at all — a one-off fetch triggered by a button that isn't submitting anything. Plain `useState` and an async handler are simpler there; `useActionState` earns its keep specifically around `<form action={...}>`.

## How

**Signature:**

```js
const [state, formAction, isPending] = useActionState(action, initialState, permalink?)
```

- **`action(previousState, payload)`** — your function. `previousState` is always first: on the first call it's `initialState`, on every call after that it's whatever `action` last returned. `payload` is whatever you pass to `formAction` yourself, or the form's `FormData` when `formAction` is wired directly to `<form action={formAction}>`.
- **`initialState`** — only used for the very first render; ignored after the first dispatch.
- **`permalink`** *(optional)* — a URL for progressive enhancement with Server Components. If the form submits before JS has loaded, the browser navigates there instead of running the action client-side. The same component (same `action` and `permalink`) has to render on that destination page too, or state can't pass through.

**Basic example:**

```jsx
async function updateCartAction(prevState, formData) {
  const quantity = Number(formData.get('quantity'))
  const result = await addToCart(quantity)

  if (result.error) {
    return { ...prevState, error: result.error }
  }
  return { count: result.totalItems, error: null }
}

function Checkout() {
  const [state, formAction, isPending] = useActionState(updateCartAction, {
    count: 0,
    error: null,
  })

  return (
    <form action={formAction}>
      <input type="number" name="quantity" defaultValue="1" />
      <button disabled={isPending}>Add to cart {isPending && '…'}</button>
      {state.error && <p className="error">{state.error}</p>}
      <p>Items in cart: {state.count}</p>
    </form>
  )
}
```

**`isPending`** flips to `true` automatically for the whole time any dispatched action is in flight — no manual `useTransition` required, it's already wired in.

**Error handling — two deliberate patterns:**

1. **Return it as state** for anything recoverable (validation failures, a rejected quantity) — the form stays interactive and you render the error from `state`, exactly like the example above.
2. **Throw** only for truly unexpected failures. A thrown error cancels every currently *queued* action (not just the one that threw) and propagates to the nearest Error Boundary — it's a hard stop, not a form error message.

**Sequential queuing.** Dispatching multiple times in quick succession doesn't run them in parallel — each call waits for the previous one to finish and receives its return value as `previousState`. Four rapid clicks on a 1-second action take four seconds total, not one.

## Gotchas

- **The action's argument order flips.** A plain form action is `action(formData)`. Under `useActionState` it becomes `action(previousState, formData)` — porting an existing action over without noticing this is the single most common mistake.
- **`formAction` must run inside a transition.** Wiring it to `<form action={formAction}>` or `<button formAction={formAction}>` handles this automatically; calling it directly from a plain `onClick` handler throws, because it isn't wrapped in `startTransition`.
- **Never call the dispatch function during render.** Only from an event handler or the form's own action wiring — calling it directly in the component body is the classic "update during render" infinite loop.
- **There's no built-in reset.** If you need one, encode a sentinel into your own reducer logic (e.g. dispatching `null` means "go back to `initialState`") — the hook doesn't give you this for free.
- **Multiple ongoing actions are currently batched together.** Documented as a present limitation of the hook, not a bug you're triggering by accident — worth knowing before spending time trying to make two independent in-flight actions behave independently.

## Sources

- [React Docs — useActionState](https://react.dev/reference/react/useActionState)
