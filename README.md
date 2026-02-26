# Session Scribe

A Claude Code skill that automatically documents your development sessions, capturing architectural decisions, reasoning, and context evolution.

## What It Does

Session Scribe transforms ephemeral Claude Code sessions into structured, persistent documentation:

- **Session Summaries** - Chronological records of development activities
- **Architectural Decision Records (ADRs)** - Documented decisions with rationale
- **Change Logs** - Detailed records of file modifications
- **Knowledge Base** - Captured patterns, practices, and project context

## Installation

### From skills.sh

```bash
claude-code skill add https://skills.sh/scribe
```

### Manual Installation

1. Clone this repository:
   ```bash
   git clone https://github.com/yourusername/scribe.git
   cd scribe
   ```

2. Install as a local skill:
   ```bash
   claude-code skill add .
   ```

## Quick Start

### First-Time Setup

When you first invoke `@scribe` in a project, you'll be prompted to configure:

1. **Documentation Location** - Where to save your docs
2. **Project Keyword** - A short identifier (e.g., "my-api")

This creates a `.scribe-config.json` file that remembers your preferences.

### Basic Usage

**Document current session:**
```
@scribe
```

**Load previous session context:**
```
@scribe-load my-api
```

## Usage

### Documenting Sessions

During or after a coding session, invoke Session Scribe:

```
@scribe
```

On first use, you'll be asked:
- Where to save documentation (default: `./scribe`)
- A project keyword for this project (e.g., "my-api", "web-app")

Session Scribe will then:

1. Analyze recent file changes and tool usage
2. Extract goals, decisions, and reasoning from the conversation
3. Generate structured documentation
4. Update the documentation index
5. Save configuration for future sessions

### Loading Previous Context

Load context from previous sessions using the project keyword:

```
@scribe-load my-api
```

This displays:
- Latest session summary
- Recent architectural decisions (ADRs)
- Knowledge base articles
- Outstanding questions and next steps

**List all documented projects:**
```
@scribe-load --list
```

**Load with filters:**
```
@scribe-load my-api --after 2026-02-20
@scribe-load my-api authentication
@scribe-load my-api decisions
```

### Documentation Structure

Session Scribe creates and maintains a flat structure organized by project keyword:

```
scribe/                                  # Documentation root
  my-api/                                # Project keyword
    README.md                            # Project index
    session-2026-02-26-14-45.md         # Session summaries
    session-2026-02-27-10-30.md
    decision-0001-use-typescript.md      # Architectural decisions
    decision-0002-api-design.md
    change-2026-02-26.md                 # Daily change logs
    change-2026-02-27.md
    knowledge-authentication.md          # Knowledge articles
    knowledge-database-schema.md
  web-app/                               # Another project
    README.md
    session-2026-02-26-11-00.md
    decision-0001-use-react.md
```

**File Prefixes:**
- `session-` - Session summaries
- `decision-` - ADRs
- `change-` - Change logs
- `knowledge-` - Knowledge base

## Example Output

### Session Summary

```markdown
# Session: Implement User Authentication

**Date:** 2026-02-26 14:45
**Duration:** ~45 minutes
**Status:** Completed

## Objectives
- Add JWT-based authentication
- Create login/logout endpoints
- Implement auth middleware

## Key Activities
- Created auth service with token generation
- Added middleware for protected routes
- Implemented password hashing with bcrypt

## Decisions Made
- Using JWT over session-based auth for stateless API
- Storing tokens in httpOnly cookies for security
- 24-hour token expiration with refresh mechanism
```

### Architectural Decision Record

```markdown
# ADR 0001: Use JWT for Authentication

**Date:** 2026-02-26
**Status:** Accepted

## Context
Need to implement authentication for the API. Must support:
- Stateless operation
- Mobile and web clients
- Horizontal scaling

## Decision
Use JSON Web Tokens (JWT) for authentication.

## Rationale
- Stateless: No server-side session storage required
- Scalable: Works across multiple server instances
- Standard: Well-supported by libraries and tools
- Flexible: Can include custom claims

## Consequences
**Positive:**
- Easy to scale horizontally
- Works well with microservices
- No database lookups for each request

**Negative:**
- Cannot revoke tokens before expiration
- Slightly larger than session IDs
- Must manage token refresh carefully
```

