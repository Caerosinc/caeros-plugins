---
name: new-plugin
description: Scaffold a new Caeros plugin in the current repo from a one-line description.
aliases: create-plugin
---

You are scaffolding a new Caeros plugin in the repository currently open. The user's arguments (if any) are a one-line description of what the plugin should do. Follow the create-caeros-plugin skill from this plugin as the authoritative reference for structure and schema; read it before generating anything.

## Step 1: Gather the essentials

If the user did not provide them, ask (in one batch, not a drip-feed) for:

1. **Plugin name**: lowercase, hyphenated (e.g. `sql-review`). Propose one derived from their description.
2. **Category**: propose the best fit (e.g. "Developer Tools", "Automations", "Productivity") and let them override.
3. **One-line description** if their arguments were empty.

Sensible defaults you may assume without asking: version `0.1.0`, runtime `local`, empty permissions, developerName from their git config (`git config user.name`), brandColor `#1b365d`.

## Step 2: Choose the location

- If the repo is empty or clearly dedicated to this plugin, scaffold at the repo root.
- Otherwise scaffold under `plugins/<name>/` (creating it), so the repo can host multiple plugins.
- Never overwrite an existing plugin directory; if the name collides, stop and ask.

## Step 3: Generate the scaffold

Create exactly these files:

1. `.caeros-plugin/plugin.json`: full manifest per the create-caeros-plugin skill: name, version, description, keywords derived from the description, capabilities `{"skills": "./skills", "assets": "./assets"}` (add `"commands": "./commands"` only if you also scaffold a command), empty permissions `{}`, runtime `local`, and a complete `interface` block (displayName, shortDescription, longDescription, developerName, category, capabilities, one or two defaultPrompt entries, brandColor, logo and composerIcon pointing at `./assets/logo.svg`).
2. `skills/<main-skill>/SKILL.md`: one starter skill named after the plugin's core job, with frontmatter `name` and `description` (the description must state concretely when the agent should use it) and a body containing a real, useful first draft of the skill's guidance based on the user's description, not lorem ipsum.
3. `assets/logo.svg`: a simple, original geometric mark: a rounded square background (`rx` about 22 percent of the width) in the brand color with a plain geometric glyph (circle, arcs, chevrons, or letterform-free abstract shape) in a contrasting tint. Never copy or imitate an existing company's logo or trademark.

## Step 4: Validate and hand off

1. Run `python3 -m json.tool` on the generated plugin.json and fix any errors before finishing.
2. Confirm the skill frontmatter parses (starts with `---`, has `name` and `description`).
3. Print the file tree of what you created (`find <plugin-dir> -type f`).
4. Tell the user how to try it (install the repo as a local plugin in Caeros) and point them to the publish-caeros-plugin skill for distribution.

Do not commit, push, or open a PR unless the user explicitly asks.
