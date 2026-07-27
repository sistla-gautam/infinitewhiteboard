Type: grilling
Status: resolved

## Question

What exact interaction model should the canvas creation screen expose for the three starting states: new canvas, template, and AI-generated canvas?

## Answer

**Creation surface**

- A single unified surface (modal or dropdown — implementation's choice) presents all three options together. No two-step "pick a mode, then go to a mode-specific screen" flow.
- **Blank**: a plain click target. Clicking creates the canvas immediately and opens it.
- **Template**: a visual grid of cards, each showing title, description, and a thumbnail preview of the actual diagram/content (lighter for blank/whiteboard-style templates with little to preview). Clicking a card instantiates immediately and opens the canvas — no separate preview/confirm step.
- **AI-generated**: a card with an inline prompt input built in from the start (no extra click to "select" AI mode first — typing there is the action).

**Naming**

- Canvas starts as "Untitled" + timestamp. No naming is ever forced on the user.
- The canvas title (in the canvas header) is always inline-renamable at any time.
- One universal rule: the moment a canvas has content and its name is still the default (i.e., the user hasn't manually renamed it), the AI fires a one-time auto-rename based on that content, shown with a visible "typing" animation (like chat-thread naming). This covers template and AI-generated canvases immediately (they have content at t=0) and blank canvases whenever the user first adds something. It never re-fires once a name has been set (by the user or by this one-time pass).

**AI generation flow**

- The prompt is entered on the creation surface itself; the canvas does not exist yet at that point.
- On submit, the surface transitions to a loading/generating state in place (no separate transitional screen, no live-streaming build).
- Only once generation fully completes does the app navigate the user into the finished canvas.
- While generating, a "Cancel" action aborts and returns to the prompt input; no canvas is created if cancelled.
- On failure, show an inline error on the same surface with a "try again" action — no silent fallback to a blank canvas.
