# WorkoutTracker

A Django-based workout tracking application built with spec-driven development practices.

## Project Structure

```
WorkoutTracker/
├── WorkoutTracker/          # Project configuration
├── apps/                    # Django apps
│   └── [future apps here]
├── docs/                    # Project documentation
│   ├── specs/              # Feature specifications
│   ├── architecture/       # Architecture decisions
│   └── api/                # API documentation
├── templates/              # Global templates
├── static/                 # Global static files
├── tests/                  # Integration tests
└── scripts/                # Utility scripts
```

## Development Approach

This project follows **spec-driven development**:
1. Write specifications first (see `docs/specs/`)
2. Review and refine specs before implementation
3. Implement features according to approved specs
4. Iterate based on testing and feedback

## Getting Started

```bash
# Activate virtual environment
source .venv/bin/activate  # Unix
# or
.venv\Scripts\activate  # Windows

# Install dependencies
pip install -r requirements.txt

# Run migrations
python manage.py migrate

# Start development server
python manage.py runserver
```

## Working with AI Agents

See `docs/AGENT_GUIDE.md` for best practices when iterating with AI assistants.
