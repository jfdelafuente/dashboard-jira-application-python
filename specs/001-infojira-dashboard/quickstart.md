# Quickstart Guide: InfoJira Dashboard

**Feature**: InfoJira Dashboard
**Date**: 2025-12-22
**Purpose**: Setup and run instructions for local development

## Overview

This guide walks you through setting up the InfoJira Dashboard application on your local machine for development and testing. The application uses Flask (Python), Tailwind CSS, and Chart.js with mock JIRA data.

**Prerequisites**:
- Python 3.11 or higher
- Node.js 18+ and npm (for Tailwind CSS compilation)
- Git
- Code editor (VS Code recommended)

**Estimated Setup Time**: 15-20 minutes

---

## Step 1: Clone the Repository

```bash
git clone <repository-url>
cd dashboard-jira-application-python

# Switch to the feature branch
git checkout 001-infojira-dashboard
```

---

## Step 2: Backend Setup (Python/Flask)

### 2.1 Create Virtual Environment

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Windows:
venv\Scripts\activate

# On macOS/Linux:
source venv/bin/activate
```

You should see `(venv)` in your terminal prompt indicating the virtual environment is active.

### 2.2 Install Python Dependencies

```bash
# Install production dependencies
pip install -r requirements.txt

# Install development dependencies (includes pytest, black, flake8, mypy)
pip install -r requirements-dev.txt
```

**Main Dependencies**:
- Flask 3.0+ (web framework)
- SQLAlchemy 2.0+ (ORM)
- pytest 7.4+ (testing)
- python-dotenv (environment variables)

### 2.3 Configure Environment Variables

```bash
# Copy example environment file
cp .env.example .env

# Edit .env with your preferred text editor
# Example values for local development:
```

**.env contents**:

```ini
# Application Configuration
FLASK_APP=run.py
FLASK_ENV=development
SECRET_KEY=dev-secret-key-change-in-production

# Database Configuration
DATABASE_URL=sqlite:///dev_infojira.db

# Mock Data Configuration
USE_MOCK_DATA=true
MOCK_DATA_PATH=backend/src/static/data/mock_jira.json

# Server Configuration
HOST=0.0.0.0
PORT=5000
DEBUG=true
```

### 2.4 Initialize Database

```bash
# Navigate to backend directory
cd backend

# Initialize database and run migrations
python -c "from src.app import create_app; from src.models.database import db; app = create_app('development'); app.app_context().push(); db.create_all(); print('Database initialized successfully')"
```

**Note**: For production, use Alembic for database migrations:

```bash
# Initialize Alembic (first time only)
alembic init migrations

# Create migration
alembic revision --autogenerate -m "Initial schema"

# Apply migration
alembic upgrade head
```

---

## Step 3: Frontend Setup (Tailwind CSS)

### 3.1 Install Node Dependencies

```bash
# Navigate to frontend directory
cd frontend

# Install dependencies
npm install
```

**Main Dependencies**:
- tailwindcss 3.4+
- postcss 8.4+
- autoprefixer 10.4+

### 3.2 Build Tailwind CSS

```bash
# One-time build
npm run build:css

# OR: Watch mode for development (runs in background)
npm run watch:css
```

**Output**: Compiled CSS will be generated at `backend/src/static/css/output.css`

**Tip**: Keep `npm run watch:css` running in a separate terminal during development so CSS updates automatically when you modify templates.

---

## Step 4: Run the Application

### 4.1 Start Flask Development Server

```bash
# From project root
python run.py

# OR: Using Flask CLI
flask run
```

**Expected Output**:

```
 * Serving Flask app 'src.app'
 * Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://127.0.0.1:5000
Press CTRL+C to quit
```

### 4.2 Access the Dashboard

Open your web browser and navigate to:

```
http://localhost:5000
```

You should see the InfoJira Dashboard with:
- Project selector dropdown (SUPP, INFRA, WEBAPP)
- Four KPI cards (initially empty until project selected)
- Charts section (bar chart and pie chart)
- Issues table

**Test**: Select "SUPP" from the project dropdown and verify:
- KPI cards update with values
- Bar chart shows issue distribution by status
- Pie chart shows priority distribution
- Issues table displays 8 issues

---

## Step 5: Verify Installation

### 5.1 Run Tests

```bash
# Run all tests
pytest

