---
tags: [frontend/rendering]
status: growing
---

# Virtual DOM

> An abstraction that simplifies *modifying* the UI — you describe the target state, the library figures out the DOM operations. It is not a performance feature.

VDOM is a thing which is used by react, for example. it will re-render staff without affecting the real dom in the browser.

It's not about performance. A VDOM is an abstraction to simplify the act of modifying a UI.
You describe how you would like your page to look like and the library takes care of taking the DOM from its current state, to the state you want it to be in.

More detailed code examples are here [A virtual DOM in 200 lines of JavaScript](https://lazamar.github.io/virtual-dom/)

## Sources

- [A virtual DOM in 200 lines of JavaScript](https://lazamar.github.io/virtual-dom/)
