# Caeros Plugins

Curated plugin marketplace for [Caeros](https://caeros.com). Plugins live under
`plugins/`, one directory each, with a `.caeros-plugin/plugin.json` manifest.

Clients install via the in-app Plugins store (hosted registry) or directly:

```sh
cae plugin add caeros-official https://github.com/Hamza-gth/caeros-plugins.git
cae plugin install caeros-official <name>
```

## Publishing changes

1. Edit/add plugins under `plugins/` (validate with `cae plugin validate <dir>`).
2. Regenerate the hosted registry index from the caeros repo:

   ```sh
   go run ./cmd/genregistry -dir <this-repo> -marketplace caeros-official \
     -source https://github.com/Hamza-gth/caeros-plugins.git -o registry-index.json
   ```

3. Update the `caeros-registry-index` secret and the gateway picks it up:

   ```sh
   gcloud secrets versions add caeros-registry-index --data-file=registry-index.json \
     --project agile-stratum-497123-p5
   ```

Installed clients get changes via `cae plugin update` / the Update button
(shallow git re-fetch + content-digest comparison).
