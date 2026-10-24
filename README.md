# Local Project MCP for Claude Code

Install once. Open a project in Claude Code. Use the MCP servers declared by that project.

Local Project MCP automatically reads the nearest `mcp.json` for the current project when a Claude Code session starts. It connects the local and remote servers declared there and exposes their tools through one session-scoped MCP connection. It uses Claude Code's documented plugin and project directory variables. There is no launch wrapper or configuration synchronization.

## Install

Requires Node.js 22 or newer, Git, and Claude Code with plugin marketplace support. Claude Code 2.1.309 is the version exercised locally; older versions have not been tested here. See [tested behavior](TESTING.md) for live installation results.

```sh
claude plugin marketplace add autobelay/claude-project-scoped-mcp
claude plugin install project-scoped-mcp@autobelay-claude-project-mcp --scope user
```

Start a new local session in a project containing `mcp.json`:

```sh
cd /path/to/project
claude
```

Ask: “Use `project_mcp_status` from Local Project MCP to report which project MCP servers are connected.” The underlying tools are individually available; Claude does not have to call a generic dispatcher.

The plugin launches its bundled JavaScript bridge directly with Node. It does not download an executable at runtime or run an install script. A project's own servers may need additional runtimes, credentials, or network access. The install has no machine-specific project path.

### Update and remove

```sh
claude plugin marketplace update autobelay-claude-project-mcp
claude plugin update project-scoped-mcp@autobelay-claude-project-mcp
```

Start a new session to use an updated bridge. To remove the plugin and marketplace:

```sh
claude plugin uninstall project-scoped-mcp@autobelay-claude-project-mcp
claude plugin marketplace remove autobelay-claude-project-mcp
```

## Why project-scoped tools?

An MCP connection has context: which checkout, credentials, assets, and product conventions it should use. That context often belongs to a repository. A global server list gives every project irrelevant tools, and identically named servers can refer to different applications. Copying project declarations into another configuration leaves two representations to maintain.

A manifest can travel with its implementation and evolve alongside it. This bridge resolves that manifest at session startup, launches its local servers from the manifest directory, and keeps each session's connections separate.

## Example: a design assistant for one application

Imagine a fictional field-notes application. Its design MCP knows local theme tokens, component variants, unpublished screen references, and the current branch's navigation conventions. A fuller implementation could offer:

- `read_design_tokens`: read the palette, spacing, and typography from this checkout.
- `find_screen_references`: locate approved screen captures.
- `inspect_component`: explain a component's variants and constraints.
- `propose_screen`: produce a mockup grounded in local assets.

Publishing that application's assets as a public MCP service would not make them useful to strangers. A hosted version would need access to unpublished files and could see a different revision from the branch being edited. A local server can read the checkout directly. It may still call a remote provider for a particular operation; local execution does not make every operation offline.

The runnable [design-studio example](examples/design-studio) is deliberately smaller. One read-only tool returns fictional tokens, with no network calls or company material:

```sh
git clone https://github.com/autobelay/claude-project-scoped-mcp.git
cd claude-project-scoped-mcp/examples/design-studio
claude
```

Ask: “Use the project-design MCP to read this project's design tokens.” The example server is bundled in the repository.

## Project manifest

Claude Code already supports native project `.mcp.json` files. This plugin is for repositories using the portable `mcp.json` convention and a single user-scoped installation, without maintaining another set of server declarations for each client.

Place `mcp.json` at the project root:

```json
{
  "mcpServers": {
    "project-design": {
      "type": "stdio",
      "title": "Project design assistant",
      "description": "Use this checkout's design system and screen references.",
      "command": "node",
      "args": ["tooling/design/server.mjs"],
      "env": { "DESIGN_TOKEN_FILE": "tooling/design/tokens.json" }
    },
    "project-docs": {
      "type": "streamable-http",
      "url": "https://example.com/mcp",
      "headers": { "Authorization": "Bearer ${env:DOCS_TOKEN}" }
    }
  }
}
```

The HTTPS URL is illustrative, not a working service. The project's `mcp.json` belongs to the bridge. The plugin's `plugin.mcp.json` configures Claude Code's connection to the bridge; it has a different role from a native Claude Code project `.mcp.json`. The bridge expands `${env:NAME}` from its environment and does not read `.env` files. Do not commit credentials to a manifest. An interactive login managed by Claude Code is not automatically shared with a remote server behind this bridge; configure authenticated headers explicitly.

