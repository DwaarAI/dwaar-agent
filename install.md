# dwaar

> Dwaar Console finds everything a company ships, scans it, and lists the findings in one place. This installs Dwaar's MCP server and skills into the coding agent you are, and signs the user in.

I want you to install Dwaar for me. Execute all the steps below; ask me once before you change my agent's configuration.

OBJECTIVE: Give this coding agent the `dwaar` MCP server (`https://console-dev.dwaar.ai/mcp`) and the Dwaar skills, signed in as me.

DONE WHEN: the `session` tool of the `dwaar` server answers with my name and my tenant.

## TODO

- [ ] Work out which coding agent you are
- [ ] Install the Dwaar plugin (or the MCP server alone) for that agent
- [ ] Reload the agent so it loads the new server
- [ ] Sign me in through the browser
- [ ] Call `session`, then `overview`, and tell me what you found

## Step 1: Install for your agent

You need to run the plugin command for the agent you are. Use the MCP-only command only when the agent has no plugin
support.

| Agent | Plugin | MCP only |
| --- | --- | --- |
| Claude Code | `claude plugin marketplace add DwaarAI/dwaar-agent && claude plugin install dwaar@dwaar` | `claude mcp add --transport http dwaar https://console-dev.dwaar.ai/mcp` |
| Codex | `codex plugin marketplace add DwaarAI/dwaar-agent && codex plugin add dwaar@dwaar` | `codex mcp add dwaar --url https://console-dev.dwaar.ai/mcp` |
| Cursor | (team marketplace import by an admin) | `~/.cursor/mcp.json`: `{"mcpServers":{"dwaar":{"url":"https://console-dev.dwaar.ai/mcp"}}}` |
| VS Code (Copilot) | Command Palette › **Chat: Install Plugin from Source** › `https://github.com/DwaarAI/dwaar-agent` | `code --add-mcp '{"name":"dwaar","type":"http","url":"https://console-dev.dwaar.ai/mcp"}'` |
| Antigravity | `agy plugin install https://github.com/DwaarAI/dwaar-agent` | `~/.gemini/config/mcp_config.json`: `{"mcpServers":{"dwaar":{"serverUrl":"https://console-dev.dwaar.ai/mcp"}}}` |
| Gemini CLI | `gemini extensions install https://github.com/DwaarAI/dwaar-agent` | `gemini mcp add --transport http -s user dwaar https://console-dev.dwaar.ai/mcp` |
| OpenCode | `npx skills add DwaarAI/dwaar-agent` | `opencode mcp add dwaar --url https://console-dev.dwaar.ai/mcp` |
| Other agents | `npx skills add DwaarAI/dwaar-agent` | Add `https://console-dev.dwaar.ai/mcp` as a remote Streamable HTTP MCP server with OAuth |

OpenCode and other agents need both lines: the skills and the server.

## Step 2: Reload

Most agents load a new MCP server only in a new session. Tell me to restart or reload the agent if yours does.

## Step 3: Sign in

You must start sign-in with the agent's login command, or by calling any Dwaar tool:

- Claude Code: `/mcp` › dwaar › Authenticate, or `claude mcp login dwaar`
- Codex: `codex mcp login dwaar`
- Gemini CLI: `/mcp auth dwaar`
- OpenCode: `opencode mcp auth dwaar`
- Cursor, VS Code, Antigravity: the browser opens when the server connects

Tell me what each browser stop is for before it opens:

1. Sign in or sign up with my work email. Personal addresses are refused.
2. **Create your organization**, only the first time, when nobody from my email domain uses Dwaar yet.
3. Consent: **Read only**, or **Read and write** if I want you to start scans and syncs. Then **Approve**.

Never ask me for a password, token or code in chat.

## Step 4: Verify

Call `session` and greet me by name with my tenant. Call `overview` and summarise it. If `session` says the tenant is
empty, send me to `https://console-dev.dwaar.ai/integrations/github` and ask me to choose **Add connection**: credentials
never go through chat.

EXECUTE NOW: Complete the above TODO list to achieve: the `session` tool of the `dwaar` server answers with my name and my tenant.

For more context, read https://docs.dwaar.ai/llms.txt and https://docs.dwaar.ai/console/getting-started/coding-agents.
