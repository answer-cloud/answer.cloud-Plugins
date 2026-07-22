# answer.cloud plugins

Official plugins for [answer.cloud](https://answer.cloud) — author AEO-friendly static sites,
define editable CMS collection templates, and deploy to `{slug}.answer.cloud` (or an `ai.`
companion subdomain) through the answercloud MCP server.

Each plugin bundles:

- the **answer-cloud-site** skill (Astro authoring rules, CMS template wizard, deploy loop,
  chat/search widget verification)
- the **answercloud MCP server** config (`https://answer.cloud/mcp` — sites, deploys, CMS tools)

## 1. Get an API key

Create a hosting API key in the [answer.cloud dashboard](https://answer.cloud) under
**Settings → API & MCP** (keys start with `aeo_host_`), then export it:

```sh
export ANSWERCLOUD_API_KEY=aeo_host_YOUR_KEY
```

Add that line to your shell profile so new sessions pick it up.

## 2. Install

### Claude Code

```
/plugin marketplace add jfals82/answer.cloud-Plugins
/plugin install answercloud@answercloud
```

The skill is invoked as `/answercloud:answer-cloud-site` (or automatically when you ask to build
or deploy an answer.cloud site), and the `answercloud` MCP tools connect using `ANSWERCLOUD_API_KEY`.

### Codex

```sh
codex plugin marketplace add jfals82/answer.cloud-Plugins
codex plugin add answercloud@answercloud
```

The `answer-cloud-site` skill appears under `/skills` (invoke with `$answer-cloud-site`), and the
`answercloud` MCP server authenticates via `ANSWERCLOUD_API_KEY`.

## Manual setup (no plugin)

- Claude Code:

  ```sh
  claude mcp add --transport http answercloud https://answer.cloud/mcp \
    --header "Authorization: Bearer aeo_host_YOUR_KEY"
  ```

- Codex — add to `~/.codex/config.toml`:

  ```toml
  [mcp_servers.answercloud]
  url = "https://answer.cloud/mcp"
  bearer_token_env_var = "ANSWERCLOUD_API_KEY"
  ```

Then copy `claude/answercloud/skills/answer-cloud-site/` into `~/.claude/skills/` (Claude Code) or
`codex/answercloud/skills/answer-cloud-site/` into `~/.agents/skills/` (Codex).

## Layout

```
.claude-plugin/marketplace.json   # Claude Code marketplace catalog
.agents/plugins/marketplace.json  # Codex marketplace catalog
claude/answercloud/               # Claude Code plugin
codex/answercloud/                # Codex plugin
```