Claude Code substitutes `${CLAUDE_PLUGIN_ROOT}` into the path of the bundled bridge and `${CLAUDE_PROJECT_DIR}` into its `--directory` argument. The bridge walks upward from that directory to the nearest `mcp.json`, stopping at a Git checkout root (`.git` directory or worktree file). A nested checkout does not inherit its parent's manifest. A worktree needs its own manifest and server assets. Without a manifest, the bridge exposes only `project_mcp_status`.

Relative command arguments and environment values remain unchanged. Stdio servers run with the manifest's parent directory as `cwd`, so their relative file access follows that checkout. No manifest contents are copied into Claude Code's global MCP configuration. Each session owns its connections, which close when the session ends. Manifest changes take effect in a new session.

## Supported features and limits

- **Model Context Protocol:** uses the [official TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk) for protocol negotiation and JSON-RPC transport. It accepts stdio, Streamable HTTP, and legacy SSE downstream, and presents stdio to Claude Code.
- **Project manifest:** implements a practical `mcp.json` subset from the evolving [Standard Client-Side Configuration Format proposal](https://github.com/modelcontextprotocol/modelcontextprotocol/issues/2633): `mcpServers`, named servers, transport types, commands, arguments, environment, headers, title, and description. This proposal is not a finalized universal standard. Interactive `oauth` configuration is unsupported.
- **Claude Code plugin:** the native `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, and `plugin.mcp.json` package the bridge. The MCP entry launches `node ${CLAUDE_PLUGIN_ROOT}/dist/bridge.cjs --directory ${CLAUDE_PROJECT_DIR}`. Claude Code [documents both substitutions](https://code.claude.com/docs/en/plugins-reference#environment-variables) for plugin MCP configuration. There are no lifecycle hooks.

Tools preserve input and output schemas, behavioral annotations, and structured, image, and text results. Programmatic names include the original server and tool labels plus a stable digest to avoid collisions. Human-readable titles use the manifest's server title or server key and the original tool title or name, without the digest. Manifest descriptions and downstream server instructions accompany tools. Paginated tool lists and tool-list change notifications are supported. Calls forward cancellation and use a five-minute timeout.

Listed resources and prompts are forwarded with namespaced identifiers. Resource templates, subscriptions, completion, sampling, elicitation, interactive OAuth, and experimental task execution are not implemented. The bridge does not advertise those capabilities. A normal tool that returns a job ID and provides polling tools works normally.

Connection attempts are concurrent and bounded to 30 seconds per initialization or list request. A failed server does not hide healthy servers' tools. `project_mcp_status` reports failures without printing raw credentials, child stderr, headers, or transport errors. A disconnected downstream server requires a new session to reconnect. Child stderr is drained and suppressed; diagnose a failing server directly in its checkout. Tool results are forwarded as returned by the server and can contain private data.

Installing the plugin enables commands in discovered project manifests to run. Use it with repositories whose tooling you trust. The bridge does not sandbox downstream servers or override Claude Code's permissions.

## Distribution and development

This repository is an independent, public [Claude Code plugin marketplace](MARKETPLACE.md), not an Anthropic directory listing. Claude Code's [publishing guide](https://code.claude.com/docs/en/plugins/publish#publish-through-your-own-marketplace) documents a single repository containing both a plugin and a marketplace entry whose source is `"./"`.

The bundled executable speaks MCP over stdin and stdout outside the plugin. `--directory PATH`, `--manifest PATH`, and `--only server-a,server-b` are available for explicit embedding and diagnosis. Normal plugin installation supplies `--directory` automatically.

```sh
npm ci
npm run build
npm test
```

The synthetic tests cover discovery, project isolation, tool and resource forwarding, failed sibling servers, authenticated HTTP, and child environments. Built artifacts are committed so Git marketplace installation does not need dependency installation or a build hook. See [local demonstration notes](TESTING.md).

This bridge was adapted from the public [Codex Project-scoped MCP](https://github.com/autobelay/codex-project-scoped-mcp) implementation. Both repositories are MIT licensed. Bundled dependency notices are in `THIRD_PARTY_NOTICES`.
