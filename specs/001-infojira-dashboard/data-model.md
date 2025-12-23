# Data Model: InfoJira Dashboard

**Feature**: InfoJira Dashboard
**Date**: 2025-12-22
**Purpose**: Entity schemas, relationships, and validation rules

## Overview

This document defines the data model for the InfoJira Dashboard application. The model supports both mock JSON data (initial implementation) and future database storage (SQLite/PostgreSQL). All entities are designed to mirror JIRA REST API v3 response structure for seamless integration.

---

## Entity Relationship Diagram

```text
┌─────────────────┐
│    Project      │
│─────────────────│
│ id: int (PK)    │
│ key: str        │◄──┐
│ name: str       │   │
└─────────────────┘   │
                      │ 1:N
                      │
┌─────────────────────┴──────────────────┐
│             Issue                      │
│────────────────────────────────────────│
│ id: int (PK)                           │
│ key: str (unique)                      │
│ project_id: int (FK → Project.id)      │
│ summary: str                           │
│ issue_type: str                        │
│ priority: str                          │
│ status: str                            │
│ ap_area: str (nullable)                │
│ assigned_group: str (nullable)         │
│ created_at: datetime                   │
│ resolved_at: datetime (nullable)       │
│ sla_due_date: datetime (nullable)      │
└────────────────────────────────────────┘
```

**Relationship Summary**:
- One Project has many Issues (1:N)
- Issues belong to exactly one Project
- No self-referential relationships or many-to-many relationships

---

## Entities

### 1. Project

Represents a JIRA project containing related issues.

**Fields**:

| Field Name | Type | Constraints | Description |
|------------|------|-------------|-------------|
| `id` | Integer | PRIMARY KEY, AUTO_INCREMENT | Internal database ID |
| `key` | String(10) | NOT NULL, UNIQUE | JIRA project key (e.g., "SUPP", "INFRA") |
| `name` | String(100) | NOT NULL | Human-readable project name (e.g., "Technical Support") |

**Validation Rules**:
- `key`: Must be uppercase, 2-10 characters, alphanumeric only
- `name`: Cannot be empty string, max 100 characters

**Indexes**:
- PRIMARY KEY on `id`
- UNIQUE INDEX on `key` (for lookup by project key)

**Sample Data**:

```python
Project(id=1, key='SUPP', name='Technical Support')
Project(id=2, key='INFRA', name='Infrastructure')
Project(id=3, key='WEBAPP', name='Web Application')
```

---

### 2. Issue

Represents a JIRA issue/ticket with metadata for KPI calculation and display.

**Fields**:

| Field Name | Type | Constraints | Description |
|------------|------|-------------|-------------|
| `id` | Integer | PRIMARY KEY, AUTO_INCREMENT | Internal database ID |
| `key` | String(20) | NOT NULL, UNIQUE | JIRA issue key (e.g., "SUPP-123") |
| `project_id` | Integer | NOT NULL, FOREIGN KEY (Project.id) | Reference to parent project |
| `summary` | String(500) | NOT NULL | Brief description of the issue |
| `issue_type` | String(50) | NOT NULL | Type of issue (Bug, Task, Story, Epic, etc.) |
| `priority` | String(20) | NOT NULL | Priority level (Critical, Blocker, High, Medium, Low) |
| `status` | String(50) | NOT NULL | Current workflow status (In Progress, Resolved, Cancelled, Closed, etc.) |
| `ap_area` | String(100) | NULLABLE | Application/functional area (e.g., "Mobile App", "API Gateway") |
| `assigned_group` | String(100) | NULLABLE | Team or group assigned to the issue (e.g., "Support Team A") |
| `created_at` | DateTime | NOT NULL | Timestamp when issue was created (UTC) |
| `resolved_at` | DateTime | NULLABLE | Timestamp when issue was resolved (NULL if not resolved) |
| `sla_due_date` | DateTime | NULLABLE | SLA deadline (NULL if no SLA defined) |

**Validation Rules**:
- `key`: Must match pattern `{PROJECT_KEY}-{NUMBER}` (e.g., "SUPP-123")
- `priority`: Must be one of: Critical, Blocker, High, Medium, Low
- `status`: Must be one of: Open, In Progress, Resolved, Cancelled, Closed, Pending, Blocked
- `created_at`: Cannot be in the future
- `resolved_at`: Must be >= `created_at` if not NULL
- `sla_due_date`: Typically >= `created_at` if not NULL (warnings if already overdue)

