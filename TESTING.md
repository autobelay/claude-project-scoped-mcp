# Tested behavior

## v0.1.1 installed plugin

On 2026-10-24, the GitHub marketplace and user-scoped installation were updated to **v0.1.1** on macOS with **Claude Code 2.1.310** and **Node.js 24.22.0**. `claude plugin details project-scoped-mcp` showed **Local Project MCP (project-scoped-mcp) 0.1.1**, the new description, and one MCP server named `local-project-mcp`.

An ordinary `claude -p` session in `examples/design-studio` loaded the installed plugin from the marketplace. It used no `--plugin-dir`, `--mcp-config`, or launch wrapper. Built-in tools were disabled, and only tools from `plugin_project-scoped-mcp_local-project-mcp` were allowed. Claude called `project_mcp_status` and the project's `read_design_tokens` tool. Status reported one connected server with one tool; the token call returned the fictional Field Notes colors, spacing, typography, and principles. No project files were written.

`npm run build` and all eight synthetic tests passed. `claude plugin validate .` passed; its only warning concerned optional artwork for an Anthropic directory listing. These checks and the CLI details output do not establish how every Claude Code interface renders the new tool titles.

## v0.1.0 installed sessions

Release **v0.1.0** was installed from the public GitHub marketplace and exercised with **Claude Code 2.1.309**, **Node.js 24.22.0**, on **macOS**, on 2026-10-23.

The final runs used the user-scoped GitHub installation with ordinary `claude -p` sessions. They did not use `--plugin-dir`, `--mcp-config`, a launch wrapper, or generated project configuration. Only the requested read-only plugin tools were allowed for these tests; built-in tools were disabled.

| Session directory | Observed result |
| --- | --- |
| Existing private project | All 19 manifest servers connected; 339 downstream tools discovered; a project-only local read-only status tool succeeded. |
| This repository's root | No manifest and zero downstream servers. |
| `examples/design-studio` | One server connected; Claude called `read_design_tokens` and received the fictional Field Notes tokens. |

Private-project results are summarized only. No private manifest, company identifiers, server implementation, credentials or raw tool transcript is published here. Discovery and one read-only call do not mean every downstream tool was executed.

Both the plugin manifest and marketplace manifest passed `claude plugin validate` with no manifest errors or warnings. This does not establish an Anthropic directory listing or approval. The optional directory checks separately note the absence of listing artwork.

## Local demonstration

The public example uses only fictional data. Install the plugin from this repository's marketplace, then start Claude Code from the example directory:

```sh
claude plugin marketplace add autobelay/claude-project-scoped-mcp
claude plugin install project-scoped-mcp@autobelay-claude-project-mcp --scope user
git clone https://github.com/autobelay/claude-project-scoped-mcp.git
cd claude-project-scoped-mcp/examples/design-studio
claude
```

Ask Claude: “Use only Local Project MCP: call `project_mcp_status`, then the project-design `read_design_tokens` tool. Report the returned tokens. Do not use shell or write files.”

In a separate session at the repository root, ask: “Call only `project_mcp_status`. Report whether a project manifest was found and the downstream server count.” The root has no `mcp.json`; the expected count is zero. The `plugin.mcp.json` file configures the plugin's bridge connection and has a different role from the project's `mcp.json` or a native Claude Code `.mcp.json`.

## Automated tests

```sh
npm ci
npm run build
npm test
```

The tests use synthetic fixtures and local processes. They cover nearest-manifest discovery, nested checkouts and worktrees, environment interpolation, distinct tool names, forwarding of tools/resources/prompts, pagination, cwd, project isolation, a failed sibling server, authenticated Streamable HTTP, structured results, child environments, explicit project paths containing spaces while the process runs from another directory, and refusal to fall back to an ancestor when a project path is missing or unexpanded. They do not use a private repository or external service.

Eight automated tests passed. The live sessions also confirmed that the plugin receives the project directory through Claude Code's documented substitution rather than assuming the plugin process working directory is the project.

Other operating systems, older Claude versions, desktop/IDE-specific sessions, hosted execution and interactive downstream OAuth are not covered by these observations.
