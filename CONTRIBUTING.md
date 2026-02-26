# Contributing to Session Scribe

Thank you for your interest in contributing to Session Scribe! This document provides guidelines for contributing to the project.

## Getting Started

1. **Fork the repository** on GitHub
2. **Clone your fork** locally
3. **Create a branch** for your changes: `git checkout -b feature/your-feature-name`
4. **Make your changes** following the guidelines below
5. **Test your changes** thoroughly
6. **Submit a pull request**

## Development Setup

```bash
# Clone the repository
git clone https://github.com/yourusername/scribe.git
cd scribe

# Install Claude Code (if not already installed)
# Follow instructions at https://docs.claude.com/claude-code

# Test the skill locally
claude-code skill add .
```

## Testing Your Changes

### Manual Testing

1. Create a test project directory
2. Invoke the Session Scribe skill: `@scribe`
3. Verify documentation is generated correctly
4. Check all templates are properly populated
5. Ensure formatting is consistent

### Test Checklist

- [ ] Session summaries are generated with correct format
- [ ] ADRs follow the template structure
- [ ] File paths are correctly referenced
- [ ] Timestamps use ISO format (YYYY-MM-DD)
- [ ] Markdown formatting is valid
- [ ] Links between documents work correctly
- [ ] README index is properly updated

## Code Style

### Skill Prompt (skill.md)

- Use clear, imperative language
- Provide concrete examples
- Include edge case handling
- Maintain consistent formatting
- Keep instructions actionable

### Documentation Templates

- Use consistent markdown formatting
- Include helpful comments and placeholders
- Provide examples where helpful
- Keep structure flexible but guided

### Markdown Standards

- Use `#` for headers (not underlines)
- Use `**bold**` for emphasis
- Use ` ``` ` for code blocks with language tags
- Use `- [ ]` for task lists
- Keep line length reasonable (~100 chars)

## Adding New Features

### New Documentation Types

To add a new documentation type:

1. Create a template in `templates/new-type-template.md`
2. Update `skill.md` with generation instructions
3. Add examples to `examples/session-docs/`
4. Update `README.md` with usage documentation
5. Test thoroughly with real-world scenarios

### Extending Existing Features

1. Discuss the change in an issue first
2. Maintain backward compatibility
3. Update all relevant templates
4. Add examples demonstrating the feature
5. Update documentation

## Pull Request Process

1. **Update Documentation**: Ensure README and templates reflect changes
2. **Add Examples**: Provide example outputs for new features
3. **Test Thoroughly**: Verify changes work in multiple scenarios
4. **Write Clear Commit Messages**: Use conventional commits format
5. **Reference Issues**: Link to related issues/discussions

### Commit Message Format

```
type(scope): brief description

Longer description if needed

Fixes #123
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Formatting changes
- `refactor`: Code restructuring
- `test`: Adding tests
- `chore`: Maintenance tasks

### Example Commit Messages

```
feat(templates): add API endpoint documentation template

Added new template for documenting REST API endpoints with
request/response examples and error codes.

Closes #45
```

```
fix(skill): correct timestamp formatting in session summaries

Timestamps now consistently use ISO 8601 format (YYYY-MM-DD HH:MM)
instead of locale-specific formats.

Fixes #67
```

## What to Contribute

### Ideas for Contributions

- **Templates**: New documentation types or improved existing ones
- **Examples**: More real-world example outputs
- **Features**: Enhanced skill capabilities (see roadmap)
- **Documentation**: Improved usage guides and tutorials
- **Bug Fixes**: Reported issues and edge cases
- **Integration**: Scripts or tools to enhance functionality

### Priority Areas

1. Local LLM integration for summarization
2. MCP server for documentation access
3. Hooks integration for automatic triggering
4. Custom template support
5. Configuration options

## Code of Conduct

### Our Standards

- Be respectful and inclusive
- Provide constructive feedback
- Focus on what's best for the community
- Show empathy towards others

### Unacceptable Behavior

- Harassment or discriminatory language
- Trolling or insulting comments
- Personal or political attacks
- Publishing others' private information

## Questions?

- Open an issue for bugs or feature requests
- Start a discussion for questions or ideas
- Check existing issues before creating new ones

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

## Recognition

Contributors will be recognized in:
- GitHub contributors list
- Release notes for significant contributions
- README acknowledgments section

Thank you for helping make Session Scribe better! 🙏