**Derived Fields** (calculated, not stored):
- `is_open`: True if status NOT IN (Resolved, Cancelled, Closed)
- `is_critical`: True if priority IN (Critical, Blocker)
- `is_overdue`: True if `sla_due_date` is not NULL and < current datetime
- `resolution_time_hours`: (`resolved_at` - `created_at`).total_seconds() / 3600 if resolved_at is not NULL

**Indexes**:
- PRIMARY KEY on `id`
- UNIQUE INDEX on `key`
- INDEX on `project_id` (foreign key, frequent join)
- INDEX on `status` (KPI filtering by status)
- INDEX on `priority` (KPI filtering by priority)
- COMPOSITE INDEX on `(project_id, status)` (optimizes KPI queries)

**Sample Data**:

```python
Issue(
    id=101,
    key='SUPP-123',
    project_id=1,
    summary='Application crashes on startup',
    issue_type='Bug',
    priority='Critical',
    status='In Progress',
    ap_area='Mobile App',
    assigned_group='Support Team A',
    created_at=datetime(2025, 12, 1, 10, 0, 0),
    resolved_at=None,
    sla_due_date=datetime(2025, 12, 5, 18, 0, 0)
)

Issue(
    id=102,
    key='SUPP-124',
    project_id=1,
    summary='Slow API response times',
    issue_type='Bug',
    priority='High',
    status='Resolved',
    ap_area='API Gateway',
    assigned_group='Support Team B',
    created_at=datetime(2025, 11, 28, 14, 30, 0),
    resolved_at=datetime(2025, 12, 2, 9, 15, 0),
    sla_due_date=datetime(2025, 12, 3, 14, 30, 0)
)
```

---

## Enumerations

### IssueType

Valid values for `Issue.issue_type`:

- Bug
- Task
- Story
- Epic
- Sub-task
- Improvement
- New Feature

**Note**: Additional types may exist in JIRA; system should handle unknown types gracefully (display as-is).

### Priority

Valid values for `Issue.priority`:

- Critical
- Blocker
- High
- Medium
- Low

**KPI Logic**: `is_critical` = priority IN (Critical, Blocker)

**UI Color Coding**:
- Critical/Blocker: Red (`bg-red-100 text-red-800`)
- High: Orange (`bg-orange-100 text-orange-800`)
- Medium: Yellow (`bg-yellow-100 text-yellow-800`)
- Low: Green (`bg-green-100 text-green-800`)

### Status

Valid values for `Issue.status`:

- Open
- In Progress
- Pending
- Blocked
- Resolved
- Closed
- Cancelled

**KPI Logic**:
- `is_open` = status NOT IN (Resolved, Cancelled, Closed)
- "Open issues" for KPI = all issues where `is_open` is True

**Chart Categories** (bar chart):
- In Progress (includes "In Progress", "Pending", "Blocked")
- Resolved
- Cancelled
- Closed

---

## SQLAlchemy Models

### Project Model

```python
# backend/src/models/project.py
from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import relationship
from .database import Base

class Project(Base):
    __tablename__ = 'projects'

    id = Column(Integer, primary_key=True)
    key = Column(String(10), unique=True, nullable=False, index=True)
    name = Column(String(100), nullable=False)

    # Relationship
    issues = relationship('Issue', back_populates='project', cascade='all, delete-orphan')

    def __repr__(self):
        return f'<Project {self.key}: {self.name}>'

    def to_dict(self):
        return {
            'id': self.id,
            'key': self.key,
            'name': self.name
        }
```

### Issue Model

