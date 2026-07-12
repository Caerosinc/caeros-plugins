# Caeros Plugins

Curated plugin marketplace for [Caeros](https://caeros.com). Plugins live under
`plugins/`, one directory each, with a `.caeros-plugin/plugin.json` manifest.

Clients install via the in-app Plugins store (hosted registry) or directly:

```sh
cae plugin add caeros-official https://github.com/Caerosinc/caeros-plugins.git
cae plugin install caeros-official <name>
```

Installed clients get changes via `cae plugin update` / the Update button
(shallow git re-fetch + content-digest comparison).

## Contributing a plugin

Each plugin is one directory under `plugins/` with a
`.caeros-plugin/plugin.json` manifest plus capability dirs (`skills/`, `apps/`,
`commands/`, …). Validate with `cae plugin validate <dir>` and test locally
with `cae plugin install-local <dir>` before opening a PR.
