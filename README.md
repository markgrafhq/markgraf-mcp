# Markgraf local MCP

Markgraf is a local stdio MCP server for creating, validating, and playing bounded Markgraf diagrams. It provides `get_markgraf_guide`, `validate_markgraf`, and `render_markgraf`. Successful rendering returns structured content with `ok`, `source`, and `title`, and declares the bundled player resource at `ui://markgraf/player-0.0.1.html`.

## Local processing and data handling

The host chat supplies Markgraf DSL source and an optional title over stdio. The local server validates and prepares that source, then the bundled local player renders it. Markgraf itself has no telemetry, persistence, outbound network requests, user-file access, AI model calls, or image loading. It does not save diagrams. The chat host still receives the source supplied to its tools and the returned result; the host's own data handling policy therefore applies to those inputs and outputs.

The server rejects image attributes, including URLs, local paths, and data URIs. It also applies bounded complexity limits before layout: source is limited to 16,384 Unicode code points, nesting is limited, and document, frame, operation, node, edge, copy, dimension, and font-size limits are enforced. These restrictions keep the self-contained player from fetching assets or accepting unbounded work.

## Runtime and host requirements

The bundled runtime has no `node_modules`; `server/server.mjs` is a fully bundled Node ES module and `server/player.html` is self-contained. It requires Node.js 22 or newer.

For an MCPB installation, the manifest launches `node`; Claude Desktop supplies the bundled Node runtime used by MCPB. Other generic stdio clients must have Node.js 22 or newer available themselves—this package does not promise or install Node for them. Hosts must support MCP Apps rendering to display the interactive player. A host without Apps rendering can receive the normal MCP tool result but cannot display the bundled player inline.

Claude's local stdio configuration is for Claude Desktop MCPB, Claude Code, and local Cowork sessions that can start a command on the user's computer. It does **not** work in claude.ai web chat, which does not start local MCP commands.

## Install locally

