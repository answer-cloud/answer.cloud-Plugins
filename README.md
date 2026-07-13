# aeopress plugins

Official plugins for [aeo.press](https://aeo.press) — author AEO-friendly static sites, define
editable CMS collection templates, and deploy to `{slug}.aeo.press` (or an `ai.` companion
subdomain) through the aeopress MCP server.

Each plugin bundles:

- the **aeo-press-site** skill (Astro authoring rules, CMS template wizard, deploy loop)
- the **aeopress MCP server** config (`https://aeo.press/mcp` — sites, deploys, CMS tools)

## 1. Get an API key

Create a hosting API key in the [aeo.press dashboard](https://aeo.press) under
**Settings → API & MCP** (keys start with `aeo_host_`), then export it:

```sh
export AEOPRESS_API_KEY=aeo_host_YOUR_KEY
```

Add that line to your shell profile so new sessions pick it up.

## 2. Install

### Claude Code

```
/plugin marketplace add jfals82/aeopress-plugins
/plugin install aeopress@aeopress
```

The skill is invoked as `/aeopress:aeo-press-site` (or automatically when you ask to build or
deploy an aeo.press site), and the `aeopress` MCP tools connect using `AEOPRESS_API_KEY`.

### Codex

```sh
codex plugin marketplace add jfals82/aeopress-plugins
codex plugin add aeopress
```

The `aeo-press-site` skill appears under `/skills` (invoke with `$aeo-press-site`), and the
`aeopress` MCP server authenticates via `AEOPRESS_API_KEY`.

## Manual setup (no plugin)

- Claude Code:

  ```sh
  claude mcp add --transport http aeopress https://aeo.press/mcp \
    --header "Authorization: Bearer aeo_host_YOUR_KEY"
  ```

- Codex — add to `~/.codex/config.toml`:

  ```toml
  [mcp_servers.aeopress]
  url = "https://aeo.press/mcp"
  bearer_token_env_var = "AEOPRESS_API_KEY"
  ```

Then copy `claude/aeopress/skills/aeo-press-site/` into `~/.claude/skills/` (Claude Code) or
`codex/aeopress/skills/aeo-press-site/` into `~/.agents/skills/` (Codex).

## Layout

```
.claude-plugin/marketplace.json   # Claude Code marketplace catalog
.agents/plugins/marketplace.json  # Codex marketplace catalog
claude/aeopress/                  # Claude Code plugin
codex/aeopress/                   # Codex plugin
```
