# Setup Guide

## Prerequisites

- Python 3.11+
- Git
- Virtual environment

## Initial Setup

### 1. Clone and Navigate

```bash
cd WorkoutTracker
```

### 2. Create Virtual Environment

```bash
# Windows
python -m venv .venv
.venv\Scripts\activate

# Unix/macOS
python -m venv .venv
source .venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### 4. Environment Configuration

Create a `.env` file in the root directory:

```env
SECRET_KEY=your-secret-key-here
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1
DATABASE_URL=sqlite:///db.sqlite3
```

### 5. Database Setup

```bash
python manage.py migrate
python manage.py createsuperuser
```

### 6. Run Development Server

```bash
python manage.py runserver
```

Visit http://localhost:8000/admin to access the admin interface.

## Development Tools

### Code Formatting

```bash
black .
```

### Linting

```bash
flake8
```

### Type Checking

```bash
mypy apps/
```

### Running Tests

```bash
# All tests
pytest

# Specific app
pytest apps/workouts/

# With coverage
pytest --cov=apps --cov-report=html

# Fast tests only (skip slow/integration)
pytest -m "not slow"
```

## Creating New Apps

```bash
# Create app directory
mkdir apps/app_name
cd apps/app_name

# Create Django app
python ../../manage.py startapp app_name .

# Create test directory
mkdir tests
touch tests/__init__.py
touch tests/test_models.py
touch tests/test_views.py

# Create template directory
mkdir -p templates/app_name
mkdir -p static/app_name
```

Then add the app to `INSTALLED_APPS` in `WorkoutTracker/settings.py`:

```python
INSTALLED_APPS = [
    # ...
    'apps.app_name',
]
```

## Common Issues

### Issue: ModuleNotFoundError

Solution: Ensure virtual environment is activated and dependencies are installed.

### Issue: Database locked

Solution: Close any other processes accessing the database, or delete `db.sqlite3` and re-migrate.

### Issue: Port already in use

Solution: Kill the process using port 8000 or run on a different port:
```bash
python manage.py runserver 8001
```

## Next Steps

1. Review `docs/AGENT_GUIDE.md` for collaboration practices
2. Read `docs/specs/SPEC_001_core_models.md` for the initial feature spec
3. Start implementing features following the spec-driven approach
