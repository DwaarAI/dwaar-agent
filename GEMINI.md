# Dwaar

This extension adds the `dwaar` MCP server (Dwaar Console: findings, resources, components and scans of the user's
company) and two skills:

- `dwaar-connect`: what Dwaar is, signing in (`/mcp auth dwaar`), connecting source control, the first findings. Use it
  when the user asks about Dwaar or a Dwaar tool answers 401.
- `dwaar-security-review`: the findings of the repository you are in, their fixes, and a rescan to prove them.

Report only what the Dwaar tools return. Never ask for passwords, tokens or integration credentials in chat: sign-in and
connecting integrations happen in the browser.
