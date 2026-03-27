---
name: developing-moodle
description: "Moodle plugin development and administration for PHP-based learning management system. Use when working with Moodle installations, creating or modifying plugins (local, block, mod, theme), editing db/install.xml or db/upgrade.php, writing Moodle-compliant PHP code, using Moodle CLI tools, bumping Moodle plugin version, or integrating with Moodle APIs. Triggers on: (1) Plugin development in /local/, /blocks/, /mod/, (2) Moodle database operations, (3) Moodle coding standards compliance, (4) Moodle CLI administration, (5) version.php changes or bump Moodle version."
---

# Moodle Plugin Development

## Quick Reference

| Task | Approach |
|------|----------|
| Purge caches | `sudo php admin/cli/purge_caches.php` |
| Run upgrade | `sudo php admin/cli/upgrade.php --non-interactive` |
| Check schema | `sudo php admin/cli/check_database_schema.php` |
| Uninstall plugin | `sudo php admin/cli/uninstall_plugins.php --plugins=<name> --run` |
| List contrib plugins | `sudo php admin/cli/uninstall_plugins.php --show-contrib` |

## Directory Structure

```
/var/www/moodle/
├── admin/cli/          # CLI tools
├── blocks/             # Block plugins
├── local/              # Local plugins (custom functionality)
├── mod/                # Activity modules
├── theme/              # Themes
├── course/             # Course management
└── user/               # User management

/var/www/moodledata/    # Private file storage
```

## Plugin Anatomy

Every plugin requires these core files:

```
local/pluginname/
├── version.php         # Required: version, dependencies
├── db/
│   ├── install.xml     # Database schema (XMLDB format)
│   ├── upgrade.php     # Schema migrations
│   ├── access.php      # Capability definitions
│   └── services.php    # Web service definitions
├── classes/            # Autoloaded classes (PSR-4 style)
├── lang/en/            # Language strings
│   └── local_pluginname.php
├── lib.php             # Plugin API hooks
├── settings.php        # Admin settings
└── index.php           # Main entry point
```

### version.php Template

```php
<?php
defined('MOODLE_INTERNAL') || die();

$plugin->component = 'local_pluginname';  // Frankenstyle name
$plugin->version = 2025020100;            // YYYYMMDDXX format
$plugin->requires = 2022112800;           // Minimum Moodle version
$plugin->maturity = MATURITY_STABLE;
$plugin->release = '1.0.0';

// Dependencies (optional)
$plugin->dependencies = [
    'local_samms_api' => 2025010100,
];
```

**CRITICAL:** Always increment version after code changes. Update dependency versions when dependent plugins change.

Do the following to bump a version:

- Run `git status` to look for changes since the last time a version number was changed in a plugin. Look at unstaged files and committed files.
- If the plugin has one or more dependencies in the `version.php`, find the dependency, note its version number, then update the dependency versions if necessary. 
- If a plugin has a dependency mentioned in the version file, confirm the dependency version numbers are correct.
- Update the changelog in README if the change involves end-user functionality. Ignore minor tweaks and layout changes.

### Database Schema (db/install.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<XMLDB PATH="local/pluginname/db" VERSION="2025020100">
  <TABLES>
    <TABLE NAME="local_pluginname_records" COMMENT="Store records">
      <FIELDS>
        <FIELD NAME="id" TYPE="int" LENGTH="10" NOTNULL="true" SEQUENCE="true"/>
        <FIELD NAME="userid" TYPE="int" LENGTH="10" NOTNULL="true"/>
        <FIELD NAME="timecreated" TYPE="int" LENGTH="10" NOTNULL="true"/>
        <FIELD NAME="timemodified" TYPE="int" LENGTH="10" NOTNULL="true"/>
      </FIELDS>
      <KEYS>
        <KEY NAME="primary" TYPE="primary" FIELDS="id"/>
        <KEY NAME="userid" TYPE="foreign" FIELDS="userid" REFTABLE="user" REFFIELDS="id"/>
      </KEYS>
      <INDEXES>
        <INDEX NAME="timecreated" UNIQUE="false" FIELDS="timecreated"/>
      </INDEXES>
    </TABLE>
  </TABLES>
</XMLDB>
```

### Database Upgrade (db/upgrade.php)

```php
<?php
defined('MOODLE_INTERNAL') || die();

