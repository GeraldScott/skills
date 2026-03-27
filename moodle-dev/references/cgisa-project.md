# CGISA Moodle Project Reference

This reference contains project-specific information for the CGISA (Chartered Governance Institute of Southern Africa) Moodle installation.

## Custom Plugins

| Plugin | Location | Purpose |
|--------|----------|---------|
| student_exams | `local/student_exams` | Module results management |
| finance | `local/finance` | Finance management |
| personal_information | `local/personal_information` | Personal information maintenance |
| samms_api | `local/samms_api` | Backend API integration (dependency) |
| evolution_api | `local/evolution_api` | Evolution API integration |

### Plugin Dependencies

When `local/samms_api` or `local/evolution_api` version changes, update dependencies in all dependent plugins' `version.php` files.

## SAMMS API Integration

The Moodle plugins communicate with the SAMMS backend via `local/samms_api`.

### Backend Resources

API endpoints are defined in:
```
/home/geraldo/samms/samms-web/src/main/java/io/archton/samms/web/rest/*Resource.java
```

Search these files to understand available GET, POST, PUT, and DELETE operations.

### Domain Entities

Data structures are defined in:
```
/home/geraldo/samms/samms-web/src/main/java/io/archton/samms/domain/*.java
```

**CRITICAL:** PHP data objects in Moodle plugins must align with these Java entity structures.

### Integration Pattern

```php
// Example API call via samms_api
$api = new \local_samms_api\client();
$response = $api->get('/students/' . $studentid);
$data = json_decode($response->body);
```

## Version Management

### version.php Updates

After any code change:
1. Increment version number (YYYYMMDDXX format)
2. Update dependency versions if samms_api or evolution_api changed
3. Update README.md zip file version references
4. Update CHANGELOG if functionality changed

### Example Version Update

```php
// Before
$plugin->version = 2025013100;
$plugin->dependencies = [
    'local_samms_api' => 2025010100,
];

// After (increment last two digits for same-day changes)
$plugin->version = 2025013101;
$plugin->dependencies = [
    'local_samms_api' => 2025013100,  // Updated dependency
];
```

## File Paths

| Resource | Path |
|----------|------|
| Moodle root | `/var/www/moodle` |
| Data directory | `/var/www/moodledata` |
| CLI tools | `/var/www/moodle/admin/cli/` |
| SAMMS backend | `/home/geraldo/samms/samms-web/src/main` |
| REST resources | `<samms>/java/io/archton/samms/web/rest/` |
| Domain entities | `<samms>/java/io/archton/samms/domain/` |

## Development Workflow

1. Make code changes to plugin files
2. Increment version in `version.php`
3. Update dependencies if needed
4. Run upgrade: `sudo php /var/www/moodle/admin/cli/upgrade.php --non-interactive`
5. Purge caches: `sudo php /var/www/moodle/admin/cli/purge_caches.php`
6. Test functionality
7. Update README.md and CHANGELOG if applicable
