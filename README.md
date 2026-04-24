# Hello World — ChurchCRM Community Plugin Example

Minimal reference plugin for [ChurchCRM](https://churchcrm.io) community plugin authors.

Fork this repo, rename the plugin id, swap out the logic, and follow the submission flow below to get your plugin added to the approved allowlist.

---

## What this example demonstrates

- **Minimal `plugin.json`** with a single text setting and one hook.
- **`AbstractPlugin` subclass** that subscribes to `Hooks::PERSON_CREATED` in `boot()`.
- **Plugin-local gettext** via `dgettext('hello-world', '…')`. Never call plain `gettext()` or `_()` from a plugin — those go to the core `messages` textdomain and your strings will not resolve.
- **Plugin-local i18next JSON** at `locale/i18n/en_US.json` — a flat `key → string` map the frontend reads from `window.CRM.plugins['hello-world'].i18n`.
- **Sandboxed config access** through `$this->getConfigValue()` / `$this->setConfigValue()`. The base class enforces that plugins can only read or write `plugin.hello-world.*` keys.

---

## Directory layout

```
community-plugin-hello-world/
├── plugin.json
├── src/
│   └── HelloWorldPlugin.php
└── locale/
    └── i18n/
        └── en_US.json
```

A real plugin would additionally ship:

- `routes/routes.php` — Slim routes under `/plugins/{id}/...`
- `views/*.php` — view templates
- `locale/textdomain/{locale}/LC_MESSAGES/{id}.mo` — compiled gettext files
- `help.json` — user-facing help content rendered in the settings modal

---

## Scaffold a new plugin from this template

From your ChurchCRM checkout:

```bash
php scripts/create-plugin.php my-plugin --author="Your Name"
```

This clones the latest template from this repo into `src/plugins/community/my-plugin/`, rewrites the plugin id, namespace, and class name, and leaves you with a runnable starting point.

---

## Submission flow

1. Build your plugin and tag a release with an immutable zip artifact (GitHub release assets are ideal).
2. Compute the SHA-256 of the zip: `sha256sum my-plugin-1.0.0.zip`
3. Open a PR against [`ChurchCRM/CRM`](https://github.com/ChurchCRM/CRM) that adds your entry to `src/plugins/approved-plugins.json`.
4. A maintainer will run the [`plugin-security-scan.md`](https://github.com/ChurchCRM/CRM/blob/main/.agents/skills/churchcrm/plugin-security-scan.md) checklist against your zip.

---

## Related documentation

- [`plugin-development.md`](https://github.com/ChurchCRM/CRM/blob/main/.agents/skills/churchcrm/plugin-development.md) — full technical reference and the allow/forbid capability contract.
- [`plugin-create.md`](https://github.com/ChurchCRM/CRM/blob/main/.agents/skills/churchcrm/plugin-create.md) — community plugin quickstart + submission flow.
- [`plugin-security-scan.md`](https://github.com/ChurchCRM/CRM/blob/main/.agents/skills/churchcrm/plugin-security-scan.md) — the review checklist every approved plugin must pass.
- [Installing Community Plugins](https://docs.churchcrm.io/administration/plugins/installing-community-plugins) — end-user install walkthrough.