function xmldb_local_pluginname_upgrade($oldversion) {
    global $DB;
    $dbman = $DB->get_manager();

    if ($oldversion < 2025020200) {
        // Define new field
        $table = new xmldb_table('local_pluginname_records');
        $field = new xmldb_field('status', XMLDB_TYPE_INTEGER, '2', null, 
            XMLDB_NOTNULL, null, '0', 'timemodified');

        if (!$dbman->field_exists($table, $field)) {
            $dbman->add_field($table, $field);
        }

        upgrade_plugin_savepoint(true, 2025020200, 'local', 'pluginname');
    }

    return true;
}
```

## Coding Standards

Follow Moodle's official coding style (https://moodledev.io/general/development/policies/codingstyle):

### Formatting
- **Indentation**: 4 spaces (never tabs)
- **Line length**: Maximum 132 characters
- **Line endings**: Unix LF only
- **PHP tags**: Long tags `<?php` only (never short tags)
- **No trailing whitespace**

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Files | lowercase, descriptive | `application_form.php` |
| Classes | lowercase with underscores | `class application_form` |
| Functions | Frankenstyle prefix | `local_pluginname_get_data()` |
| Variables | lowercase, concise | `$applicationdata` |
| Constants | UPPERCASE, Frankenstyle | `LOCAL_PLUGINNAME_STATUS_PENDING` |

### PHPDoc Requirements

```php
<?php
/**
 * Brief description of the file.
 *
 * @package    local_pluginname
 * @copyright  2025 Your Name <email@example.com>
 * @license    http://www.gnu.org/copyleft/gpl.html GNU GPL v3 or later
 */

namespace local_pluginname;

defined('MOODLE_INTERNAL') || die();

/**
 * Class description.
 */
class myclass {

    /**
     * Method description.
     *
     * @param int $userid The user ID.
     * @param string $status The status value.
     * @return bool True on success.
     */
    public function process_record(int $userid, string $status): bool {
        // Implementation
    }
}
```

### Strings
- Single quotes for literals: `$text = 'Hello';`
- Double quotes for interpolation: `$msg = "User $id";`
- Concatenation with spaces: `$full = $first . ' ' . $last;`

## Database API

### Query Patterns

```php
global $DB;

// Single record
$user = $DB->get_record('user', ['id' => $userid], '*', MUST_EXIST);

// Multiple records
$records = $DB->get_records('local_pluginname_records', ['status' => 1]);

// Custom SQL
$sql = "SELECT r.*, u.firstname, u.lastname
          FROM {local_pluginname_records} r
          JOIN {user} u ON u.id = r.userid
         WHERE r.status = :status";
$records = $DB->get_records_sql($sql, ['status' => 1]);

// Insert
$record = new stdClass();
$record->userid = $userid;
$record->timecreated = time();
$record->timemodified = time();
$id = $DB->insert_record('local_pluginname_records', $record);

// Update
$record->id = $id;
$record->timemodified = time();
$DB->update_record('local_pluginname_records', $record);

