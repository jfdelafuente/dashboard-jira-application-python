# Research: InfoJira Dashboard

**Feature**: InfoJira Dashboard
**Date**: 2025-12-22
**Purpose**: Technology decisions, architectural patterns, and best practices research

## Overview

This document captures the research findings and technical decisions made during Phase 0 planning for the InfoJira Dashboard. All decisions are evaluated against the project constitution and performance requirements.

---

## 1. Flask Application Architecture

### Decision: Application Factory Pattern with Blueprints

**Chosen Approach**: Use Flask application factory pattern with Blueprint-based route organization

**Rationale**:
- **Testability**: Factory pattern enables multiple app instances with different configs (test, dev, prod)
- **Modularity**: Blueprints separate dashboard routes from API routes
- **Configuration**: Environment-specific settings (SQLite vs PostgreSQL) easily managed
- **Scalability**: Supports future addition of admin panel, authentication, etc.

**Alternatives Considered**:
- **Single app.py file**: Rejected - becomes unwieldy and hard to test as features grow
- **Flask-RESTful**: Rejected - overkill for simple AJAX endpoints, adds unnecessary dependency

**Implementation Pattern**:

```python
# src/app.py
def create_app(config_name='development'):
    app = Flask(__name__)
    app.config.from_object(config[config_name])

    db.init_app(app)

    from .routes.dashboard import dashboard_bp
    from .routes.api import api_bp

    app.register_blueprint(dashboard_bp)
    app.register_blueprint(api_bp, url_prefix='/api')

    return app
```

**Constitutional Alignment**: ✅ Supports Principle IV (Code Quality & Maintainability) through clear separation of concerns

---

## 2. Data Provider Abstraction

### Decision: Abstract Base Class for Data Sources

**Chosen Approach**: Define `DataProvider` ABC with implementations for mock JSON and future JIRA API

**Rationale**:
- **Future-proofing**: Enables swapping mock data for live JIRA API without changing business logic
- **Testability**: Mock provider can be replaced with test fixtures
- **Contract testing**: Abstract interface defines contract that JIRA integration must fulfill
- **Single Responsibility**: Data fetching separate from KPI calculation and filtering

**Alternatives Considered**:
- **Direct JSON file reading**: Rejected - hard-codes data source, violates Open/Closed Principle
- **Conditional logic (if USE_MOCK)**: Rejected - creates code branches, harder to test

**Implementation Pattern**:

```python
# src/services/data_provider.py
from abc import ABC, abstractmethod
from typing import List, Optional
from ..models.project import Project
from ..models.issue import Issue

class DataProvider(ABC):
    @abstractmethod
    def get_projects(self) -> List[Project]:
        """Retrieve all available projects."""
        pass

    @abstractmethod
    def get_issues(self, project_key: str,
                   issue_type: Optional[str] = None,
                   ap_area: Optional[str] = None) -> List[Issue]:
        """Retrieve issues with optional filters."""
        pass

# src/services/mock_data_service.py
class MockDataService(DataProvider):
    def __init__(self, json_file_path: str):
        self.data = self._load_json(json_file_path)

    def get_projects(self) -> List[Project]:
        # Implementation
        pass

    def get_issues(self, project_key, issue_type=None, ap_area=None):
        # Implementation
        pass
```

**Constitutional Alignment**: ✅ Supports Principle V (Integration & Contract Testing) and Principle IV (Maintainability)

---

## 3. Database Strategy

### Decision: Dual Database Support with SQLAlchemy ORM

**Chosen Approach**: SQLAlchemy ORM with SQLite (dev/test) and PostgreSQL (production)

**Rationale**:
- **Development speed**: SQLite requires no setup, ideal for local development
- **Production robustness**: PostgreSQL handles concurrent users and larger datasets
- **ORM benefits**: Database-agnostic queries, automatic migrations via Alembic
- **Testing**: In-memory SQLite for fast test execution
- **Constitutional requirement**: Supports <100ms query performance through indexes

**Alternatives Considered**:
- **PostgreSQL only**: Rejected - slows local development, requires Docker/server setup
- **No database (JSON only)**: Rejected - limits query performance and filtering capabilities
- **MongoDB**: Rejected - relational data structure (projects → issues) better suited for RDBMS

**Configuration Pattern**:

```python
# src/config.py
class Config:
    SQLALCHEMY_TRACK_MODIFICATIONS = False

class DevelopmentConfig(Config):
    SQLALCHEMY_DATABASE_URI = 'sqlite:///dev_infojira.db'
    DEBUG = True

class TestConfig(Config):
    SQLALCHEMY_DATABASE_URI = 'sqlite:///:memory:'
    TESTING = True

class ProductionConfig(Config):
    SQLALCHEMY_DATABASE_URI = os.getenv('DATABASE_URL', 'postgresql://user:pass@localhost/infojira')
    DEBUG = False
```

