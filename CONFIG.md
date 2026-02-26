# Session Scribe Configuration

Session Scribe uses a `.scribe-config.json` file in your project directory to manage documentation settings.

## First-Time Setup

When you first invoke `@scribe` in a project, you'll be prompted to configure:

1. **Documentation Location**
   - `./scribe` - In current project (default)
   - `./docs` - In project's docs folder
   - `~/.scribe` - Global location for all projects
   - Custom path

2. **Project Keyword**
   - Short identifier for this project (e.g., "my-api", "auth-service")
   - Used in filenames and for `@scribe-load`
   - Must be lowercase with hyphens only

## Configuration File

The `.scribe-config.json` file is created automatically:

```json
{
  "version": "1.0",
  "docsPath": "./scribe",
  "projectKeyword": "my-project",
  "createdAt": "2026-02-26T15:00:00Z",
  "lastSessionNumber": 0,
  "preferences": {
    "autoGenerateADRs": true,
    "autoUpdateChangelog": true,
    "includeCodeSnippets": true,
    "maxSnippetLines": 20,
    "timestampFormat": "YYYY-MM-DD-HH-MM",
    "adrNumberFormat": "0000"
  },
  "metadata": {
    "projectName": "My Awesome Project",
    "description": "A brief description",
    "repository": "https://github.com/user/repo",
    "team": ["Alice", "Bob"]
  }
}
```

## Configuration Options

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `version` | string | Config schema version (currently "1.0") |
| `docsPath` | string | Where to save documentation |
| `projectKeyword` | string | Unique project identifier |

### Optional Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `createdAt` | string | (auto) | When config was created (ISO 8601) |
| `lastSessionNumber` | integer | 0 | Counter for sessions (auto-incremented) |

### Preferences

| Preference | Type | Default | Description |
|------------|------|---------|-------------|
| `autoGenerateADRs` | boolean | `true` | Auto-create ADR for significant decisions |
| `autoUpdateChangelog` | boolean | `true` | Update changelog with each session |
| `includeCodeSnippets` | boolean | `true` | Include code examples in docs |
| `maxSnippetLines` | integer | `20` | Max lines in code snippets (5-100) |
| `timestampFormat` | string | `YYYY-MM-DD-HH-MM` | Filename timestamp format |
| `adrNumberFormat` | string | `0000` | ADR numbering format |

#### Timestamp Formats

- `YYYY-MM-DD-HH-MM` - Full timestamp (e.g., 2026-02-26-14-45)
- `YYYY-MM-DD` - Date only (e.g., 2026-02-26)
- `YYYYMMDD-HHMM` - Compact format (e.g., 20260226-1445)

#### ADR Number Formats

- `0000` - Four digits (0001, 0002, ..., 9999)
- `000` - Three digits (001, 002, ..., 999)
- `00` - Two digits (01, 02, ..., 99)

### Metadata (Optional)

Descriptive information about your project:

```json
{
  "metadata": {
    "projectName": "My Awesome API",
    "description": "REST API for awesome features",
    "repository": "https://github.com/myorg/awesome-api",
    "team": ["Alice", "Bob", "Charlie"]
  }
}
```

This metadata appears in documentation headers and helps organize multi-project setups.

## Usage Examples

### Local Project Documentation

```json
{
  "docsPath": "./scribe",
  "projectKeyword": "my-api"
}
```

Creates: `./scribe/my-api/` in your project

Files: `session-2026-02-26-14-45.md`, `decision-0001-title.md`, etc.

### Global Documentation Repository

```json
{
  "docsPath": "~/.scribe/my-api",
  "projectKeyword": "my-api"
}
```

Creates: `~/.scribe/my-api/`

Useful for documenting multiple related projects in one place.

### Minimal Code Snippets

```json
{
  "preferences": {
    "includeCodeSnippets": true,
    "maxSnippetLines": 10
  }
}
```

Limits code examples to 10 lines for more concise documentation.

### Manual ADR Creation

```json
{
  "preferences": {
    "autoGenerateADRs": false
  }
}
```

You'll need to explicitly request ADRs: `@scribe create ADR for [decision]`

## Modifying Configuration

You can edit `.scribe-config.json` directly at any time. Changes take effect on next `@scribe` invocation.

### Safe to Modify

