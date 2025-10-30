# Agent Collaboration Guide

This guide helps you effectively collaborate with AI agents (like Claude Code) on this project.

## Project Context

**App Purpose**: WorkoutTracker is a Django application for tracking fitness workouts, exercises, and progress over time.

**Development Philosophy**: Spec-driven development with clear documentation before implementation.

## How to Work with Agents

### 1. Start with Specifications

Before asking an agent to implement features:
- Create or reference a spec in `docs/specs/`
- Ensure the spec includes requirements, acceptance criteria, and edge cases
- Have the agent review the spec for completeness

### 2. Provide Context

When starting a new session, reference:
- Relevant spec files
- Related models/views/templates
- Any architectural decisions from `docs/architecture/`

### 3. Iterative Development

Effective iteration pattern:
```
1. Define feature in spec
2. Review architecture implications
3. Implement in small, testable increments
4. Test and validate
5. Refine based on results
```

### 4. Code Organization

Apps should be:
- **Single-purpose**: Each app handles one domain
- **Self-contained**: Include models, views, templates, tests
- **Well-documented**: Clear docstrings and comments

## Project Standards

### Code Style
- Follow PEP 8
- Use type hints where beneficial
- Write descriptive docstrings
- Keep functions focused and testable

### Testing
- Write tests alongside implementation
- Aim for high coverage of business logic
- Use factories for test data

### Git Workflow
- Meaningful commit messages
- Feature branches for new work
- Keep commits focused and atomic

## Common Agent Tasks

### Creating a New Feature
```
1. "Create a spec for [feature name] in docs/specs/"
2. Review and refine the spec
3. "Implement [feature name] according to specs/[filename].md"
4. "Write tests for [feature name]"
5. "Review and refactor if needed"
```

### Debugging Issues
```
1. "Investigate [symptom/error]"
2. Provide relevant logs/traces
3. "Fix [issue] and add tests to prevent regression"
```

### Refactoring
```
1. "Review [module/feature] for improvements"
2. "Refactor [specific area] while maintaining behavior"
3. Verify tests pass
```

## File Locations Reference

- **Specs**: `docs/specs/SPEC_*.md`
- **Architecture Decisions**: `docs/architecture/ADR_*.md`
- **Models**: `apps/[app_name]/models.py`
- **Views**: `apps/[app_name]/views.py`
- **Tests**: `apps/[app_name]/tests/`
- **Templates**: `apps/[app_name]/templates/[app_name]/`
- **API docs**: `docs/api/`

## Tips for Effective Collaboration

1. **Be specific**: "Add user authentication" vs "Implement JWT-based auth with refresh tokens"
2. **Reference specs**: "Implement the workout logging feature from specs/SPEC_001_workout_logging.md"
3. **Request reviews**: "Review this implementation for security issues"
4. **Ask for tests**: "Add comprehensive tests for the exercise model"
5. **Seek explanations**: "Explain the tradeoffs between approach A and B"

## Current Project Status

**Phase**: Initial setup
**Next Steps**: Define core domain models and initial features

See `docs/specs/` for planned features and their implementation status.
