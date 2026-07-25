# answer.cloud plugins

Official plugins for [answer.cloud](https://answer.cloud) — author AEO-friendly static sites,
define editable CMS collection templates, and deploy to `{slug}.answer.cloud` (or an `ai.`
companion subdomain) from Codex or Claude Code.

The plugin is the complete setup. It includes:

- the answer.cloud build and content workflow
- site, deploy, domain, CMS, and GitHub tools
- AI-generated brand-matched chat styles and chat/search widget verification

You do not need to configure a server or download a separate skill.

## 1. Get an API key

Create a hosting API key in the [answer.cloud dashboard](https://answer.cloud) under
**Plugin & API keys** (keys start with `aeo_host_`), then export it:

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

Start a new Claude Code session after installation, then ask the Answer Cloud plugin to build,
deploy, or manage your site.

### Codex

```sh
codex plugin marketplace add jfals82/answer.cloud-Plugins
codex plugin add answercloud@answercloud
```

Start a new Codex session after installation, then ask the Answer Cloud plugin to build, deploy,
or manage your site.

## Layout

```
.claude-plugin/marketplace.json   # Claude Code marketplace catalog
.agents/plugins/marketplace.json  # Codex marketplace catalog
claude/answercloud/               # Claude Code plugin
codex/answercloud/                # Codex plugin
```
