# Claude Code marketplace distribution

This repository contains both a Claude Code plugin and an independent marketplace catalog. The plugin manifest is `.claude-plugin/plugin.json`, the catalog is `.claude-plugin/marketplace.json`, and `plugin.mcp.json` declares one local stdio bridge. The manifest points to that MCP configuration. The catalog points to the repository root with `"source": "./"`, as shown in [Claude Code's publishing guide](https://code.claude.com/docs/en/plugins/publish#add-the-marketplace-file-to-your-repository).

```sh
claude plugin marketplace add autobelay/claude-project-scoped-mcp
claude plugin install project-scoped-mcp@autobelay-claude-project-mcp --scope user
```

Claude Code expands `${CLAUDE_PLUGIN_ROOT}` to the installed plugin directory and `${CLAUDE_PROJECT_DIR}` to the current project root in the MCP server arguments. The plugin runs its bundled bridge with Node 22 or newer. It has no runtime package download, installation script, hook, daemon, or hosted endpoint. A project's own servers may need additional software or credentials. See the [plugin variable reference](https://code.claude.com/docs/en/plugins-reference#environment-variables) and [MCP documentation](https://code.claude.com/docs/en/mcp#plugin-provided-mcp-servers).

## Anthropic directory status

This GitHub marketplace is separate from Anthropic's directory and official marketplace. This plugin is not an Anthropic directory listing and is not affiliated with Anthropic. [Claude Code's publishing guide](https://code.claude.com/docs/en/plugins/publish#submit-to-anthropics-directory) describes directory submission through the developer portal, while a self-hosted marketplace can be published directly from a Git repository. Directory distribution and its review are Anthropic's decisions.

## Public demonstration

Open `examples/design-studio` in a local Claude Code session with the plugin installed. Its `mcp.json` starts a bundled, synthetic server whose read-only `read_design_tokens` tool returns fictional Field Notes tokens. In a separate session at this repository's root, `project_mcp_status` reports no project manifest and no downstream servers. The example needs no account, credentials, cloud service, or company data. See [TESTING.md](TESTING.md) for reproduction commands and observations.

## Building the bundled executable

`npm ci` and `npm run build` produce `dist/bridge.cjs` and the synthetic example bundle. The runtime bridge has no npm dependencies. The JavaScript bundles are committed in the plugin repository so an installed plugin can start them directly with Node. `npm test` exercises the bridge against synthetic fixtures.