## Use Cases

### After Major Feature Development
```
I just finished implementing the user dashboard. @scribe document this session.
```

### When Making Architectural Decisions
```
We decided to use PostgreSQL instead of MongoDB. @scribe capture this decision.
```

### End of Day Documentation
```
@scribe summarize today's session
```

### Knowledge Capture
```
@scribe document the authentication flow we implemented
```

### Starting Work on Different Project
```
@scribe-load my-api
# Reviews recent sessions, decisions, and knowledge
# Now ready to continue work with full context
```

### Reviewing Past Decisions
```
@scribe-load my-api decisions
# Shows all architectural decisions made
# Helps understand why things are the way they are
```

### Finding Session About Specific Topic
```
@scribe-load my-api authentication
# Finds all sessions mentioning authentication
# Quick way to recall implementation details
```

## Best Practices

1. **Invoke Regularly** - Document sessions while context is fresh
2. **Be Descriptive** - Provide clear commit messages and comments
3. **Review Generated Docs** - Check accuracy and completeness
4. **Commit Documentation** - Version control your session docs alongside code
5. **Link to Code** - Use file paths and line numbers in discussions

## Configuration

Session Scribe uses `.scribe-config.json` to store project-specific settings.

### Automatic Setup

On first invocation, Session Scribe prompts you to configure:

1. **Documentation Location**
   - `./scribe` (default, in current project)
   - `./docs` (in docs folder)
   - `~/.scribe` (global location for all projects)
   - Custom path

2. **Project Keyword**
   - Short identifier (e.g., "my-api", "web-app")
   - Used in filenames and for `@scribe-load`
   - Must be lowercase with hyphens

### Configuration File

The `.scribe-config.json` stores your preferences:

```json
{
  "version": "1.0",
  "docsPath": "./scribe",
  "projectKeyword": "my-api",
  "preferences": {
    "autoGenerateADRs": true,
    "autoUpdateChangelog": true,
    "includeCodeSnippets": true,
    "maxSnippetLines": 20
  }
}
```

### Editing Configuration

You can edit `.scribe-config.json` directly:

- Change `docsPath` to move documentation location
- Change `projectKeyword` to rename project identifier
- Toggle preferences for auto-generation features
- Adjust `maxSnippetLines` (5-100) for code example length

See [CONFIG.md](CONFIG.md) for full configuration reference.

### Multi-Project Setup

Each project gets its own configuration:

```
~/projects/
  api/.scribe-config.json (keyword: "my-api")
  web/.scribe-config.json (keyword: "my-web")
  mobile/.scribe-config.json (keyword: "my-mobile")
```

Switch between projects with `@scribe-load`:
```
@scribe-load my-api    # Load API project context
@scribe-load my-web    # Load web project context
```

### Global Documentation

To centralize documentation across projects:

```json
{
  "docsPath": "~/.scribe",
  "projectKeyword": "my-api"
}
```

All projects save to `~/.scribe/[keyword]/` for easy cross-project access.

## Benefits

- **Institutional Memory** - Never lose track of why decisions were made
- **Onboarding** - New team members can understand project evolution
- **Context Switching** - Quickly recall what you were working on
- **Code Review** - Share architectural reasoning with reviewers
- **Documentation** - Auto-generated docs that stay in sync with development

## Integration Ideas

### Git Hooks
Automatically document commits:
```bash
# .git/hooks/post-commit
claude-code "@scribe document last commit"
```

### CI/CD
Generate documentation during builds:
```yaml
- name: Document Session
  run: claude-code "@scribe"
```

### MCP Server (Future)
Expose documentation as searchable context for future sessions.

## Contributing

Contributions welcome! This skill can be extended to:

- Integrate with local LLMs for summarization
- Support custom documentation templates
- Add searchable documentation interface
- Generate context packs for session reloading
- Track decision evolution across sessions

## License

MIT

## Links

- [Claude Code Documentation](https://docs.claude.com/claude-code)
- [skills.sh](https://skills.sh)
- [Architectural Decision Records](https://adr.github.io/)