**Indexing Strategy** (to meet <100ms query target):
- Index on `issue.project_id` (foreign key)
- Index on `issue.status` (frequent filter)
- Index on `issue.priority` (frequent filter)
- Composite index on `(issue.project_id, issue.status)` for KPI queries

**Constitutional Alignment**: ✅ Supports Principle III (Performance by Design) through proper indexing and query optimization

---

## 4. Frontend Architecture

### Decision: Server-Side Rendering with Progressive Enhancement

**Chosen Approach**: Jinja2 templates with Tailwind CSS, Chart.js for charts, vanilla JavaScript for interactivity

**Rationale**:
- **Performance**: Initial page load <2s with server-rendered HTML (no framework overhead)
- **Accessibility**: WCAG AA compliance easier with semantic HTML
- **Simplicity**: No build complexity (React/Vue), faster development
- **Progressive enhancement**: Core functionality works without JavaScript, AJAX enhances UX
- **SEO-friendly**: Server-rendered content (future consideration)

**Alternatives Considered**:
- **React SPA**: Rejected - adds complexity, build process, initial load overhead for limited benefit
- **Vue.js**: Rejected - same concerns as React for a dashboard-focused app
- **Htmx**: Considered - but vanilla AJAX is sufficient for filter updates

**JavaScript Organization**:
- **dashboard.js**: Main controller, coordinates filters/charts/table
- **charts.js**: Chart.js configuration and rendering
- **filters.js**: Filter interaction and AJAX requests
- **table.js**: Sorting, pagination, search logic

**Constitutional Alignment**: ✅ Supports Principle II (UX Excellence) and Principle III (Performance by Design)

---

## 5. Tailwind CSS Integration

### Decision: JIT Mode with PostCSS Build Process

**Chosen Approach**: Tailwind CSS v3.4+ with Just-In-Time compiler

**Rationale**:
- **Performance**: JIT generates only used classes, minimal CSS bundle (~10KB)
- **Developer experience**: All Tailwind utilities available without config
- **Responsive design**: Built-in breakpoints (sm:, md:, lg:, xl:) for mobile/tablet/desktop
- **Accessibility**: Color contrast utilities support WCAG AA compliance
- **Customization**: Easy to extend with project-specific colors/spacing

**Build Process**:

```json
// frontend/package.json
{
  "scripts": {
    "build:css": "tailwindcss -i ./src/input.css -o ../backend/src/static/css/output.css",
    "watch:css": "tailwindcss -i ./src/input.css -o ../backend/src/static/css/output.css --watch"
  },
  "devDependencies": {
    "tailwindcss": "^3.4.0",
    "autoprefixer": "^10.4.16",
    "postcss": "^8.4.32"
  }
}
```

**Responsive Breakpoints**:
- `sm`: 640px (mobile landscape)
- `md`: 768px (tablet)
- `lg`: 1024px (desktop)
- `xl`: 1280px (large desktop)

**Constitutional Alignment**: ✅ Supports Principle II (UX Excellence) through responsive design and accessibility

---

## 6. Chart.js Configuration

### Decision: Chart.js with Custom Plugins for Performance

**Chosen Approach**: Chart.js v4.4+ with lazy loading and data decimation

**Rationale**:
- **Performance**: Meets <1s rendering target for 500 issues through data sampling
- **Interactivity**: Built-in tooltips, hover effects, responsive sizing
- **Accessibility**: ARIA labels, keyboard navigation support
- **Customization**: Plugin system for custom behaviors
- **No dependencies**: Pure JavaScript, no jQuery required

**Performance Optimization**:

```javascript
// charts.js
const chartConfig = {
  animation: {
    duration: 400  // Faster than default 1000ms
  },
  plugins: {
    decimation: {
      enabled: true,
      algorithm: 'lttb',  // Largest Triangle Three Buckets
      samples: 100  // Reduce data points for large datasets
    }
  },
  responsive: true,
  maintainAspectRatio: false
};
```

**Chart Types**:
- **Bar Chart** (Status): Vertical bars, 4 categories, tooltips show exact count
- **Pie Chart** (Priority): Doughnut style, percentage labels, color-coded by urgency

**Constitutional Alignment**: ✅ Supports Principle III (Performance by Design) and Principle II (UX Excellence)

---

## 7. KPI Calculation Strategy

### Decision: Service Layer with Cached Calculations

**Chosen Approach**: Dedicated `KPICalculator` service with query-level optimization

