# Local Installation Guide

## Install Session Scribe Locally

### Step 1: Install the Skill

From the `scribe` directory:

```bash
cd /Users/dreamcast/repos/scribe
claude-code skill add .
```

Or install from anywhere using the full path:

```bash
claude-code skill add /Users/dreamcast/repos/scribe
```

### Step 2: Verify Installation

Check that the skills are installed:

```bash
claude-code skill list
```

You should see:
- `scribe` - Document current Claude Code session
- `scribe-load` - Load previous session context by project keyword

### Step 3: Test in a Project

Navigate to any project and start a Claude Code session:

```bash
cd ~/projects/my-test-project
claude-code
```

In the Claude Code session, type:

```
@scribe
```

You'll be prompted to:
1. Choose documentation location
2. Enter a project keyword

After answering, Session Scribe will document your session!

## Usage Examples

### Document Current Session

```
@scribe
```

On first use, you'll see prompts like:
```
Where should Session Scribe save documentation?
- ./session-docs (default, in current project)
- ./docs/sessions (in docs folder)
- ~/.scribe/my-test-project (global location)
- Other (custom path)
```

Then:
```
What keyword identifies this project?
- my-test-project (suggested from directory)
- Other (custom keyword)
```

After configuration, it will analyze your session and create documentation.

### Load Previous Context

```
@scribe-load my-test-project
```

This will show:
- Latest session summary
- Recent architectural decisions
- Knowledge base
- Outstanding questions

### List All Projects

```
@scribe-load --list
```

Shows all projects you've documented.

## Quick Test

Here's a complete test workflow:

```bash
# 1. Create a test project
mkdir -p ~/test-scribe-project
cd ~/test-scribe-project
echo "# Test Project" > README.md

# 2. Start Claude Code
claude-code

# In the Claude Code session:
# "Create a simple Node.js project with package.json"
# (Claude will create files)

# Then invoke:
# "@scribe"

# Answer the prompts:
# - Location: ./session-docs
# - Keyword: test-project

# Check the results:
# "Show me what was created in session-docs/"

# You should see:
# session-docs/
#   README.md
#   sessions/test-project-2026-02-26-XX-XX.md
```

## Troubleshooting

### "Skill not found"

Make sure the skill is installed:
```bash
claude-code skill list
```

If not listed, reinstall:
```bash
claude-code skill add /Users/dreamcast/repos/scribe
```

### "Configuration error"

Delete the config and start over:
```bash
rm .scribe-config.json
```

Then invoke `@scribe` again to reconfigure.

### Skill doesn't respond

Make sure you're using the `@` symbol:
- ✅ `@scribe`
- ❌ `scribe`
- ❌ `/scribe`

### Can't find documented project with @scribe-load

Make sure you're using the exact keyword from `.scribe-config.json`:
```bash
# Check the keyword in your project
cat .scribe-config.json | grep projectKeyword
```

## Uninstall

### Remove the Skills

To remove Session Scribe:

```bash
claude-code skill remove scribe
claude-code skill remove scribe-load
```

### Verify Removal

Check that the skills are gone:

```bash
claude-code skill list
```

### What Gets Removed?

- ✅ Skill definitions and invocation commands
- ❌ **Your documentation is NOT removed** - All `session-docs/` folders remain
- ❌ **Config files remain** - `.scribe-config.json` files stay in projects

### Clean Up Documentation (Optional)

If you want to remove generated documentation from a project:

```bash
cd ~/projects/my-project
rm -rf session-docs/
rm .scribe-config.json
```

**Note**: Your documentation is just markdown files and is still useful even without the skill installed!

## Next Steps

Once installed:
1. Use `@scribe` after any significant coding session
2. Use `@scribe-load <keyword>` when switching between projects
3. Commit your `.scribe-config.json` if working in a team
4. Review generated documentation in `session-docs/`

## Development Mode

If you're actively developing the skill:

```bash
# Make changes to skill.md or scribe-load.md
# Reload the skill
claude-code skill remove scribe
claude-code skill add .

# Or update in place (if supported)
claude-code skill update scribe
```

Changes to the skill files take effect immediately in new Claude Code sessions.
