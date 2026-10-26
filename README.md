# Answer Cloud plugins

Launch an [Answer Cloud](https://answer.cloud) website in Claude Code or Codex with one prompt:

```text
Follow instructions at https://answer.cloud/new to build my website
```

The agent reads the [public launch guide](https://answer.cloud/new), asks only for missing
email, organization name, and a short website brief, then requests your first site. It receives
short-lived access limited to building and publishing that isolated site. It builds with Astro,
publishes to production, and gives you the verified live URL. Successful production publication
triggers the account-setup email job so you can claim the account afterward; sending and
delivery can take longer and are not established by a live URL alone.

**The first launch requires neither browser sign-in nor a marketplace installation.** The agent
can call the site's restricted MCP tools directly over HTTP in the same session. Your account
email identifies where the setup link is delivered; it is not published as a contact unless you
ask. You do not need to copy passwords, API keys, or build tokens into the conversation.

This workflow needs a coding agent with project files, a terminal/HTTP client, Astro build tools,
and artifact upload capabilities, such as Claude Code or Codex. Reading the guide cannot add
those capabilities to every chat client. If a client lacks them, the agent preserves the brief
and source and provides the minimum instruction to resume in a capable environment.

## Optional plugin installation

The plugin is a convenience that bundles the wizard, the site-building skill, and a native MCP
connection for authenticated site management. It is not a prerequisite for the public launch
prompt. Browser OAuth is used for returning account access; temporary first-build access remains
limited to the newly reserved site.

### Claude Code

In a current Claude Code session:

```text
/plugin install answercloud --marketplace answer-cloud/answer.cloud-Plugins
```

The one-command form requires Claude Code v2.1.275 or later. For older versions:

```text
/plugin marketplace add answer-cloud/answer.cloud-Plugins
/plugin install answercloud@answercloud
```

### Codex

Add the `answer-cloud/answer.cloud-Plugins` repository as a marketplace in Codex, then install
Answer Cloud from it. Where the plugin commands are available:

```sh
codex plugin marketplace add answer-cloud/answer.cloud-Plugins
codex plugin add answercloud@answercloud
```

Start a new session after installation if your client needs one to load the optional plugin.
Keep the existing brief, source, and site reservation when resuming.

## Distribution and public listings

The catalogs in this repository are **repository marketplaces**. Installing from this repository
does not establish a listing in either vendor's public directory. The changes here also do not
submit or publish a listing.

- **OpenAI:** submit the plugin's remote MCP endpoint at the
  [OpenAI plugin portal](https://platform.openai.com/plugins), following the
  [submission guide](https://developers.openai.com/plugins/guides/submit-claude-plugin).
  Include the onboarding and site-building skills in the submission. Claude approval does not
  transfer to OpenAI's [public plugin directory](https://developers.openai.com/plugins/build/plugins).
- **Claude:** follow the [plugin publishing guide](https://code.claude.com/docs/en/plugins/publish)
  for repository distribution and public marketplace partnership. Remote connector directory
  submission uses [Claude Directory management](https://claude.ai/directory/manage); that is a
  separate process from an official Claude Code marketplace listing.

MCP-only clients, including ChatGPT custom apps, can connect to `https://app.answer.cloud/mcp` with
OAuth for authenticated account operations. They still need the build and upload capabilities
above to complete the public first-site launch workflow.

## Maintainer layout

```text
.claude-plugin/marketplace.json                   # Claude Code repository catalog
.agents/plugins/marketplace.json                 # Codex repository catalog
claude/answercloud/skills/answer-cloud-onboarding # Claude wizard
codex/answercloud/skills/answer-cloud-onboarding  # Codex wizard + UI metadata
```

Canonical skill content lives in the parent application at `.claude/skills/`. Sync a single
skill with `bin/sync_plugins answer-cloud-onboarding`; omit the name to sync all canonical
skills. The sync refuses to overwrite locally changed generated copies. Reconcile those
changes into their canonical source before a full sync or plugin release. After reconciling,
use `bin/sync_plugins --force <skill-name>` to deliberately replace that skill's generated copies.