**Rationale**:
- **Testability**: Business logic isolated from routes and models
- **Performance**: Single database query per KPI using aggregations
- **Accuracy**: 100% accuracy requirement met through tested calculation logic
- **Maintainability**: KPI formulas documented and versioned

**Calculation Logic**:

```python
# src/services/kpi_calculator.py
class KPICalculator:
    def calculate_open_issues(self, issues: List[Issue]) -> int:
        """Count issues with status not in (Resolved, Cancelled, Closed)."""
        return len([i for i in issues if i.status not in ['Resolved', 'Cancelled', 'Closed']])

    def calculate_critical_issues(self, issues: List[Issue]) -> int:
        """Count issues with priority Critical or Blocker."""
        return len([i for i in issues if i.priority in ['Critical', 'Blocker']])

    def calculate_avg_resolution_time(self, issues: List[Issue]) -> Optional[float]:
        """Calculate average time between created and resolved dates (in hours).
        Returns None if no resolved issues."""
        resolved = [i for i in issues if i.status in ['Resolved', 'Closed'] and i.resolved_at]
        if not resolved:
            return None
        total_hours = sum((i.resolved_at - i.created_at).total_seconds() / 3600 for i in resolved)
        return total_hours / len(resolved)

    def calculate_overdue_sla(self, issues: List[Issue]) -> int:
        """Count issues where current date > sla_due_date."""
        now = datetime.utcnow()
        return len([i for i in issues if i.sla_due_date and now > i.sla_due_date])
```

**Performance Optimization**:
- Use database aggregations (COUNT, AVG) when possible instead of loading all records
- Cache results for 60 seconds to avoid recalculation on repeated requests

**Constitutional Alignment**: ✅ Supports Principle I (Test-First Development) through isolated, testable logic

---

## 8. Filter and Search Implementation

### Decision: Client-Side Filtering with Server-Side Fallback

**Chosen Approach**: Hybrid filtering - client-side for table, server-side for KPIs/charts

**Rationale**:
- **Performance**: Client-side table filtering meets <200ms sort requirement
- **User experience**: Instant feedback for search/sort without network latency
- **Accuracy**: Server-side KPI recalculation ensures correctness
- **Scalability**: Server handles large datasets (>1000 issues) via pagination

**Implementation Strategy**:

1. **Project/Issue Type/AP Area filters**: Trigger server request, update all dashboard elements
2. **Table search/priority filter**: Client-side only, filters already-loaded table data
3. **Sorting**: Client-side using JavaScript array methods
4. **Pagination**: Server-side for datasets >50 issues

**Search Algorithm** (client-side):

```javascript
// table.js
function filterTable(searchText, priority) {
  const rows = document.querySelectorAll('.issue-row');
  rows.forEach(row => {
    const matchesSearch = searchText === '' ||
      ['key', 'summary', 'application', 'assigned_group']
        .some(field => row.dataset[field].toLowerCase().includes(searchText.toLowerCase()));

    const matchesPriority = priority === 'all' || row.dataset.priority === priority;

    row.style.display = (matchesSearch && matchesPriority) ? '' : 'none';
  });
}
```

**Constitutional Alignment**: ✅ Supports Principle III (Performance by Design) through optimized filtering strategy

---

## 9. Mock Data Structure

### Decision: JIRA REST API v3 Format

**Chosen Approach**: Mock JSON matching JIRA Cloud REST API v3 response structure

**Rationale**:
- **Future compatibility**: Minimal code changes when integrating live JIRA API
- **Contract testing**: Test suite validates against real JIRA schema
- **Completeness**: Includes all fields needed for KPIs and display
- **Realistic**: Supports development and testing with representative data

**Sample Structure**:

```json
{
  "projects": [
    {
      "id": "10001",
      "key": "SUPP",
      "name": "Technical Support"
    }
  ],
  "issues": [
    {
      "id": "10101",
      "key": "SUPP-123",
      "fields": {
        "summary": "Application crashes on startup",
        "issuetype": { "name": "Bug" },
        "priority": { "name": "Critical" },
        "status": { "name": "In Progress" },
        "created": "2025-12-01T10:00:00.000+0000",
        "resolutiondate": null,
        "customfield_10050": { "name": "Mobile App" },  // AP Area
        "customfield_10051": "2025-12-05T18:00:00.000+0000",  // SLA Due
        "assignee": { "displayName": "Support Team A" }
      }
    }
  ]
}
```

**Data Volume for Testing**:
- Project 1: 75 issues (mixed statuses, priorities)
- Project 2: 30 issues (mostly resolved)
- Project 3: 150 issues (large dataset for performance testing)

**Constitutional Alignment**: ✅ Supports Principle V (Integration & Contract Testing)

---

## 10. Testing Strategy

### Decision: Test Pyramid with pytest