# Run with coverage report
pytest --cov=backend/src --cov-report=html

# Run only unit tests
pytest backend/tests/unit/

# Run specific test file
pytest backend/tests/unit/test_kpi_calculator.py -v
```

**Expected Result**: All tests should pass with coverage ≥80%

### 5.2 Run Code Quality Checks

```bash
# Format code with black
black backend/src backend/tests

# Check PEP 8 compliance
flake8 backend/src backend/tests

# Run type checking
mypy backend/src
```

### 5.3 Manual Testing Checklist

- [ ] Dashboard loads within 2 seconds
- [ ] Project selector displays all projects (SUPP, INFRA, WEBAPP)
- [ ] KPIs update when project selected (<500ms)
- [ ] Critical issues KPI card is visually highlighted (red/orange)
- [ ] Bar chart displays issue status distribution
- [ ] Pie chart displays priority distribution
- [ ] Chart tooltips show on hover
- [ ] Issues table displays all columns correctly
- [ ] Table search box filters issues in real-time
- [ ] Table sorting works by clicking column headers
- [ ] Issue type and AP Area filters update all dashboard elements
- [ ] Responsive design works on mobile view (resize browser to <768px)
- [ ] No JavaScript errors in browser console

---

## Step 6: Development Workflow

### 6.1 Project Structure Overview

```text
backend/src/
├── app.py              # Flask application factory
├── config.py           # Configuration (dev/test/prod)
├── models/             # Database models (Project, Issue)
├── services/           # Business logic (KPI calculation, filtering)
├── routes/             # Flask routes (dashboard, API endpoints)
├── templates/          # Jinja2 HTML templates
└── static/             # CSS, JavaScript, mock data

backend/tests/
├── unit/               # Unit tests (services, models)
├── integration/        # Integration tests (routes, database)
└── contract/           # Contract tests (JIRA API compatibility)
```

### 6.2 Common Development Tasks

**Add a new KPI**:

1. Update `services/kpi_calculator.py` with new calculation method
2. Write unit test in `tests/unit/test_kpi_calculator.py`
3. Update `routes/api.py` to include new KPI in response
4. Update `templates/components/kpi_card.html` to display new KPI
5. Run tests: `pytest backend/tests/unit/test_kpi_calculator.py -v`

**Add a new filter**:

1. Update `services/filter_service.py` with filter logic
2. Write unit test
3. Update `routes/api.py` to accept new query parameter
4. Update `templates/components/filters.html` with new dropdown
5. Update `static/js/filters.js` to handle new filter
6. Run integration tests: `pytest backend/tests/integration/`

**Modify Tailwind styles**:

1. Edit HTML templates with Tailwind utility classes
2. Ensure `npm run watch:css` is running (auto-rebuilds CSS)
3. Refresh browser to see changes
4. For custom styles, edit `frontend/src/input.css` and add to `tailwind.config.js`

### 6.3 Debugging Tips

**Flask Debug Mode**:
- Set `FLASK_ENV=development` in `.env`
- Detailed error pages with stack traces
- Auto-reload on code changes

**Flask Debug Toolbar** (installed in dev dependencies):

```python
# backend/src/app.py
if app.config['DEBUG']:
    from flask_debugtoolbar import DebugToolbarExtension
    toolbar = DebugToolbarExtension(app)
```

Access toolbar at bottom of page to see:
- SQL queries and execution time
- Template rendering time
- Request/response details

**Browser DevTools**:
- Open with F12 (Chrome/Firefox)
- Console tab: Check for JavaScript errors
- Network tab: Monitor AJAX requests and response times
- Performance tab: Profile page load and render times

**Python Debugger**:

```python
# Add breakpoint in code
import pdb; pdb.set_trace()

# Or use VS Code debugger
# Create .vscode/launch.json:
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: Flask",
      "type": "python",
      "request": "launch",
      "module": "flask",
      "env": {
        "FLASK_APP": "run.py",
        "FLASK_ENV": "development"
      },
      "args": ["run", "--no-debugger"],
      "jinja": true
    }
  ]
}
```

---

## Step 7: Database Management

### 7.1 SQLite (Development)

**View database**:

```bash
# Install SQLite browser (optional)
# Download from: https://sqlitebrowser.org/

