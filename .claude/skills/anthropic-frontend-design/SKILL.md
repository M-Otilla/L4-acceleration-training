---
name: anthropic-frontend-design
description: Design and implement polished, accessible frontend experiences for this workspace. Use this skill when asked to build screens, improve UX, create app layouts, tune visual polish, or turn a product idea into a front-end implementation.
argument-hint: "feature or screen description"
model: sonnet
---

# Frontend design

Use this skill to turn a product requirement into a clear, buildable frontend experience that matches the existing app and user needs.

## 1. Start from the product problem

- Identify the user, goal, and context before choosing UI patterns.
- Confirm constraints such as mobile vs desktop, data shape, edge cases, and business rules.
- If requirements are ambiguous, ask only the missing product questions before designing details.

## 2. Match the existing app

- Read the current frontend structure, route layout, and component patterns before making changes.
- Reuse existing design tokens, styling conventions, component names, and state handling.
- Prefer the smallest possible change that solves the user problem.

## 3. Design the interaction

- Describe the screen or flow in plain language: who uses it, what they want, and what happens next.
- Prefer a simple information hierarchy: primary action, supporting context, secondary actions.
- Design for real content and edge states; show empty, loading, error, and validation states.
- Keep the interface responsive and consistent across screen sizes.

## 4. Build accessible, polished UI

- Use semantic HTML, clear labels, and proper keyboard focus states.
- Ensure contrast, spacing, alignment, and visual hierarchy support readability.
- Use concise copy, consistent terminology, and familiar patterns from the app.
- Avoid unnecessary animation and over-design; prioritize usability and clarity.

## 5. Validate before finishing

- Check that the page works with real data and common interaction paths.
- Verify accessibility basics: focus order, labels, buttons, and forms.
- Confirm the design remains consistent with the current app structure.
- Explain trade-offs and any assumptions clearly.

## 6. Deliver output

When asked to implement, produce the UI in the repo's existing framework and conventions. When asked for design guidance, provide:

- a concise summary of the UX goal
- the proposed component/layout structure
- key states and edge cases
- accessibility and interaction considerations
- implementation notes for the current workspace

Treat the interface as a product experience, not just a set of components. Make it clear, usable, and polished.
