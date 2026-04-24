# Hello World — ChurchCRM Community Plugin Example

Minimal reference plugin for [ChurchCRM](https://churchcrm.io).

**Fork this repo, rename the plugin id, swap out the logic, and follow
the submission flow below** to get your plugin added to the approved
allowlist.

---

## Table of contents

1. [Quick start](#quick-start)
2. [What this example demonstrates](#what-this-example-demonstrates)
3. [Directory layout](#directory-layout)
4. [AbstractPlugin API](#abstractplugin-api)
5. [Available hooks](#available-hooks)
6. [Adding features: routes, views & settings](#adding-features-routes-views--settings)
7. [Accessing the database](#accessing-the-database)
8. [Localization (PHP + JS)](#localization-php--js)
9. [Real-world core plugins — learn by example](#real-world-core-plugins--learn-by-example)
10. [Publishing your plugin](#publishing-your-plugin)
11. [Documentation & support](#documentation--support)

---

## Quick start

```bash
# From your ChurchCRM checkout — scaffold a new plugin based on this template:
php scripts/create-plugin.php my-plugin --author="Your Name"
```

That command clones this repo into `src/plugins/community/my-plugin/`,
rewrites the plugin id, namespace, and class name, and leaves you with a
runnable starting point.

Alternatively, **fork this repo** on GitHub, clone it, and rename the
identifiers by hand.

---

## What this example demonstrates

- **Minimal `plugin.json`** — a single text setting and one hook declaration.
- **`AbstractPlugin` subclass** — subscribes to `Hooks::PERSON_CREATED`
  in `boot()`.
- **Plugin-local gettext** via `dgettext('hello-world', '…')`. Never call
  plain `gettext()` or `_()` from a plugin — those go to the core `messages`
  textdomain and your strings will not resolve.
- **Plugin-local i18next JSON** at `locale/i18n/en_US.json` — a flat
  `key → string` map that the frontend reads from
  `window.CRM.plugins['hello-world'].i18n`.
- **Sandboxed config access** via `$this->getConfigValue()` /
  `$this->setConfigValue()`. The base class enforces that plugins can only
  read or write `plugin.hello-world.*` keys.

---

## Directory layout

```
community-plugin-hello-world/
├── plugin.json                    # required manifest
├── src/
│   └── HelloWorldPlugin.php       # required: extends AbstractPlugin
└── locale/
    └── i18n/
        └── en_US.json             # JS i18next strings (optional)
```

A full-featured plugin would additionally ship:

```
├── routes/
│   └── routes.php                 # Slim routes under /plugins/{id}/…
├── views/
│   └── settings.php               # Twig or plain-PHP templates
├── locale/
│   ├── i18n/
│   │   └── en_US.json             # JS translations
│   └── textdomain/
│       └── en_US/LC_MESSAGES/
│           └── hello-world.mo     # compiled PHP gettext
└── help.json                      # user-facing help content for the settings modal
```

---

## AbstractPlugin API

These are the methods your plugin class can use or override:

| Method | Description |
|--------|-------------|
| `boot(): void` | Called when the plugin is loaded. Register hooks here. |
| `activate(): void` | Called once when the admin enables the plugin. Run first-time setup here. |
| `deactivate(): void` | Called when the admin disables the plugin. |
| `uninstall(): void` | Called when the plugin is uninstalled. Clean up DB rows / config here. |
| `getConfigValue(string $key): string` | Read a plugin-owned config key. Key must exist in `plugin.json settings[].key`. |
| `getBooleanConfigValue(string $key): bool` | Read a boolean config value. |
| `setConfigValue(string $key, string $value): void` | Write a plugin-owned config key. |
| `getHeadContent(): string` | Return raw HTML injected into `<head>` on every page. |
| `getFooterContent(): string` | Return raw HTML injected before `</body>` on every page. |
| `getMenuItems(): array` | Return extra nav-menu entries. |
| `getClientConfig(): array` | Return a JSON-serialisable array exposed as `window.CRM.plugins[id].config`. |
| `isConfigured(): bool` | Override to add extra "is this plugin ready?" logic. |
| `getConfigurationError(): ?string` | Return a human-readable error string if not configured. |
| `log(string $msg, string $level, array $ctx): void` | Write to the ChurchCRM app log. Levels: `debug`, `info`, `warning`, `error`. |

---

## Available hooks

Declare hooks your plugin uses in `plugin.json → hooks[]`. Hooks fall into
two categories:

- **Action hooks** — the plugin reacts to an event. Return value is ignored.
- **Filter hooks** — the plugin can modify the data. The callback must
  return the (modified) value.

### People

| Constant | String | Type | Receives |
|----------|--------|------|---------|
| `Hooks::PERSON_PRE_CREATE` | `person.pre_create` | Filter | `array $personData` → return modified array |
| `Hooks::PERSON_CREATED` | `person.created` | Action | `Person $person` |
| `Hooks::PERSON_PRE_UPDATE` | `person.pre_update` | Filter | `Person $person, array $changes` → return modified `$changes` |
| `Hooks::PERSON_UPDATED` | `person.updated` | Action | `Person $person, array $oldData` |
| `Hooks::PERSON_DELETED` | `person.deleted` | Action | `int $personId, array $personData` |
| `Hooks::PERSON_VIEW_TABS` | `person.view.tabs` | Filter | `array $tabs, Person $person` → return modified `$tabs` |

### Families

| Constant | String | Type | Receives |
|----------|--------|------|---------|
| `Hooks::FAMILY_PRE_CREATE` | `family.pre_create` | Filter | `array $familyData` |
| `Hooks::FAMILY_CREATED` | `family.created` | Action | `Family $family` |
| `Hooks::FAMILY_PRE_UPDATE` | `family.pre_update` | Filter | `Family $family, array $changes` |
| `Hooks::FAMILY_UPDATED` | `family.updated` | Action | `Family $family, array $oldData` |
| `Hooks::FAMILY_DELETED` | `family.deleted` | Action | `int $familyId, array $familyData` |
| `Hooks::FAMILY_VIEW_TABS` | `family.view.tabs` | Filter | `array $tabs, Family $family` |

### Finance

| Constant | String | Type | Receives |
|----------|--------|------|---------|
| `Hooks::DONATION_RECEIVED` | `donation.received` | Action | `Pledge $pledge` |
| `Hooks::DEPOSIT_CLOSED` | `deposit.closed` | Action | `Deposit $deposit` |

### Events

| Constant | String | Type | Receives |
|----------|--------|------|---------|
| `Hooks::EVENT_CREATED` | `event.created` | Action | `Event $event` |
| `Hooks::EVENT_CHECKIN` | `event.checkin` | Action | `EventAttend $attendance, Event $event, Person $person` |
| `Hooks::EVENT_CHECKOUT` | `event.checkout` | Action | `EventAttend $attendance, Event $event, Person $person` |
| `Hooks::SYSTEM_CALENDARS_REGISTER` | `systemcalendars.register` | Filter | `SystemCalendar[] $calendars` → return modified array |

### Groups

| Constant | String | Type | Receives |
|----------|--------|------|---------|
| `Hooks::GROUP_MEMBER_ADDED` | `group.member.added` | Action | `Person2group2roleP2g2r $membership, Group $group, Person $person` |
| `Hooks::GROUP_MEMBER_REMOVED` | `group.member.removed` | Action | `int $personId, Group $group` |

### Email

| Constant | String | Type | Receives |
|----------|--------|------|---------|
| `Hooks::EMAIL_PRE_SEND` | `email.pre_send` | Filter | `array $emailData` → return modified array |
| `Hooks::EMAIL_SENT` | `email.sent` | Action | `array $emailData, bool $success` |

### UI / Menu

| Constant | String | Type | Receives |
|----------|--------|------|---------|
| `Hooks::MENU_BUILDING` | `menu.building` | Filter | `array $menuItems` → return modified array |
| `Hooks::DASHBOARD_WIDGETS` | `dashboard.widgets` | Filter | `array $widgets` → return modified array |
| `Hooks::SETTINGS_PANELS` | `settings.panels` | Filter | `array $panels` → return modified array |
| `Hooks::ADMIN_PAGE` | `admin.page` | Action | `string $pageId` |

### Reports / System

| Constant | String | Type | Receives |
|----------|--------|------|---------|
| `Hooks::REPORT_PRE_GENERATE` | `report.pre_generate` | Filter | `array $reportData, string $reportType` |
| `Hooks::REPORT_TYPES` | `report.types` | Filter | `array $reportTypes` |
| `Hooks::SYSTEM_INIT` | `system.init` | Action | _(none)_ |
| `Hooks::SYSTEM_UPGRADED` | `system.upgraded` | Action | `string $fromVersion, string $toVersion` |
| `Hooks::CRON_RUN` | `cron.run` | Action | _(none)_ |
| `Hooks::API_RESPONSE` | `api.response` | Filter | `array $response, string $endpoint` |

### Registering a hook in `boot()`

```php
public function boot(): void
{
    HookManager::on(Hooks::PERSON_CREATED, function (Person $person): void {
        $greeting = $this->getConfigValue('greeting');
        $this->log("New person: {$person->getFullName()} — {$greeting}", 'info');
    });
}
```

### Adding calendar events via `systemcalendars.register`

Use the `SYSTEM_CALENDARS_REGISTER` filter to contribute a calendar that appears
in the ChurchCRM sidebar alongside Birthdays and Anniversaries. Your class must
implement `ChurchCRM\SystemCalendars\SystemCalendar`:

```php
use ChurchCRM\Plugin\Hook\HookManager;
use ChurchCRM\Plugin\Hooks;
use ChurchCRM\SystemCalendars\SystemCalendar;
use ChurchCRM\model\ChurchCRM\Event;
use Propel\Runtime\Collection\ObjectCollection;

// In your plugin class:
public function boot(): void
{
    HookManager::addFilter(Hooks::SYSTEM_CALENDARS_REGISTER, [$this, 'registerCalendars']);
}

public function registerCalendars(array $calendars): array
{
    $calendars[] = new MyPluginCalendar($this);
    return $calendars;
}

// Separate class implementing the interface:
class MyPluginCalendar implements SystemCalendar
{
    public function getId(): int          { return 9001; }  // pick a stable unique int
    public function getName(): string     { return 'My Plugin Events'; }
    public function getAccessToken(): bool { return true; } // true = visible to all users
    public function getForegroundColor(): string { return '#ffffff'; }
    public function getBackgroundColor(): string { return '#3b82f6'; }
    public static function isAvailable(): bool   { return true; }

    public function getEvents(string $start, string $end): ObjectCollection
    {
        // Return an ObjectCollection of Event objects for the date range.
        // $start and $end are ISO-8601 strings (e.g. "2026-01-01T00:00:00").
        $collection = new ObjectCollection();
        // … build Event objects and append to $collection …
        return $collection;
    }

    public function getEventById(int $id): ObjectCollection
    {
        // Return a single-item ObjectCollection for the given event id.
        return new ObjectCollection();
    }
}
```

Declare the capability in `approved-plugins.json` when submitting:
```json
"permissions": ["calendar.register"]
```

See [`src/plugins/core/holidays/`](https://github.com/ChurchCRM/CRM/tree/master/src/plugins/core/holidays)
for a full working implementation.

---

## Adding features: routes, views & settings

### Routes

Create `routes/routes.php`. ChurchCRM automatically includes it when the
plugin is loaded. Routes are mounted under `/plugins/{id}/`:

```php
<?php
// routes/routes.php

use Slim\App;
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;

return function (App $app): void {
    $app->get('/plugins/hello-world/settings', function (Request $req, Response $res): Response {
        ob_start();
        require __DIR__ . '/../views/settings.php';
        $res->getBody()->write(ob_get_clean());
        return $res;
    });
};
```

### Views

Plain PHP templates live under `views/`. Include ChurchCRM's header/footer
to inherit the Tabler shell:

```php
<?php
// views/settings.php
require_once __DIR__ . '/../../../../Include/Header.php';
?>
<div class="container-xl">
    <h2><?= dgettext('hello-world', 'Hello World Settings') ?></h2>
</div>
<?php require_once __DIR__ . '/../../../../Include/Footer.php'; ?>
```

### Settings

Declare settings in `plugin.json`. They appear automatically in the
Admin → Plugins settings modal:

```json
"settings": [
    { "key": "greeting", "label": "Greeting", "type": "text", "required": false },
    { "key": "enabled",  "label": "Enable feature", "type": "boolean", "required": false }
]
```

Read them in PHP:

```php
$greeting = $this->getConfigValue('greeting');
$isEnabled = $this->getBooleanConfigValue('enabled');
```

---

## Accessing the database

ChurchCRM uses **Propel ORM** — always use it instead of raw SQL.

```php
use ChurchCRM\model\ChurchCRM\PersonQuery;
use ChurchCRM\model\ChurchCRM\FamilyQuery;
use ChurchCRM\model\ChurchCRM\GroupQuery;

// Fetch a person by PK — always cast to (int)
$person = PersonQuery::create()->findPk((int) $personId);

// Query with conditions
$members = PersonQuery::create()
    ->filterByFamId((int) $familyId)
    ->orderByLastName()
    ->find();

// Iterate results
foreach ($members as $person) {
    echo $person->getFullName();
}
```

**Rules:**
- Always cast dynamic IDs: `(int) $id`
- Guard `findOne()` / `findPk()` results against `null` before calling methods
- Never use raw `RunQuery()` or `mysqli_*` calls
- DDL migrations (ALTER TABLE) go in a `.sql` file registered in `upgrade.json`
  — do not run DDL from plugin PHP code

---

## Localization (PHP + JS)

### PHP — gettext

```php
// Always use dgettext with your plugin id, never plain _() or gettext()
$label = dgettext('hello-world', 'Welcome to Hello World');
```

Ship compiled `.mo` files at:
`locale/textdomain/{locale}/LC_MESSAGES/hello-world.mo`

Build them with:

```bash
xgettext --keyword=dgettext:2 --from-code=UTF-8 -o locale/hello-world.pot src/*.php
msgfmt locale/textdomain/en_US/LC_MESSAGES/hello-world.po \
       -o locale/textdomain/en_US/LC_MESSAGES/hello-world.mo
```

### JS — i18next

Ship a flat `key → string` JSON at `locale/i18n/en_US.json`:

```json
{ "helloworld.greeting": "Hello!", "helloworld.settings.title": "Settings" }
```

Read in JavaScript:

```js
const label = window.CRM.plugins['hello-world'].i18n['helloworld.greeting'];
```

Full reference: <https://docs.churchcrm.io/administration/plugins/plugin-localization>

---

## Real-world core plugins — learn by example

The built-in plugins under `src/plugins/core/` each demonstrate a
different pattern:

| Plugin | Pattern to study |
|--------|-----------------|
| `custom-links` | `menu.building` filter hook — adding nav items |
| `holidays` | `systemcalendars.register` filter hook — contributing system calendars |
| `mailchimp` | Full lifecycle: `boot()`, hook registration, external API calls, `activate()`/`deactivate()`/`uninstall()` |
| `google-analytics` | `getHeadContent()` / `getFooterContent()` — injecting scripts on every page |
| `external-backup` | `cron.run` action hook — scheduled background work, WebDAV integration |
| `gravatar` | `getConfigValue()` + `isConfigured()` — simple API-key pattern |
| `vonage` | Multi-setting plugin, `testWithSettings()` — lets admins verify credentials before saving |
| `openlp` | REST API integration, `getClientConfig()` — exposing config to the frontend JS |

---

## Publishing your plugin

1. Tag a release on GitHub with a zip artifact
   (GitHub releases are ideal — immutable URLs, no CDN required):

   ```bash
   git tag v1.0.0 && git push origin v1.0.0
   # then create the release + attach the zip via GitHub UI or gh CLI
   ```

2. Compute the SHA-256 of the zip:

   ```bash
   sha256sum my-plugin-1.0.0.zip
   ```

3. Open a PR against [ChurchCRM/CRM](https://github.com/ChurchCRM/CRM)
   adding your entry to `src/plugins/approved-plugins.json`:

   ```json
   {
     "id": "my-plugin",
     "name": "My Plugin",
     "description": "What it does.",
     "version": "1.0.0",
     "downloadUrl": "https://github.com/you/my-plugin/releases/download/v1.0.0/my-plugin-1.0.0.zip",
     "sha256": "<the hash from step 2>",
     "authorUrl": "https://github.com/you/my-plugin"
   }
   ```

4. A maintainer will run the
   [security & compliance scan](https://docs.churchcrm.io/administration/plugins/plugin-security-and-compliance)
   against your zip. The checklist is public — run it against yourself
   before opening the PR.

5. Once approved, your plugin appears in **Admin → Plugins → Browse** for
   all ChurchCRM installations.

---

## Documentation & support

| Resource | Description |
|----------|-------------|
| [Plugin overview](https://docs.churchcrm.io/administration/plugins/) | Architecture, plugin types, how discovery works |
| [Installing community plugins](https://docs.churchcrm.io/administration/plugins/installing-community-plugins) | Admin walkthrough: browse, install from URL, manage |
| [Security & compliance](https://docs.churchcrm.io/administration/plugins/plugin-security-and-compliance) | The scan checklist every approved plugin must pass |
| [Localization guide](https://docs.churchcrm.io/administration/plugins/plugin-localization) | PHP gettext + JS i18next, `.mo` build steps, locale layout |
| [ChurchCRM/CRM](https://github.com/ChurchCRM/CRM) | Core repo — browse `src/plugins/core/` for real examples |
| [Community Discord](https://discord.gg/churchcrm) | Ask questions, share what you're building, get reviewer feedback |
