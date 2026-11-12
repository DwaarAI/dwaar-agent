---
name: dwaar-connect
description: Connect this coding agent to Dwaar Console, sign in, and get the first findings. Use when the user asks what Dwaar is, asks to connect, install, set up or sign in to Dwaar, or when a Dwaar tool is missing, answers 401 or "needs authentication", or lacks the write tools.
license: Apache-2.0
---

# Connect to Dwaar

Dwaar Console finds everything a company ships (repositories, images and packages, cloud accounts and workloads, APIs,
AI agents, vendors), scans it, and lists the problems as findings in one place, ranked with severity, EPSS and CISA KEV.
Its MCP server lets you read findings, resources, components and scans from chat, and start rescans.

- MCP server: `https://console.stg.dwaar.ai/mcp`, the shared address, server name `dwaar` (Streamable HTTP, OAuth
  sign-in). The Claude Code plugin registers it as `plugin:dwaar:dwaar`.
- Console (`<console>` below): the console the dwaar server's instructions name ("The console is …"). Without them,
  it is `https://console.stg.dwaar.ai` on the shared server; on a dedicated tenant, ask the user for their console
  address. Never derive it from the MCP address: the MCP server and the console can be on different domains.

Never ask the user for a password, token, OAuth code or integration credential in chat. Sign-in and connecting
integrations happen in the browser.

## 0. Shared or dedicated address

Use the shared MCP address above unless the user explicitly says they have a dedicated Dwaar tenant and gives its MCP
address. Never guess a dedicated address, and do not ask every user whether they have one. A dedicated tenant has its
own console and MCP addresses, possibly on different domains. With a dedicated address, install the skills
(`npx skills add DwaarAI/dwaar-agent`) and use the agent's MCP-only command from step 2 with that address in place of
the shared one: the plugin carries the shared address.

## 1. Check the tools

Look for the Dwaar tools: `session`, `search`, `overview`, `orgs`, `products`, `product`, `resources`, `resource`,
`components`, `component`, `findings`, `finding`, `findingCounts`, `scans`, `scan`, `providers`, `discoveryJobs`,
`connections`, and with write access `rescanResource`, `retryScan`, `cancelScan`, `syncConnection`. Your agent may show
them with a prefix for the server (for example `mcp__plugin_dwaar_dwaar__session` after the Claude Code plugin install,
`mcp__dwaar__session` after an MCP-only install).

- The tools are there: go to step 4.
- The Dwaar server (`dwaar`, or `plugin:dwaar:dwaar` from the Claude Code plugin) is configured but needs sign-in, or a
  call answers 401: go to step 3.
- No Dwaar server: go to step 2.

## 2. Install

Work out which agent you are. Give the user the one command for it, say what it changes, and ask before you run it.
Prefer the plugin: it adds the server and these skills. Use the MCP-only command when the agent has no plugins or the
user wants only the server.

| Agent | Plugin | MCP only |
| --- | --- | --- |
| Claude Code | `claude plugin marketplace add DwaarAI/dwaar-agent` then `claude plugin install dwaar@dwaar` | `claude mcp add --transport http dwaar https://console.stg.dwaar.ai/mcp` |
| Codex | `codex plugin marketplace add DwaarAI/dwaar-agent` then `codex plugin add dwaar@dwaar` (or `/plugins`) | `codex mcp add dwaar --url https://console.stg.dwaar.ai/mcp` |
| Cursor | A team admin imports `https://github.com/DwaarAI/dwaar-agent` (Dashboard › Plugins & MCPs › Add Marketplace › Import from Repo) | Add `{"mcpServers":{"dwaar":{"url":"https://console.stg.dwaar.ai/mcp"}}}` to `~/.cursor/mcp.json` |
| VS Code (Copilot) | Command Palette › **Chat: Install Plugin from Source** › `https://github.com/DwaarAI/dwaar-agent` (needs `chat.plugins.enabled`) | `code --add-mcp '{"name":"dwaar","type":"http","url":"https://console.stg.dwaar.ai/mcp"}'` |
| Antigravity | `agy plugin install https://github.com/DwaarAI/dwaar-agent` | Add `"dwaar": {"serverUrl": "https://console.stg.dwaar.ai/mcp"}` under `mcpServers` in `~/.gemini/config/mcp_config.json` |
| Gemini CLI | `gemini extensions install https://github.com/DwaarAI/dwaar-agent` | `gemini mcp add --transport http -s user dwaar https://console.stg.dwaar.ai/mcp` |
| OpenCode | `npx skills add DwaarAI/dwaar-agent` (skills only) | `opencode mcp add dwaar --url https://console.stg.dwaar.ai/mcp` |
| Anything else | `npx skills add DwaarAI/dwaar-agent` (skills only) | Add `https://console.stg.dwaar.ai/mcp` as a remote (Streamable HTTP, OAuth) MCP server in its settings |

Most agents load a new MCP server only in a new session. Tell the user to restart or reload the agent, then continue
with step 3.

