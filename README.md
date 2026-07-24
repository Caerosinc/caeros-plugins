# Caeros Plugins

This repository is the curated plugin marketplace for [Caeros](https://caeros.com). Each directory under [`plugins/`](plugins/) is an installable plugin with a manifest and its bundled capabilities.

## Available plugins

| Plugin | Description | Capabilities |
| --- | --- | --- |
| [Gmail](plugins/gmail/) | Read, search, triage, draft, and send email through a connected Google account. | Skill, connected app |
| [Spreadsheets](plugins/spreadsheets/) | Create, edit, analyze, verify, and export XLSX and CSV files. | Skill |
| [X](plugins/x/) | Search posts, timelines, trends, news, and bookmarks through X's official MCP server. | Skills, MCP server |
| [Hello Caeros](plugins/hello-caeros/) | Minimal reference plugin demonstrating a bundled greeting skill. | Skill |

## Install a plugin

Install plugins from the in-app Plugins store, or add this repository as a source with the Caeros CLI:

```sh
cae plugin add caeros-official https://github.com/Caerosinc/caeros-plugins.git
cae plugin install caeros-official <name>
```

Replace `<name>` with a plugin directory name such as `gmail`, `spreadsheets`, `x`, or `hello-caeros`.

Update an installed plugin from the in-app **Update** button or the CLI:

```sh
cae plugin update
```

The update process performs a shallow Git re-fetch and compares content digests.

## Plugin structure

Each plugin lives in its own directory under `plugins/`:

```text
plugins/<name>/
├── .caeros-plugin/
│   └── plugin.json
├── skills/          # Optional bundled skills
├── apps/            # Optional connected app definitions
├── commands/        # Optional commands
├── assets/          # Optional interface assets
└── .mcp.json        # Optional MCP server definitions
```

The required `.caeros-plugin/plugin.json` manifest identifies the plugin and declares its capability directories, runtime, permissions, and marketplace metadata. See [`plugins/hello-caeros`](plugins/hello-caeros/) for a minimal example.

## Contributing a plugin

1. Create a directory at `plugins/<name>/`.
2. Add `.caeros-plugin/plugin.json` and declare each bundled capability.
3. Add the corresponding capability directories and files.
4. Validate the plugin:

   ```sh
   cae plugin validate plugins/<name>
   ```

5. Install it locally for an end-to-end test:

   ```sh
   cae plugin install-local plugins/<name>
   ```

6. Open a pull request with the plugin files and any relevant README updates.

Keep permissions narrow, document required account connections, and include prompts that demonstrate the plugin's main workflows.