```python
# backend/src/models/issue.py
from sqlalchemy import Column, Integer, String, DateTime, ForeignKey, Index
from sqlalchemy.orm import relationship
from datetime import datetime
from .database import Base

class Issue(Base):
    __tablename__ = 'issues'

    id = Column(Integer, primary_key=True)
    key = Column(String(20), unique=True, nullable=False, index=True)
    project_id = Column(Integer, ForeignKey('projects.id'), nullable=False, index=True)
    summary = Column(String(500), nullable=False)
    issue_type = Column(String(50), nullable=False)
    priority = Column(String(20), nullable=False, index=True)
    status = Column(String(50), nullable=False, index=True)
    ap_area = Column(String(100), nullable=True)
    assigned_group = Column(String(100), nullable=True)
    created_at = Column(DateTime, nullable=False)
    resolved_at = Column(DateTime, nullable=True)
    sla_due_date = Column(DateTime, nullable=True)

    # Relationship
    project = relationship('Project', back_populates='issues')

    # Composite index for KPI queries
    __table_args__ = (
        Index('idx_project_status', 'project_id', 'status'),
    )

    def __repr__(self):
        return f'<Issue {self.key}: {self.summary[:50]}>'

    @property
    def is_open(self) -> bool:
        """Check if issue is open (not resolved/cancelled/closed)."""
        return self.status not in ['Resolved', 'Cancelled', 'Closed']

    @property
    def is_critical(self) -> bool:
        """Check if issue is critical priority."""
        return self.priority in ['Critical', 'Blocker']

    @property
    def is_overdue(self) -> bool:
        """Check if issue is past SLA due date."""
        if not self.sla_due_date:
            return False
        return datetime.utcnow() > self.sla_due_date

    @property
    def resolution_time_hours(self) -> float | None:
        """Calculate resolution time in hours (None if not resolved)."""
        if not self.resolved_at:
            return None
        delta = self.resolved_at - self.created_at
        return delta.total_seconds() / 3600

    def to_dict(self):
        return {
            'id': self.id,
            'key': self.key,
            'project_id': self.project_id,
            'summary': self.summary,
            'issue_type': self.issue_type,
            'priority': self.priority,
            'status': self.status,
            'ap_area': self.ap_area,
            'assigned_group': self.assigned_group,
            'created_at': self.created_at.isoformat() if self.created_at else None,
            'resolved_at': self.resolved_at.isoformat() if self.resolved_at else None,
            'sla_due_date': self.sla_due_date.isoformat() if self.sla_due_date else None,
            'is_open': self.is_open,
            'is_critical': self.is_critical,
            'is_overdue': self.is_overdue
        }
```

---

## Mock JSON Data Structure

For initial development, mock data will be stored in `backend/src/static/data/mock_jira.json` matching JIRA REST API v3 format:

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
        "project": {
          "id": "10001",
          "key": "SUPP"
        },
        "summary": "Application crashes on startup",
        "issuetype": {
          "name": "Bug"
        },
        "priority": {
          "name": "Critical"
        },
        "status": {
          "name": "In Progress"
        },
        "customfield_10050": {
          "value": "Mobile App"
        },
        "customfield_10060": "Support Team A",
        "created": "2025-12-01T10:00:00.000+0000",
        "resolutiondate": null,
        "customfield_10070": "2025-12-05T18:00:00.000+0000"
      }
    }
  ]
}
```

**Field Mapping** (JIRA API → Internal Model):

| JIRA API Field | Internal Field | Transformation |
|----------------|----------------|----------------|
| `id` | `id` | String → Integer |
| `key` | `key` | Direct mapping |
| `fields.project.id` | `project_id` | String → Integer |
| `fields.summary` | `summary` | Direct mapping |
| `fields.issuetype.name` | `issue_type` | Extract name |
| `fields.priority.name` | `priority` | Extract name |
| `fields.status.name` | `status` | Extract name |
| `fields.customfield_10050.value` | `ap_area` | Extract value (nullable) |
| `fields.customfield_10060` | `assigned_group` | Direct mapping (nullable) |
| `fields.created` | `created_at` | ISO 8601 → DateTime |
| `fields.resolutiondate` | `resolved_at` | ISO 8601 → DateTime (nullable) |
| `fields.customfield_10070` | `sla_due_date` | ISO 8601 → DateTime (nullable) |

---

## Data Access Patterns

### Common Queries

1. **Get all projects**:

```python
projects = session.query(Project).all()
```

2. **Get issues for a project with optional filters**:

```python
query = session.query(Issue).filter(Issue.project_id == project_id)
if issue_type:
    query = query.filter(Issue.issue_type == issue_type)
if ap_area:
    query = query.filter(Issue.ap_area == ap_area)
issues = query.all()
```

3. **Calculate open issues KPI**:

```python
open_count = session.query(func.count(Issue.id))\
    .filter(Issue.project_id == project_id)\
    .filter(~Issue.status.in_(['Resolved', 'Cancelled', 'Closed']))\
    .scalar()
```

4. **Calculate critical issues KPI**:

```python
critical_count = session.query(func.count(Issue.id))\
    .filter(Issue.project_id == project_id)\
    .filter(Issue.priority.in_(['Critical', 'Blocker']))\
    .scalar()
```

5. **Calculate average resolution time**:

```python
from sqlalchemy import func

avg_hours = session.query(
    func.avg(
        func.julianday(Issue.resolved_at) - func.julianday(Issue.created_at)
    ) * 24
).filter(Issue.project_id == project_id)\
 .filter(Issue.resolved_at.isnot(None))\
 .scalar()