If the agent cannot open a browser (CI, `claude -p`, a remote shell without one), sign-in cannot work. Use an API key
instead: the user creates one in `<console>` › Settings › API (preset **Read only (MCP)**) and adds the server with the
header `Authorization: Bearer dwc_…` themselves, for example
`claude mcp add --transport http dwaar https://console.stg.dwaar.ai/mcp --header "Authorization: Bearer dwc_…"` or
`codex mcp add dwaar --url https://console.stg.dwaar.ai/mcp --bearer-token-env-var DWAAR_API_KEY`. Do not ask them to paste
the key into chat.

## 3. Sign in

Sign-in starts on the first Dwaar tool call, or with the agent's login command:

| Agent | Sign in |
| --- | --- |
| Claude Code, plugin install | `/mcp` › plugin:dwaar:dwaar › Authenticate, or `claude mcp login plugin:dwaar:dwaar` |
| Claude Code, MCP-only install | `/mcp` › dwaar › Authenticate, or `claude mcp login dwaar` |
| Codex | `codex mcp login dwaar` |
| Cursor | Prompted when the server connects; its MCP settings show the server's sign-in state |
| VS Code | Prompted on the first tool call |
| Antigravity | Prompted in the browser when the server connects |
| Gemini CLI | `/mcp auth dwaar` |
| OpenCode | `opencode mcp auth dwaar` |

Over SSH without a local browser, `claude mcp login plugin:dwaar:dwaar --no-browser` (plugin install),
`claude mcp login dwaar --no-browser` (MCP-only install) and `codex mcp login dwaar --no-browser` print a URL to open
elsewhere; the user pastes back the address they land on.

Before the browser opens, tell the user what each stop is for:

1. **Sign in or sign up** with their work email. Personal addresses (Gmail, Outlook and the like) are refused.
2. **Create your organization**, only the first time, when nobody from their email domain uses Dwaar yet. The company
   name is the usual choice; ticking the confirmation makes them the tenant's first administrator.
3. **Consent.** Choose **Read only**, or **Read and write** if they want you to start scans and syncs
   (`rescanResource`, `retryScan`, `cancelScan`, `syncConnection`), then **Approve**.

The agent receives an API key named after it with "(MCP)", valid 30 days, limited to what their role allows. They can
revoke it in `<console>` › Settings › API.

## 4. Confirm the connection

1. Call `session`. Greet the user by name and name their tenant. `tenant.isEmpty` says whether anything is connected yet.
2. Call `overview` for the picture: findings by severity, the most urgent ones, coverage per resource type.
3. Tenant empty, or no resources: go to step 5. Otherwise go to step 6.

## 5. Connect source control (browser)

No tool can create a connection: a connection stores a credential, and credentials never pass through chat.

1. Send the user to `<console>/integrations/github` and ask them to choose **Add connection**. With the **GitHub App**
   method it is a few clicks on GitHub, where they pick the repositories. For GitLab, Bitbucket, a registry or a cloud,
   call `providers` and use `<console>/integrations/<provider id>`.
2. Ask them to tell you when it is saved. Then follow it, waiting about 15 s, 30 s, 60 s, then 2 min between checks,
   and give a one-line progress note at each check:
   - `connections` (`provider=github`): the new connection, with its id and status.
   - `discoveryJobs` (`connection=<id>`): until the first job is `completed`. It reports `scansCreated`.
   - `scans` (`status=queued,running`): until none are left, or enough have completed to show findings.
3. A failed discovery or scan: show its error and link `<console>/integrations/github`. After about 20 minutes with
   work still queued, stop polling, say what is still running, and offer to check again later.

## 6. First summary

1. `findingCounts` with no filters: sum the open cells by severity and by type.
2. `findings` with `severity=critical,high`, `first=10`.
3. Give a short summary: totals by severity, then the worst five with their resource, component or CVE, KEV and EPSS,
   each linked as `<console>/findings?finding=<id>`. Report only what the tools return.
4. If the user is in a code repository, offer the `dwaar-security-review` skill: match this repository, explain its
   findings, fix them and rescan.

## Troubleshooting

| Symptom | Cause | What to do |
| --- | --- | --- |
| 401, "needs authentication", tools gone | Not signed in yet, the 30-day key expired, or it was revoked in Settings › API | Run the agent's sign-in from step 3 again |
| Browser shows **Ask for an invite** | The company already uses Dwaar | Ask one of its administrators for an invite, then sign in again from the agent |
| Browser shows **Use your organization email** | Signed in with a personal address | Choose **Sign out**, then sign in with the work email |
| Browser shows **Choose an organization** | The person belongs to several tenants | Pick the one to work in |
| Browser shows **This sign-in cannot continue** | The request was wrong or expired | Start sign-in again from the agent |
| `rescanResource` and the other write tools are missing | Signed in with **Read only**, or the role lacks `scans:run` / `scans:cancel` / `integrations:sync` | Sign in again and choose **Read and write**; if only **Read only** is offered, an administrator must grant the role |
| A tool answers `not_found` for something the user sees in the console | It is in an organization the key cannot read, or another org than expected | Call `orgs` and pass the right `org` |
| An error ending in `· Ref <id>` | Server-side error | Quote the message and the Ref to the user |