- `preferences.*` - All preference settings
- `metadata.*` - All metadata fields
- `docsPath` - (but existing docs won't move)

### Don't Manually Change

- `version` - Managed by Session Scribe
- `lastSessionNumber` - Auto-incremented counter
- `createdAt` - Historical timestamp

### Changing Documentation Location

If you change `docsPath`, you'll need to manually move existing documentation:

```bash
mv ./session-docs ./docs/sessions
```

Or update the config to point to the existing location.

### Changing Project Keyword

If you change `projectKeyword`, new sessions will use the new keyword, but existing files keep their original names. Consider this breaking change for `@scribe-load`.

To rename existing files:

```bash
cd session-docs/sessions
for f in old-keyword-*.md; do
  mv "$f" "${f/old-keyword/new-keyword}"
done
```

## Version Control

### Should I Commit `.scribe-config.json`?

**Yes, if:**
- Team members should use same settings
- Documentation is project-specific
- You want to share configuration

**No, if:**
- Each developer has custom preferences
- Documentation path is environment-specific
- Configuration contains local paths

Add to `.gitignore` if not committing:

```gitignore
# Session Scribe config
.scribe-config.json
```

### Team Collaboration

For teams, commit a template:

```bash
# Commit the template
git add .scribe-config.template.json

# Each developer copies it
cp .scribe-config.template.json .scribe-config.json
# Then edit for local paths
```

Or commit with relative paths that work for everyone:

```json
{
  "docsPath": "./session-docs",
  "projectKeyword": "team-project"
}
```

## Multi-Project Setup

For managing multiple projects:

### Option 1: Per-Project Config

Each project has its own `.scribe-config.json`:

```
~/projects/
  api/.scribe-config.json (keyword: "my-api")
  web/.scribe-config.json (keyword: "my-web")
  mobile/.scribe-config.json (keyword: "my-mobile")
```

Use `@scribe-load` to switch contexts:
- `@scribe-load my-api`
- `@scribe-load my-web`

### Option 2: Global Documentation

All projects save to global location:

```json
{
  "docsPath": "~/.scribe/my-api",
  "projectKeyword": "my-api"
}
```

```
~/.scribe/
  my-api/
  my-web/
  my-mobile/
```

List all projects: `@scribe-load --list`

## Validation

The configuration follows a JSON Schema: `config-schema.json`

Session Scribe validates configuration on load and reports errors:

```
Configuration error in .scribe-config.json:
- projectKeyword must be lowercase with hyphens only
- maxSnippetLines must be between 5 and 100
```

## Reset Configuration

To reset and reconfigure:

1. Delete `.scribe-config.json`
2. Invoke `@scribe`
3. Answer setup prompts

Your existing documentation is preserved (not deleted).

## Examples

### Minimal Configuration

```json
{
  "version": "1.0",
  "docsPath": "./docs",
  "projectKeyword": "myapp"
}
```

### Full Configuration

```json
{
  "version": "1.0",
  "docsPath": "./session-docs",
  "projectKeyword": "awesome-api",
  "createdAt": "2026-02-26T15:00:00Z",
  "lastSessionNumber": 42,
  "preferences": {
    "autoGenerateADRs": true,
    "autoUpdateChangelog": true,
    "includeCodeSnippets": true,
    "maxSnippetLines": 15,
    "timestampFormat": "YYYY-MM-DD-HH-MM",
    "adrNumberFormat": "0000"
  },
  "metadata": {
    "projectName": "Awesome API",
    "description": "Production REST API serving awesome features",
    "repository": "https://github.com/myorg/awesome-api",
    "team": ["Alice (Tech Lead)", "Bob (Backend)", "Charlie (DevOps)"]
  }
}
```

### Monorepo Configuration

For monorepos with multiple services:

```
monorepo/
  services/
    api/.scribe-config.json (keyword: "monorepo-api")
    web/.scribe-config.json (keyword: "monorepo-web")
  .scribe-config.json (keyword: "monorepo-shared")
```

Each service gets its own documentation, plus shared docs at root.

## Troubleshooting

### "Configuration not found"

Session Scribe looks in the current working directory. Ensure you're in the project root or specify path.

### "Invalid project keyword"

Keywords must be:
- Lowercase letters
- Numbers
- Hyphens only
- 2-50 characters

Valid: `my-api`, `auth-v2`, `web-frontend`
Invalid: `My_API`, `auth.service`, `a`

### "Documentation path not writable"

Session Scribe can't create or write to the specified `docsPath`. Check:
- Directory exists or parent is writable
- Permissions allow writing
- Path is valid (no typos)

### "Session number mismatch"

If `lastSessionNumber` is corrupted, reset it:

```json
{
  "lastSessionNumber": 0
}
```

Session Scribe will count existing sessions and update accordingly.

## See Also

- [README.md](README.md) - Main documentation
- [CONTRIBUTING.md](CONTRIBUTING.md) - Contributing guidelines
- [skill.md](skill.md) - Skill implementation
- [scribe-load.md](scribe-load.md) - Context loading skill
