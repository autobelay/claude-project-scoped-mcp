# Local demonstration

The public example uses only fictional data. Install the plugin from this repository's marketplace, then start Claude Code from the example directory:

```sh
claude plugin marketplace add autobelay/claude-project-scoped-mcp
claude plugin install project-scoped-mcp@autobelay-claude-project-mcp --scope user
git clone https://github.com/autobelay/claude-project-scoped-mcp.git
cd claude-project-scoped-mcp/examples/design-studio
claude
```

Ask Claude: “Use only the project-scoped MCP: call `project_mcp_status`, then the project-design `read_design_tokens` tool. Report the returned tokens. Do not use shell or write files.”

In a separate session at the repository root, ask: “Call only `project_mcp_status`. Report whether a project manifest was found and the downstream server count.” The root has no `mcp.json`; the expected count is zero. The `plugin.mcp.json` file configures the plugin's bridge connection and has a different role from the project's `mcp.json` or a native Claude Code `.mcp.json`.

## Automated tests

```sh
npm ci
npm run build
npm test
```

The tests use synthetic fixtures and local processes. They cover nearest-manifest discovery, nested checkouts and worktrees, environment interpolation, distinct tool names, forwarding of tools/resources/prompts, pagination, cwd, project isolation, a failed sibling server, authenticated Streamable HTTP, structured results, and child environments. They do not use a private repository or external service.
