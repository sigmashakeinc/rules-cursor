# rules-cursor

Cursor AI editor governance rules — guards against CurXecute MCP config injection (CVE-2025-54135/54136), silent code execution via .vscode/tasks.json with workspace trust disabled, SSH key exfiltration, and malicious npm packages that target Cursor users.

**14 rules · 3 files**

![rules-cursor — AI agent governance demo](demo.cast)

> [▶ Watch interactive demo on SigmaShake Hub](https://hub.sigmashake.com/ruleset/rules-cursor)

## Install

```bash
ssg hub pull rules-cursor
```

Available on the [SigmaShake Hub](https://hub.sigmashake.com) — the open registry for AI agent governance rules. Compatible with Claude Code and any AI coding agent using the `ssg` hook protocol.

## Rules

### cursor_exec_mcp_injection.rules — MCP injection and execution (5 rules)

| Rule | Decision | Severity | Description |
|------|----------|----------|-------------|
| `no-cursor-mcp-config-rewrite` | DENY | error | CVE-2025-54135: Blocks .cursor/mcp.json writes (CurXecute RCE vector) |
| `no-cursor-mcp-config-swap` | DENY | error | CVE-2025-54136: Blocks MCP config copy/move/symlink operations |
| `no-cursor-vscode-tasks-write` | DENY | error | Blocks .vscode/tasks.json writes (auto-execute on folder open) |
| `ask-cursor-settings-modification` | ASK | warning | Prompts before .vscode/settings.json or .cursorrules writes |
| `log-cursor-extension-install` | LOG | info | Audits all Cursor/VS Code extension installations |

### cursor_read_ssh_exfil.rules — SSH and credential read protection (4 rules)

| Rule | Decision | Severity | Description |
|------|----------|----------|-------------|
| `no-cursor-ssh-private-key-read` | DENY | error | Blocks reading .ssh/id_* private keys and SSH config |
| `no-cursor-npm-credentials-read` | DENY | error | Blocks .npmrc/.yarnrc reads (npm token theft vector) |
| `ask-cursor-dotfile-read` | ASK | warning | Prompts before reading home directory dotfiles |
| `log-cursor-env-file-read` | LOG | info | Audit trail for .env file access in Cursor sessions |

### cursor_write_supply_chain.rules — Supply chain write safety (5 rules)

| Rule | Decision | Severity | Description |
|------|----------|----------|-------------|
| `ask-cursor-package-lifecycle-scripts` | ASK | warning | Prompts on preinstall/postinstall hooks in package.json |
| `no-cursor-rules-prompt-injection` | DENY | error | Blocks prompt injection in .cursorrules (system prompt override) |
| `ask-cursor-mcp-server-reference` | ASK | warning | Prompts on new MCP server references in config files |
| `no-cursor-eval-in-vscode-config` | DENY | error | Blocks eval/Function/child_process in .cursor/.vscode config |
| `log-cursor-rules-write` | LOG | info | Audit trail for .cursorrules file modifications |

## Why this matters

Cursor ships with **VS Code Workspace Trust disabled by default** — which means any config file in a cloned repository can execute code silently when the project is opened. Combined with Cursor's powerful AI capabilities, this creates a uniquely dangerous attack surface:

- **CVE-2025-54135 (CurXecute)** (Oasis Security): Attackers crafted malicious Slack messages that, when summarized by Cursor's AI, rewrote `.cursor/mcp.json` to point to attacker-controlled MCP servers and executed arbitrary commands with developer privileges — without any user approval.
- **CVE-2025-54136**: If an attacker has write access to a branch containing MCP server configs, they can swap harmless configs with malicious commands.
- **Workspace Trust disabled**: A `.vscode/tasks.json` with `runOn: "folderOpen"` turns a "casual open folder" into silent code execution (Oasis Security Research).
- **SSH key exfiltration**: Documented attack: using prompt injection to force Cursor to read and exfiltrate SSH private keys via benign tool combinations.
- **Malicious npm packages**: SecurityWeek documented malicious npm packages specifically targeting macOS Cursor users for credential theft.

## Compatible AI clients

- Cursor (primary target)
- Works alongside: `rules-security`, `rules-secrets`, `rules-mcp`

## About

Part of the [SigmaShake Hub](https://hub.sigmashake.com) — open-source governance rules for AI coding agents.
Install the `ssg` CLI to enforce these rules: `npm install -g @sigmashake/ssg`