// Delete
$DB->delete_records('local_pluginname_records', ['id' => $id]);
```

### Transactions

```php
$transaction = $DB->start_delegated_transaction();
try {
    // Database operations
    $DB->insert_record('table1', $data1);
    $DB->insert_record('table2', $data2);
    $transaction->allow_commit();
} catch (Exception $e) {
    $transaction->rollback($e);
}
```

## Templates (Mustache)

**Prefer Mustache templates over legacy PHP rendering (`html_writer`, inline HTML in renderers) for all UI output.** The legacy Forms API (`moodleform`) should only be used when you genuinely need server-side form validation, file uploads, or complex form element types that Mustache cannot easily replicate. For display pages, dashboards, tables, and simple interactive UIs, always use Mustache templates with AMD JavaScript modules.

### File Location & Naming

Templates live in `templates/*.mustache` within the plugin directory. Reference them as `componentname/filename` (no extension):

```
local/pluginname/templates/
├── main.mustache              → local_pluginname/main
├── record_list.mustache       → local_pluginname/record_list
└── record_card.mustache       → local_pluginname/record_card
```

Subdirectories under `templates/` are supported since Moodle 3.8.

### Basic Syntax

```mustache
{{! Comment — ignored in output }}

{{variable}}              {{! HTML-escaped output }}
{{{rawhtml}}}             {{! Unescaped — only use with format_text() output }}

{{#condition}}            {{! Section: renders if truthy / iterates if array }}
  <p>{{name}}</p>
{{/condition}}

{{^condition}}            {{! Inverted section: renders if falsy/empty }}
  <p>No data</p>
{{/condition}}

{{> core/loading }}       {{! Partial: include another template }}
```

### Moodle Helpers

```mustache
{{#str}} pluginname, local_pluginname {{/str}}             {{! Language string }}
{{#str}} greeting, local_pluginname, {{username}} {{/str}} {{! String with placeholder }}
{{#pix}} t/edit, core, Edit this item {{/pix}}             {{! Icon: key, component, alt }}
{{#userdate}} {{timestamp}}, {{#str}} strftimedate, core_langconfig {{/str}} {{/userdate}}
{{#shortentext}} 80, {{{description}}} {{/shortentext}}    {{! Truncate text }}
```

### Template Inheritance (Blocks)

Parent template defines replaceable blocks with `$`:

```mustache
{{! @template local_pluginname/layout }}
<div class="local_pluginname_layout">
  <h2>{{$heading}} Default heading {{/heading}}</h2>
  <div>{{$content}} Default content {{/content}}</div>
</div>
```

Child template overrides blocks:

```mustache
{{< local_pluginname/layout }}
  {{$heading}} My Custom Page {{/heading}}
  {{$content}}
    <p>Custom content here.</p>
  {{/content}}
{{/ local_pluginname/layout}}
```

### Renderable + Templatable Pattern (Recommended)

Create a renderable class that prepares data for the template. This enforces separation of logic from presentation.

**classes/output/records_page.php:**

```php
<?php
namespace local_pluginname\output;

defined('MOODLE_INTERNAL') || die();

use renderable;
use renderer_base;
use templatable;
use stdClass;

class records_page implements renderable, templatable {

    /** @var array $records The records to display. */
    private array $records;

    /** @var bool $canmanage Whether the user can manage records. */
    private bool $canmanage;

    public function __construct(array $records, bool $canmanage) {
        $this->records = $records;
        $this->canmanage = $canmanage;
    }

    /**
     * Export data for mustache template.
     *
     * @param renderer_base $output
     * @return stdClass
     */
    public function export_for_template(renderer_base $output): stdClass {
        $data = new stdClass();
        $data->canmanage = $this->canmanage;
        $data->hasrecords = !empty($this->records);
        $data->records = array_values(array_map(function($record) {
            return (object) [
                'id' => $record->id,
                'name' => $record->name,
                'status' => $record->status,
            ];
        }, $this->records));
        return $data;
    }
}
```

**templates/records_page.mustache:**

```mustache
{{!
    @template local_pluginname/records_page

    Records listing page.

    Context variables required for this template:
    * canmanage bool - Whether user can manage records.
    * hasrecords bool - Whether there are records to show.
    * records array - List of record objects with id, name, status.

    Example context (json):
    { "canmanage": true, "hasrecords": true, "records": [{"id": 1, "name": "Test", "status": 1}] }
}}
<div class="local_pluginname_records_page">
    {{#hasrecords}}
    <table class="table table-striped">
        <thead>
            <tr>
                <th>{{#str}} name, local_pluginname {{/str}}</th>
                <th>{{#str}} status, local_pluginname {{/str}}</th>
                {{#canmanage}}<th>{{#str}} actions, local_pluginname {{/str}}</th>{{/canmanage}}
            </tr>
        </thead>
        <tbody>
            {{#records}}
            <tr>
                <td>{{name}}</td>
                <td>{{status}}</td>
                {{#canmanage}}
                <td>
                    <a href="edit.php?id={{id}}">{{#pix}} t/edit, core, Edit {{/pix}}</a>
                    <a href="delete.php?id={{id}}">{{#pix}} t/delete, core, Delete {{/pix}}</a>
                </td>
                {{/canmanage}}
            </tr>
            {{/records}}
        </tbody>
    </table>
    {{/hasrecords}}
    {{^hasrecords}}
    <p>{{#str}} norecords, local_pluginname {{/str}}</p>
    {{/hasrecords}}
</div>
```

**Rendering from a page script:**

```php
$records = $DB->get_records('local_pluginname_records');
$canmanage = has_capability('local/pluginname:manage', $context);

$page = new \local_pluginname\output\records_page($records, $canmanage);
echo $OUTPUT->render($page);
```

When the renderable class name matches the template filename, `$OUTPUT->render()` automatically calls `export_for_template()` and `render_from_template()` — no explicit renderer method needed.

### Initialising JavaScript from Templates

Use `{{#js}}` blocks to load AMD modules. Use `data-*` attributes (not CSS classes) for JS hooks:

```mustache
<div id="{{uniqid}}-records" data-region="records-list">
    {{! template content }}
</div>
{{#js}}
require(['local_pluginname/records'], function(module) {
    module.init('{{uniqid}}-records');
});
{{/js}}
```

### Rendering Templates from JavaScript (AJAX)

```javascript
import Templates from 'core/templates';
import {exception as displayException} from 'core/notification';

const context = { name: 'Example', status: 1 };

Templates.renderForPromise('local_pluginname/record_card', context)
    .then(({html, js}) => {
        Templates.replaceNodeContents(document.querySelector('[data-region="detail"]'), html, js);
    })
    .catch((error) => displayException(error));
```

### Key Rules

1. **Escape by default** — `{{var}}` is HTML-escaped. Only use `{{{var}}}` for content already processed by `format_text()`.
2. **Use `array_values()`** — Mustache treats non-sequential PHP array keys as objects (not iterable). Always wrap with `array_values()`.
3. **Add boolean flags for empty checks** — Mustache cannot test `count()`. Add explicit `hasitems` booleans in `export_for_template()`.
4. **Use Bootstrap classes** — Moodle ships Bootstrap; use its utility classes (`table`, `btn`, `alert`, `card`, etc.) rather than custom CSS.
5. **`data-*` attributes for JS** — Never rely on CSS classes for JavaScript selectors. Theme designers may change classes.
6. **Document templates** — Include `@template`, context variable descriptions, and example JSON in the header comment. This enables the Template Library tool in Moodle.
7. **Purge caches** — Templates are cached. Run `sudo php /var/www/moodle/admin/cli/purge_caches.php` after editing templates (or enable Theme Designer Mode during development).
8. **Theme overridability** — Templates can be overridden by themes. Keep templates focused on structure, not logic, to make overrides straightforward.

### When to Still Use the Forms API

The legacy `moodleform` class is still appropriate for:
- Forms requiring **server-side validation** with error messages rendered inline next to fields.
- **File upload** elements (`filepicker`, `filemanager`).
- Complex form element types (`date_time_selector`, `editor`, `autocomplete`).
- Forms that need **draft area** integration for rich text editors.

For everything else — display pages, dashboards, listing tables, action menus, cards, modals — use Mustache templates.

## Forms API

```php
<?php
namespace local_pluginname\form;

defined('MOODLE_INTERNAL') || die();

require_once($CFG->libdir . '/formslib.php');

class edit_form extends \moodleform {

    protected function definition() {
        $mform = $this->_form;

        $mform->addElement('text', 'name', get_string('name', 'local_pluginname'));
        $mform->setType('name', PARAM_TEXT);
        $mform->addRule('name', null, 'required', null, 'client');

        $mform->addElement('select', 'status', get_string('status', 'local_pluginname'), [
            0 => get_string('inactive', 'local_pluginname'),
            1 => get_string('active', 'local_pluginname'),
        ]);

        $this->add_action_buttons();
    }

    public function validation($data, $files) {
        $errors = parent::validation($data, $files);
        // Custom validation
        return $errors;
    }
}
```

## Output API

```php
global $PAGE, $OUTPUT;

$PAGE->set_url(new moodle_url('/local/pluginname/index.php'));
$PAGE->set_context(context_system::instance());
$PAGE->set_title(get_string('pluginname', 'local_pluginname'));
$PAGE->set_heading(get_string('pluginname', 'local_pluginname'));

echo $OUTPUT->header();
echo $OUTPUT->heading(get_string('title', 'local_pluginname'));

// Render template
echo $OUTPUT->render_from_template('local_pluginname/main', $templatedata);

echo $OUTPUT->footer();
```

## Capabilities (db/access.php)

```php
<?php
defined('MOODLE_INTERNAL') || die();

$capabilities = [
    'local/pluginname:view' => [
        'captype' => 'read',
        'contextlevel' => CONTEXT_SYSTEM,
        'archetypes' => [
            'user' => CAP_ALLOW,
        ],
    ],
    'local/pluginname:manage' => [
        'captype' => 'write',
        'contextlevel' => CONTEXT_SYSTEM,
        'archetypes' => [
            'manager' => CAP_ALLOW,
        ],
        'riskbitmask' => RISK_CONFIG,
    ],
];
```

### Checking Capabilities

```php
$context = context_system::instance();

// Require capability (throws exception if not allowed)
require_capability('local/pluginname:manage', $context);

// Check capability (returns bool)
if (has_capability('local/pluginname:view', $context)) {
    // User can view
}
```

## Language Strings

File: `lang/en/local_pluginname.php`

```php
<?php
defined('MOODLE_INTERNAL') || die();

$string['pluginname'] = 'Plugin Name';
$string['modulename'] = 'Plugin Name';
$string['modulenameplural'] = 'Plugin Names';
$string['status'] = 'Status';
$string['active'] = 'Active';
$string['inactive'] = 'Inactive';
$string['error:notfound'] = 'Record not found';
```

Usage:
```php
echo get_string('pluginname', 'local_pluginname');
echo get_string('error:notfound', 'local_pluginname', $recordid);
```

## Events API

### Defining Events

```php
<?php
namespace local_pluginname\event;

defined('MOODLE_INTERNAL') || die();

class record_created extends \core\event\base {

    protected function init() {
        $this->data['crud'] = 'c';
        $this->data['edulevel'] = self::LEVEL_OTHER;
        $this->data['objecttable'] = 'local_pluginname_records';
    }

    public static function get_name() {
        return get_string('eventrecordcreated', 'local_pluginname');
    }

    public function get_description() {
        return "User {$this->userid} created record {$this->objectid}";
    }
}
```

### Triggering Events

```php
$event = \local_pluginname\event\record_created::create([
    'objectid' => $record->id,
    'context' => context_system::instance(),
]);
$event->trigger();
```

## External API (Web Services)

File: `db/services.php`

```php
<?php
defined('MOODLE_INTERNAL') || die();

$functions = [
    'local_pluginname_get_records' => [
        'classname' => 'local_pluginname\external\get_records',
        'methodname' => 'execute',
        'description' => 'Get records',
        'type' => 'read',
        'ajax' => true,
        'capabilities' => 'local/pluginname:view',
    ],
];
```

File: `classes/external/get_records.php`

```php
<?php
namespace local_pluginname\external;

defined('MOODLE_INTERNAL') || die();

use external_api;
use external_function_parameters;
use external_single_structure;
use external_multiple_structure;
use external_value;

class get_records extends external_api {

    public static function execute_parameters(): external_function_parameters {
        return new external_function_parameters([
            'status' => new external_value(PARAM_INT, 'Status filter', VALUE_DEFAULT, null),
        ]);
    }

    public static function execute(?int $status = null): array {
        global $DB;

        $params = self::validate_parameters(self::execute_parameters(), ['status' => $status]);
        $context = \context_system::instance();
        self::validate_context($context);
        require_capability('local/pluginname:view', $context);

        $conditions = $params['status'] !== null ? ['status' => $params['status']] : [];
        $records = $DB->get_records('local_pluginname_records', $conditions);

        return ['records' => array_values($records)];
    }

    public static function execute_returns(): external_single_structure {
        return new external_single_structure([
            'records' => new external_multiple_structure(
                new external_single_structure([
                    'id' => new external_value(PARAM_INT, 'Record ID'),
                    'userid' => new external_value(PARAM_INT, 'User ID'),
                    'status' => new external_value(PARAM_INT, 'Status'),
                ])
            ),
        ]);
    }
}
```

## Scheduled Tasks

File: `classes/task/sync_task.php`

```php
<?php
namespace local_pluginname\task;

defined('MOODLE_INTERNAL') || die();

class sync_task extends \core\task\scheduled_task {

    public function get_name() {
        return get_string('synctask', 'local_pluginname');
    }

    public function execute() {
        mtrace('Starting sync...');
        // Task logic
        mtrace('Sync complete.');
    }
}
```

File: `db/tasks.php`

```php
<?php
defined('MOODLE_INTERNAL') || die();

$tasks = [
    [
        'classname' => 'local_pluginname\task\sync_task',
        'blocking' => 0,
        'minute' => '0',
        'hour' => '*/6',
        'day' => '*',
        'month' => '*',
        'dayofweek' => '*',
    ],
];
```

## CLI Scripts

```php
<?php
define('CLI_SCRIPT', true);
require(__DIR__ . '/../../config.php');
require_once($CFG->libdir . '/clilib.php');

list($options, $unrecognized) = cli_get_params([
    'help' => false,
    'userid' => null,
], [
    'h' => 'help',
    'u' => 'userid',
]);

if ($options['help']) {
    echo "Usage: php script.php [--userid=ID]

Options:
  -h, --help    Show this help
  -u, --userid  User ID to process
";
    exit(0);
}

// Script logic
```

## Common Pitfalls

1. **Forgetting `defined('MOODLE_INTERNAL') || die();`** - Required in all PHP files except entry points
2. **Not purging caches** - Always run `sudo php /var/www/moodle/admin/cli/purge_caches.php` after configuration changes
3. **Wrong parameter types** - Use PARAM_* constants: PARAM_INT, PARAM_TEXT, PARAM_RAW, PARAM_BOOL
4. **Missing language strings** - All user-visible text must use `get_string()`
5. **Direct SQL without placeholders** - Always use named parameters: `['status' => $status]`
6. **Not incrementing version** - Version must increase for upgrade.php to run
7. **Capability checks** - Always verify capabilities before sensitive operations
8. **Version increase** - Always run `sudo php /var/www/moodle/admin/cli/upgrade.php` after version changes to update the Moodle database


