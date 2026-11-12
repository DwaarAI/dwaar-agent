# Dwaar for coding agents

[Dwaar Console](https://docs.dwaar.ai/console/what-is-dwaar-console) finds everything your company ships, scans it,
and lists what is wrong with it in one place. This repository connects your coding agent to it:

- **The MCP server** `https://console.stg.dwaar.ai/mcp` (console `https://console.stg.dwaar.ai`): findings,
  resources, components, products, scans and connections from chat, and rescans to prove a fix. The agent signs in as
  you, in your browser, and can never do more than your role allows.
- **Two skills**: `dwaar-connect` (install, sign in, connect source control, first findings) and
  `dwaar-security-review` (the findings of the repository you are in, their fixes, and a rescan).

The fastest path: paste `Connect me to Dwaar (docs.dwaar.ai)` into your agent. Or install by hand below. Full guide:
[Connect a coding agent](https://docs.dwaar.ai/console/getting-started/coding-agents).

These are the shared addresses, and the default. On a dedicated Dwaar tenant, see [Dedicated tenants](#dedicated-tenants).

## Claude Code

```bash
claude plugin marketplace add DwaarAI/dwaar-agent
claude plugin install dwaar@dwaar
```

MCP only:

```bash
claude mcp add --transport http dwaar https://console.stg.dwaar.ai/mcp
```

Sign in after the plugin install: the server is `plugin:dwaar:dwaar`, so `/mcp` › plugin:dwaar:dwaar › Authenticate,
or `claude mcp login plugin:dwaar:dwaar` (`claude mcp login plugin:dwaar:dwaar --no-browser` over SSH).

Sign in after the MCP-only install: the server is `dwaar`, so `/mcp` › dwaar › Authenticate, or
`claude mcp login dwaar` (`claude mcp login dwaar --no-browser` over SSH).

## Codex (CLI and IDE)

```bash
codex plugin marketplace add DwaarAI/dwaar-agent
codex plugin add dwaar@dwaar
```

Or, after adding the marketplace, run `/plugins` in Codex and install **dwaar**.

MCP only:

```bash
codex mcp add dwaar --url https://console.stg.dwaar.ai/mcp
```

Sign in: `codex mcp login dwaar`.

## Cursor

Plugin: a team admin imports this repository as a team marketplace (Dashboard › Plugins & MCPs › Add Marketplace ›
Import from Repo, `https://github.com/DwaarAI/dwaar-agent`), then members install **dwaar** from Customize.

MCP only: open
`cursor://anysphere.cursor-deeplink/mcp/install?name=dwaar&config=eyJ1cmwiOiJodHRwczovL2NvbnNvbGUuc3RnLmR3YWFyLmFpL21jcCJ9`,
or add it to `~/.cursor/mcp.json`:

```json
{ "mcpServers": { "dwaar": { "url": "https://console.stg.dwaar.ai/mcp" } } }
```

Skills only: `npx skills add DwaarAI/dwaar-agent`.

Sign in: Cursor runs the OAuth sign-in when the server connects.

## VS Code (GitHub Copilot)

Enable `chat.plugins.enabled`, run **Chat: Install Plugin from Source** from the Command Palette, and enter
`https://github.com/DwaarAI/dwaar-agent`.

MCP only:

```bash
code --add-mcp '{"name":"dwaar","type":"http","url":"https://console.stg.dwaar.ai/mcp"}'
```

Sign in: VS Code opens the browser on the first connection.

## Antigravity

```bash
agy plugin install https://github.com/DwaarAI/dwaar-agent
```

MCP only: add the server to `~/.gemini/config/mcp_config.json`:

```json
{ "mcpServers": { "dwaar": { "serverUrl": "https://console.stg.dwaar.ai/mcp" } } }
```

Sign in: Antigravity opens the browser when the server connects.

## Gemini CLI

Google replaced Gemini CLI with Antigravity CLI for individual accounts on 18 June 2026; use the Antigravity section
above unless your organization still runs Gemini CLI.

```bash
gemini extensions install https://github.com/DwaarAI/dwaar-agent
```

MCP only:

```bash
gemini mcp add --transport http -s user dwaar https://console.stg.dwaar.ai/mcp
```

Sign in: `/mcp auth dwaar`.

## OpenCode

```bash
npx skills add DwaarAI/dwaar-agent
opencode mcp add dwaar --url https://console.stg.dwaar.ai/mcp
```

Sign in: `opencode mcp auth dwaar`.

## Windsurf, Devin, Zed, Cline and others

Add the skills with `npx skills add DwaarAI/dwaar-agent`, then add `https://console.stg.dwaar.ai/mcp` as a remote MCP
server (Streamable HTTP, OAuth) in the agent's MCP settings and sign in the way it signs in to any MCP server.

## Dedicated tenants

A customer with a dedicated Dwaar tenant has its own console and MCP addresses, and the MCP address may be on a
different domain from the console. Use a dedicated address only if Dwaar gave you one. The plugin carries the shared
address, so install the skills and add the server with your address using the agent's MCP-only command, for example in
Claude Code:

```bash
npx skills add DwaarAI/dwaar-agent
claude mcp add --transport http dwaar <your MCP address>
```

Then sign in with `claude mcp login dwaar`. Once signed in, the server's instructions name your console.

## What happens at sign-in

The browser opens and you pass up to three stops:

1. **Sign in or sign up** with your work email. Personal addresses are refused.
2. **Create your organization**, the first time only, when nobody from your email domain uses Dwaar yet.
3. **Consent**: **Read only**, or **Read and write** to let the agent start scans and syncs. Then **Approve**.

The agent gets an API key named after it with "(MCP)", valid for 30 days and cut to your role on every call. Revoke it
in the console under Settings › API.

Then connect source control in the console (for GitHub, `https://console.stg.dwaar.ai/integrations/github` › Add
connection). Credentials never pass through chat.

## Headless (CI, `claude -p`)

No browser, so use an API key from Settings › API (preset **Read only (MCP)**) as a bearer token:

```bash
claude mcp add --transport http dwaar https://console.stg.dwaar.ai/mcp --header "Authorization: Bearer dwc_…"
codex mcp add dwaar --url https://console.stg.dwaar.ai/mcp --bearer-token-env-var DWAAR_API_KEY
```

## What is in this repository

| File | Read by |
| --- | --- |
| `plugin.json`, `mcp.json` | [Agent Plugins 1.0](https://agent-plugins.org): Codex, Cursor, VS Code, Antigravity |
| `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.mcp.json` | Claude Code (`.mcp.json` also by VS Code for Claude-format plugins) |
| `.codex-plugin/plugin.json`, `.agents/plugins/marketplace.json` | Codex |
| `.cursor-plugin/plugin.json`, `.cursor-plugin/marketplace.json` | Cursor |
| `mcp_config.json` | Antigravity |
| `gemini-extension.json`, `GEMINI.md` | Gemini CLI |
| `skills/*/SKILL.md` | Every agent ([Agent Skills](https://agentskills.io)) |
| `install.md` | Agents following [install.md](https://github.com/mintlify/install-md) |

## Changing the console host

The shared MCP server is on `console.stg.dwaar.ai` until a production console exists. To move it, change the host in
`mcp.json`, `.mcp.json`, `mcp_config.json`, `gemini-extension.json`, the skills, `install.md` and this README
(`grep -rl console.stg.dwaar.ai .`), including the Cursor link's `config` (base64 of `{"url":"<MCP address>"}`), and
bump `version` in `plugin.json`, `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json`, `.cursor-plugin/plugin.json`
and `gemini-extension.json` so installed copies update.

## License

[Apache-2.0](LICENSE)