# OR: Use command line
sqlite3 backend/dev_infojira.db

# Run SQL queries
sqlite> SELECT * FROM projects;
sqlite> SELECT key, summary, status FROM issues WHERE project_id = 1;
sqlite> .quit
```

**Reset database**:

```bash
# Delete database file
rm backend/dev_infojira.db

# Recreate
cd backend
python -c "from src.app import create_app; from src.models.database import db; app = create_app('development'); app.app_context().push(); db.create_all()"
```

### 7.2 PostgreSQL (Production)

**Local PostgreSQL setup** (optional for testing production config):

```bash
# Install PostgreSQL 15+
# Download from: https://www.postgresql.org/download/

# Create database
createdb infojira_dev

# Update .env
DATABASE_URL=postgresql://username:password@localhost:5432/infojira_dev

# Run migrations
alembic upgrade head
```

---

## Step 8: Performance Testing

### 8.1 Load Testing with Locust

```bash
# Install locust
pip install locust

# Create locustfile.py (example):
from locust import HttpUser, task, between

class DashboardUser(HttpUser):
    wait_time = between(1, 3)

    @task
    def load_dashboard(self):
        self.client.get("/")

    @task(3)
    def get_project_data(self):
        self.client.get("/api/projects/SUPP/issues")

# Run load test
locust -f locustfile.py --host=http://localhost:5000
```

Access Locust web UI at `http://localhost:8089` and configure:
- Number of users: 50
- Spawn rate: 10 users/second

**Performance Targets** (from constitution):
- Dashboard load: <2s (p95)
- API responses: <500ms (p95)
- Filter operations: <500ms (p95)

### 8.2 Profiling

```bash
# Install profiling tools
pip install flask-profiler

# Enable in app.py (development only)
from flask_profiler import Profiler
profiler = Profiler()
profiler.init_app(app)

# Access profiler at:
http://localhost:5000/flask-profiler/
```

---

## Troubleshooting

### Common Issues

**1. "ModuleNotFoundError: No module named 'flask'"**
- Ensure virtual environment is activated: `source venv/bin/activate` (or `venv\Scripts\activate` on Windows)
- Reinstall dependencies: `pip install -r requirements.txt`

**2. "Database not found" error**
- Run database initialization (Step 2.4)
- Check `DATABASE_URL` in `.env` points to valid path

**3. Tailwind CSS not applying styles**
- Ensure `npm run build:css` or `npm run watch:css` has been run
- Check `backend/src/static/css/output.css` exists and is not empty
- Clear browser cache (Ctrl+Shift+R)

**4. Charts not rendering**
- Check browser console for JavaScript errors
- Ensure Chart.js is loaded (check `base.html` template)
- Verify API endpoint returns valid JSON data

**5. Slow performance / timeouts**
- Check database indexes are created (see `data-model.md`)
- Reduce mock data size for testing
- Enable Flask debug toolbar to identify slow queries

**6. Tests failing**
- Ensure test database is using in-memory SQLite: `SQLALCHEMY_DATABASE_URI = 'sqlite:///:memory:'`
- Check pytest configuration in `pyproject.toml` or `pytest.ini`
- Run tests with verbose output: `pytest -v -s`

---

## Next Steps

After completing this quickstart:

1. **Review Architecture**: Read `research.md` for technology decisions and patterns
2. **Understand Data Model**: Review `data-model.md` for entity schemas
3. **Explore API Contracts**: Check `contracts/dashboard-api.yaml` for endpoint specifications
4. **Generate Tasks**: Run `/speckit.tasks` to create implementation task list
5. **Start Development**: Follow TDD workflow (write tests first!)

---

## Additional Resources

- **Flask Documentation**: https://flask.palletsprojects.com/
- **Tailwind CSS Docs**: https://tailwindcss.com/docs
- **Chart.js Docs**: https://www.chartjs.org/docs/
- **SQLAlchemy Docs**: https://docs.sqlalchemy.org/
- **pytest Docs**: https://docs.pytest.org/

---

## Support

For issues or questions:
- Check existing issues in GitHub repository
- Review constitution compliance in `.specify/memory/constitution.md`
- Consult `plan.md` for architectural decisions

---

**Last Updated**: 2025-12-22
**Version**: 1.0.0
