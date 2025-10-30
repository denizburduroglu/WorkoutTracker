# ADR-001: Django Project Structure

**Status**: Accepted
**Date**: 2025-10-30
**Deciders**: Project Team
**Technical Story**: Initial project setup

## Context

We need to establish a clear, scalable project structure for the WorkoutTracker Django application. The structure should:
- Support multiple Django apps as the project grows
- Be intuitive for new developers and AI agents
- Follow Django and Python best practices
- Enable easy testing and maintenance

## Decision

We will use an app-centric structure with the following organization:

```
WorkoutTracker/
├── WorkoutTracker/          # Project configuration (settings, urls, wsgi)
├── apps/                    # All Django apps live here
│   ├── core/               # Core/shared functionality
│   ├── workouts/           # Workout tracking
│   └── [future apps]/      # Additional domain-specific apps
├── docs/                   # Project documentation
│   ├── specs/             # Feature specifications
│   ├── architecture/      # Architecture Decision Records
│   └── api/               # API documentation
├── templates/             # Global templates
├── static/                # Global static files
├── media/                 # User-uploaded files
├── tests/                 # Integration tests
└── scripts/               # Utility scripts
```

Each app will have this internal structure:
```
apps/app_name/
├── __init__.py
├── models.py              # Data models
├── views.py               # Views/controllers
├── urls.py                # URL routing
├── forms.py               # Forms (if needed)
├── serializers.py         # DRF serializers (if using REST API)
├── admin.py               # Admin configuration
├── apps.py                # App configuration
├── templates/
│   └── app_name/         # App-specific templates
├── static/
│   └── app_name/         # App-specific static files
├── tests/
│   ├── test_models.py
│   ├── test_views.py
│   └── test_integration.py
└── management/
    └── commands/         # Custom management commands
```

## Consequences

### Positive

- **Clear separation of concerns**: Each app handles a specific domain
- **Scalability**: Easy to add new apps without cluttering the root
- **Agent-friendly**: Clear patterns make it easy for AI to navigate and modify
- **Testability**: Tests are colocated with code, easy to find and run
- **Documentation-driven**: Specs and ADRs provide context for decisions

### Negative

- **More initial setup**: Need to create directory structure upfront
- **Import paths**: Slightly longer import paths (`apps.workouts.models` vs `workouts.models`)

### Neutral

- **Learning curve**: New team members need to understand the structure (mitigated by docs)

## Alternatives Considered

### Alternative 1: Flat Structure

Keep all apps at the root level alongside manage.py

**Why not chosen**: Becomes cluttered as more apps are added. Harder to distinguish between project-level and app-level code.

### Alternative 2: Monolithic App

Keep all functionality in a single Django app

**Why not chosen**: Doesn't scale well. Makes it harder to reason about different domains. Testing becomes more complex.

### Alternative 3: Microservices

Split into multiple Django projects/services

**Why not chosen**: Overkill for the current project size. Adds complexity in deployment, data consistency, and development workflow.

## References

- Two Scoops of Django (Django best practices book)
- Django documentation: https://docs.djangoproject.com/en/5.2/intro/reusable-apps/
- Cookiecutter Django: https://github.com/cookiecutter/cookiecutter-django
