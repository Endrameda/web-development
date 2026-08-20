---
tags: [frontend/state, frontend/ux]
status: growing
---

# Optimistic UI

> Update the UI the moment the user acts, assume the server will agree, and roll back if it does not.

Optimistic UI is a design and development approach(pattern) where changes made by a user are immediately reflected on the user interface (UI), even before those changes are confirmed by the server. this technique creates a faster and more seamless user experience by assuming the server-side operation will succeeded and updating the UI accordingly.

### How It works?
When user performs an action (e.g. adding an item to cart or liking a post), the UI updates immediately, giving the impression of instant responsiveness.

While the UI is updated optimistically, a request is sent to the server to confirm the action.

If the server confirms the action, no further UI changes are needed. If the server fails the request (e.g. due to an error or invalid input), the UI is reverted to its previous state.

### Benefits and challenges of Optimistic UI
1. **Enhanced User Experience**. Immediate feedback makes the application feel faster and more responsive, improving user satisfaction.
2. **Reduced** [**Perceived Latency**](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Performance/Perceived_performance)**.** Users don't have to wait for server responses before seeing updates, masking potential delays.  
3. **Increased Engagement.** Quick, seamless interactions encourage users to continue engaging with the application.

As for challenges:
1. **Error Handling**. Developers must implement robust mechanisms to handle server failures and gracefully revert UI changes if needed.
2. **State Management Complexity.** Managing intermediate state (e.g. pending, succeeded, failed) can add complexity to the codebase.
3. **Data Integrity.** Ensuring consistency between the UI and server-side data requires careful design.
### Examples of Optimistic UI
The most knowns are **Social Media Likes.** When a user likes a post, the like count and icon change instantly, even before the server confirms the action.

In case you are using a **To-Do List App** you've probably noticed that the change will appear when you add or delete a task

In the eCommerce world, **Cart Updates** are an obvious use case. When a user adds an item to their cart, the cart count updates instantly without waiting for server confirmation.

## Sources

- [MDN — Perceived performance](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Performance/Perceived_performance)