Download the prebuilt packages from the [public distribution releases](https://github.com/markgrafhq/markgraf-mcp/releases/latest).

In Claude Desktop, open **Settings → Desktop app → Extensions → Advanced settings → Install Extension**, then select `markgraf-0.0.1.mcpb`. Older Desktop versions put Extensions directly in Settings. Review the local tool permissions. Claude supplies Node and starts the server; no hosting account, API key, or terminal process is needed.

For Claude Code, unpack `markgraf-plugin-0.0.1.zip` and start a session with `claude --plugin-dir /absolute/path/to/the/unpacked/folder`. Generic clients need Node 22 or newer and can launch `node /absolute/path/to/server/server.mjs` over stdio.

Try these self-contained examples:

- “Draw a static diagram of an API storing data in a database.”
- “Animate a client calling an API and receiving a reply.”
- “Show a sequence diagram of a client sending a request to an API.”

The guide tool supplies the DSL, validation returns source diagnostics, and the rendering tool opens the player where the host supports MCP Apps. Playback has pause, replay, seeking, host-theme support, and reduced-motion handling. Reviewers need no test account, credentials, or external sample data.

## Troubleshooting and support

- **No inline player:** the client must support MCP Apps. Other clients can receive tool results but cannot display the player.
- **Node is unavailable:** generic stdio clients need Node.js 22 or newer on their command path. Claude Desktop's MCPB runtime supplies Node itself.
- **A diagram is rejected:** use `get_markgraf_guide` and `validate_markgraf` to correct the source. Image assets and diagrams exceeding the documented limits are deliberately rejected.

Report product problems through [GitHub Issues](https://github.com/markgrafhq/markgraf-mcp/issues). Report suspected security vulnerabilities privately through [GitHub security advisories](https://github.com/markgrafhq/markgraf-mcp/security/advisories/new), not through a public issue.

## Maintainer build (Markgraf monorepo)

The public distribution repository contains the assembled runtime. These commands are for maintainers working in the Markgraf monorepo, not for installing from the distribution repository. Install the repository's PureScript/Spago prerequisites and its existing sibling dependencies, then:

```sh
bun install --cwd mcp
bun run --cwd mcp test
bun run --cwd mcp package
claude plugin validate mcp/dist/package
```

The build produces `mcp/dist/server.mjs` and a self-contained player with embedded fonts. Packaging produces `mcp/dist/markgraf-0.0.1.mcpb` and `mcp/dist/markgraf-plugin-0.0.1.zip`. Both archives contain the same runtime and all three host manifest formats. Source limits and tool policy live in PureScript; the reusable `purescript-mcp-sdk` dependency contains only official SDK bindings.

## Packaging sources and assembly

The source manifests deliberately share one plugin root for the local Claude and OpenAI formats:

| Source | Assemble at artifact path | Consumer |
| --- | --- | --- |
| `mcp/packaging/claude-mcpb/manifest.json` | `manifest.json` | Claude Desktop MCPB |
| `mcp/packaging/plugin/plugin.json` | `plugin.json` | Portable/OpenAI plugin |
| `mcp/packaging/plugin/.claude-plugin/plugin.json` | `.claude-plugin/plugin.json` | Claude plugin |
| `mcp/packaging/plugin/.mcp.json` | `.mcp.json` | Claude Code/local Cowork local stdio server |
| `mcp/packaging/plugin/mcp.json` | `mcp.json` | OpenAI desktop-only local stdio server |
| generated `mcp/dist/server.mjs` | `server/server.mjs` | All packages |
| generated `mcp/dist/player.html` | `server/player.html` | All packages |
| generated `mcp/dist/logo.svg` | `server/logo.svg` | All packages |

Package the MCPB root as `markgraf-0.0.1.mcpb`, with `manifest.json` at the archive root. Package the shared plugin root as `markgraf-plugin-0.0.1.zip`. Both artifact roots use the runtime paths above. The MCPB manifest uses `${__dirname}/server/server.mjs`; the Claude plugin uses `${CLAUDE_PLUGIN_ROOT}/server/server.mjs`; the OpenAI plugin uses `${PLUGIN_ROOT}/server/server.mjs`. The plugin configurations set their working directory to the respective plugin root.

The MCP integration and its bundled Markgraf runtime are MIT-licensed. Packages include the license and third-party notices, including the bundled font's separate license.

## Installation and validation

After assembly, validate the Claude plugin root with Claude Code's `claude plugin validate` command and validate the MCPB source root with `mcpb validate manifest.json`; `mcpb pack` also validates the manifest while creating the archive. Test the resulting local package in its intended desktop host with Node 22 or newer. The manifest source files are JSON-only configuration; they never point at a remote MCP service, a package launcher, or an installer.

Claude's current plugin build and pre-submission documentation are the primary references for plugin layout, local stdio behavior, local CLI validation, and directory requirements:

- <https://claude.com/docs/plugins/build.md>
- <https://claude.com/docs/plugins/pre-submission-checklist.md>
- <https://raw.githubusercontent.com/anthropics/mcpb/main/MANIFEST.md>

For OpenAI desktop-only plugins, use the current packaging and submission instructions:

- <https://developers.openai.com/plugins/build/plugins.md>
- <https://developers.openai.com/plugins/deploy/desktop-only.md>

The OpenAI desktop-only path requires a submitting organization to have submission access and developer or business identity verification. Local MCP submissions are limited to approved partners: an OpenAI representative must confirm eligibility and allowlist the organization before portal submission and review. Sign in to the submission portal only after those prerequisites are satisfied; package validation does not grant directory approval.

Claude Desktop extensions use the [desktop-extension submission form](https://clau.de/desktop-extention-submission), linked by [Anthropic's local MCP installation guide](https://support.claude.com/en/articles/10949351-getting-started-with-local-mcp-servers-on-claude-desktop). This is separate from the plugin/remote-connector developer portal. A local MCPB can be installed privately while directory review is pending.

Publishing the optional Claude plugin through the [developer portal](https://claude.com/docs/plugins/submit.md) requires an authenticated eligible submitter and a GitHub repository containing the complete assembled plugin folder—not just these source manifests. Local plugin commands run in Claude Code and local Cowork, not claude.ai web chat.
