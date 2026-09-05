# answer.cloud plugins

Official plugins for [answer.cloud](https://answer.cloud) — author AEO-friendly static sites,
define editable CMS collection templates, and deploy to `{slug}.answer.cloud` (or an `ai.`
companion subdomain) from Codex, Claude Code, or ChatGPT.

The plugin is the complete setup. It includes:

- the answer.cloud build and content workflow
- site, deploy, domain, CMS, and GitHub tools
- official answer.cloud style-guide defaults for new unbranded sites and CMS templates
- AI-generated brand-matched chat styles and chat/search widget verification

You do not need to configure a server or download a separate skill.

## 1. Install

### Claude Code

```
/plugin marketplace add answer-cloud/answer.cloud-Plugins
/plugin install answercloud@answercloud
```

Start a new Claude Code session after installation, then ask the Answer Cloud plugin to build,
deploy, or manage your site.

### Codex

```sh
codex plugin marketplace add answer-cloud/answer.cloud-Plugins
codex plugin add answercloud@answercloud
```

Start a new Codex session after installation, then ask the Answer Cloud plugin to build, deploy,
or manage your site.

### ChatGPT

In ChatGPT developer mode, create a custom app with the MCP endpoint
`https://answer.cloud/mcp`, choose OAuth, and scan the tools.

## 2. Connect your account

The first Answer Cloud action opens a secure browser sign-in. Choose the organization the agent
may access and approve the requested read and deploy permissions. There is no API key to copy or
save. Short-lived access refreshes automatically until you disconnect the agent in Answer Cloud.

## Layout

```
.claude-plugin/marketplace.json   # Claude Code marketplace catalog
.agents/plugins/marketplace.json  # Codex marketplace catalog
claude/answercloud/               # Claude Code plugin
codex/answercloud/                # Codex plugin
```