```

6. **Get issues by status for bar chart**:

```python
status_counts = session.query(Issue.status, func.count(Issue.id))\
    .filter(Issue.project_id == project_id)\
    .group_by(Issue.status)\
    .all()
```

---

## Database Migrations

Using Alembic for schema versioning:

### Initial Migration (v1)

```python
# migrations/versions/001_initial_schema.py
def upgrade():
    op.create_table(
        'projects',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('key', sa.String(length=10), nullable=False),
        sa.Column('name', sa.String(length=100), nullable=False),
        sa.PrimaryKeyConstraint('id'),
        sa.UniqueConstraint('key')
    )
    op.create_index(op.f('ix_projects_key'), 'projects', ['key'])

    op.create_table(
        'issues',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('key', sa.String(length=20), nullable=False),
        sa.Column('project_id', sa.Integer(), nullable=False),
        sa.Column('summary', sa.String(length=500), nullable=False),
        sa.Column('issue_type', sa.String(length=50), nullable=False),
        sa.Column('priority', sa.String(length=20), nullable=False),
        sa.Column('status', sa.String(length=50), nullable=False),
        sa.Column('ap_area', sa.String(length=100), nullable=True),
        sa.Column('assigned_group', sa.String(length=100), nullable=True),
        sa.Column('created_at', sa.DateTime(), nullable=False),
        sa.Column('resolved_at', sa.DateTime(), nullable=True),
        sa.Column('sla_due_date', sa.DateTime(), nullable=True),
        sa.ForeignKeyConstraint(['project_id'], ['projects.id']),
        sa.PrimaryKeyConstraint('id'),
        sa.UniqueConstraint('key')
    )
    op.create_index(op.f('ix_issues_key'), 'issues', ['key'])
    op.create_index(op.f('ix_issues_project_id'), 'issues', ['project_id'])
    op.create_index(op.f('ix_issues_status'), 'issues', ['status'])
    op.create_index(op.f('ix_issues_priority'), 'issues', ['priority'])
    op.create_index('idx_project_status', 'issues', ['project_id', 'status'])
```

---

## Performance Considerations

1. **Indexes**: All frequently queried fields (project_id, status, priority) are indexed
2. **Composite Index**: `(project_id, status)` optimizes KPI queries
3. **Eager Loading**: Use `joinedload` to avoid N+1 queries when fetching projects with issues
4. **Query Optimization**: Use database aggregations (COUNT, AVG) instead of loading all records
5. **Pagination**: LIMIT/OFFSET for large result sets (>50 issues)

**Estimated Query Performance** (with indexes):
- Get projects: <5ms
- Get issues for project: <50ms (100 issues), <100ms (1000 issues)
- KPI calculations: <30ms each (using aggregations)
- Full dashboard data fetch: <200ms total

All targets meet constitutional <100ms query requirement.

---

## Data Validation

### Application-Level Validation

```python
# backend/src/models/issue.py
from sqlalchemy.orm import validates

class Issue(Base):
    # ...

    @validates('priority')
    def validate_priority(self, key, priority):
        valid_priorities = ['Critical', 'Blocker', 'High', 'Medium', 'Low']
        if priority not in valid_priorities:
            raise ValueError(f'Invalid priority: {priority}. Must be one of {valid_priorities}')
        return priority

    @validates('status')
    def validate_status(self, key, status):
        valid_statuses = ['Open', 'In Progress', 'Pending', 'Blocked', 'Resolved', 'Closed', 'Cancelled']
        if status not in valid_statuses:
            # Log warning but allow (future JIRA statuses may exist)
            print(f'WARNING: Unknown status {status}')
        return status

    @validates('resolved_at')
    def validate_resolved_at(self, key, resolved_at):
        if resolved_at and self.created_at and resolved_at < self.created_at:
            raise ValueError('resolved_at cannot be before created_at')
        return resolved_at
```

---

## Summary

This data model provides:
- ✅ Clear entity relationships (1:N Project→Issue)
- ✅ Comprehensive field definitions with validation rules
- ✅ JIRA API v3 compatibility for future integration
- ✅ Optimized indexes for <100ms query performance
- ✅ Type hints and property methods for calculated fields
- ✅ Mock data structure matching real JIRA responses
- ✅ Database migration strategy with Alembic

All design choices support constitutional Principles III (Performance), IV (Maintainability), and V (Contract Testing).