**Chosen Approach**:
- **70% Unit tests**: Fast, isolated tests for services and models
- **25% Integration tests**: Route and database interaction tests
- **5% Contract tests**: JIRA API schema validation

**Rationale**:
- **Speed**: Unit tests run in <1s, full suite in <10s
- **Coverage**: 80% minimum, 100% for KPI calculation logic
- **Confidence**: Integration tests catch configuration issues
- **Future-proofing**: Contract tests ensure JIRA API compatibility

**Test Organization**:

```python
# tests/conftest.py
@pytest.fixture
def app():
    app = create_app('testing')
    with app.app_context():
        db.create_all()
        yield app
        db.drop_all()

@pytest.fixture
def client(app):
    return app.test_client()

@pytest.fixture
def sample_issues():
    return [
        Issue(key='TEST-1', priority='Critical', status='Open', ...),
        Issue(key='TEST-2', priority='High', status='Resolved', ...)
    ]
```

**Coverage Requirements**:
- `kpi_calculator.py`: 100% (critical business logic)
- `data_provider.py`: 100% (contract interface)
- `routes/`: 90% (API endpoints)
- `models/`: 85% (data layer)
- Overall: 80% minimum

**Constitutional Alignment**: ✅ Supports Principle I (Test-First Development NON-NEGOTIABLE)

---

## 11. Error Handling and Logging

### Decision: Structured Logging with Flask Error Handlers

**Chosen Approach**: Python `logging` module with JSON formatter, custom Flask error handlers

**Rationale**:
- **Observability**: JSON logs easily parsed by log aggregation tools
- **User experience**: User-friendly error pages, technical details logged
- **Debugging**: Correlation IDs track requests through the system
- **Monitoring**: Error rates tracked for alerting

**Logging Configuration**:

```python
# src/app.py
import logging
from pythonjsonlogger import jsonlogger

def configure_logging(app):
    logHandler = logging.StreamHandler()
    formatter = jsonlogger.JsonFormatter()
    logHandler.setFormatter(formatter)
    app.logger.addHandler(logHandler)
    app.logger.setLevel(logging.INFO)

@app.errorhandler(404)
def not_found(error):
    app.logger.warning('Page not found', extra={'url': request.url})
    return render_template('404.html'), 404

@app.errorhandler(500)
def server_error(error):
    app.logger.error('Server error', extra={'error': str(error)}, exc_info=True)
    return render_template('500.html'), 500
```

**Constitutional Alignment**: ✅ Supports Principle II (UX Excellence) and constitutional Monitoring & Observability standards

---

## 12. Performance Monitoring

### Decision: Flask-DebugToolbar (dev) + Custom Metrics (prod)

**Chosen Approach**: Development profiling with Flask-DebugToolbar, production metrics endpoint

**Rationale**:
- **Development**: Visual profiling of SQL queries, template rendering
- **Production**: Prometheus-compatible metrics endpoint
- **Validation**: Verify <2s load, <500ms filter targets

**Metrics to Track**:
- Request duration (p50, p95, p99)
- Database query time
- Chart rendering time (client-side performance API)
- Error rate by endpoint
- Active users (concurrent requests)

**Constitutional Alignment**: ✅ Supports Principle III (Performance by Design) monitoring requirement

---

## Summary of Decisions

| Area | Decision | Primary Constitutional Alignment |
|------|----------|----------------------------------|
| Flask Architecture | Application Factory + Blueprints | Principle IV (Maintainability) |
| Data Abstraction | DataProvider ABC | Principle V (Contract Testing) |
| Database | SQLAlchemy (SQLite/PostgreSQL) | Principle III (Performance) |
| Frontend | Server-Side Rendering + Tailwind | Principle II (UX Excellence) |
| Charts | Chart.js with Optimization | Principle III (Performance) |
| KPI Calculation | Dedicated Service Layer | Principle I (Test-First) |
| Filtering | Hybrid Client/Server | Principle III (Performance) |
| Mock Data | JIRA API v3 Format | Principle V (Contract Testing) |
| Testing | pytest Test Pyramid (80% coverage) | Principle I (Test-First NON-NEGOTIABLE) |
| Logging | Structured JSON Logging | Constitutional Monitoring Standards |
| Performance | Profiling + Metrics Endpoint | Principle III (Performance Monitoring) |

---

## Next Steps

Phase 1 Design:
1. Create `data-model.md` with entity schemas
2. Generate OpenAPI spec in `contracts/dashboard-api.yaml`
3. Create mock JIRA data in `contracts/jira-mock-data.json`
4. Write `quickstart.md` with setup instructions
5. Update agent context with technology stack

All research findings support constitutional compliance. No violations identified.
