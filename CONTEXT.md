# Context

## Glossary

- canvas: the only primary user-facing object in the product. A canvas is private by default and can be shared with other authenticated users in the organization. Shared users can edit it.
- template: a prebuilt arrangement of the allowed canvas primitives. Templates are fully editable after creation.
- AI assistant: the canvas-scoped chat agent. It can answer questions about the current canvas and apply requested changes using only the allowed primitives.
- organization: the authentication and authorization boundary for the app. Users are admitted by matching the approved email domain and are auto-provisioned on first sign-in.
- workspace: not a user-facing product concept in this product.
- slide: not a separate product object. Any slide-like area is just a rectangle with content on the canvas.

## Allowed canvas primitives

- text
- shapes
- connectors

