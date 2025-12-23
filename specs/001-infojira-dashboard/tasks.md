# Tasks: InfoJira Dashboard

**Feature Branch**: `001-infojira-dashboard`
**Input**: Design documents from `specs/001-infojira-dashboard/`
**Prerequisites**: plan.md, spec.md, data-model.md, contracts/

**Tests**: This feature follows **Test-First Development (TDD)** as mandated by Constitution Principle I (NON-NEGOTIABLE). Tests are written BEFORE implementation for all user stories.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Backend**: `backend/src/` for source code, `backend/tests/` for tests
- **Frontend**: `frontend/` for Tailwind tooling, templates in `backend/src/templates/`
- **Static files**: `backend/src/static/` for CSS, JavaScript, mock data

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create project directory structure per plan.md (backend/, frontend/, backend/src/, backend/tests/)
- [ ] T002 Initialize Python virtual environment and create requirements.txt with Flask 3.0+, SQLAlchemy 2.0+, python-dotenv
- [ ] T003 Create requirements-dev.txt with pytest 7.4+, pytest-flask, pytest-cov, black, flake8, mypy
- [ ] T004 [P] Initialize Node.js project in frontend/ with package.json for Tailwind CSS 3.4+
- [ ] T005 [P] Create .gitignore with venv/, node_modules/, *.pyc, __pycache__/, backend/dev_infojira.db
- [ ] T006 [P] Create .env.example with FLASK_APP, FLASK_ENV, DATABASE_URL, USE_MOCK_DATA, MOCK_DATA_PATH
- [ ] T007 Configure Tailwind CSS in frontend/tailwind.config.js with responsive breakpoints (sm, md, lg, xl)
- [ ] T008 [P] Configure PostCSS in frontend/postcss.config.js for Tailwind processing
- [ ] T009 [P] Add npm scripts in frontend/package.json for build:css and watch:css
- [ ] T010 Create run.py application entry point that imports create_app from backend/src/app.py

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T011 Create Flask application factory in backend/src/app.py with config loading
- [ ] T012 Create configuration module in backend/src/config.py with DevelopmentConfig, TestConfig, ProductionConfig classes
- [ ] T013 Create database module in backend/src/models/database.py with SQLAlchemy setup and Base class
- [ ] T014 [P] Create Project model in backend/src/models/project.py with id, key, name fields and validation
- [ ] T015 [P] Create Issue model in backend/src/models/issue.py with all fields per data-model.md and property methods (is_open, is_critical, is_overdue)
- [ ] T016 Create DataProvider abstract base class in backend/src/services/data_provider.py with get_projects() and get_issues() methods
- [ ] T017 Create MockDataService implementation in backend/src/services/mock_data_service.py that loads from JSON file
- [ ] T018 Copy mock data from specs/001-infojira-dashboard/contracts/jira-mock-data.json to backend/src/static/data/mock_jira.json
- [ ] T019 [P] Create base HTML template in backend/src/templates/base.html with Tailwind CSS link and Chart.js CDN
- [ ] T020 [P] Configure pytest in backend/tests/conftest.py with app fixture and in-memory SQLite test database
- [ ] T021 [P] Build Tailwind CSS by running npm run build:css in frontend/ to generate backend/src/static/css/output.css

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - View Project Health Overview (Priority: P1) 🎯 MVP

**Goal**: Display 4 KPI cards (total open issues, critical issues, avg resolution time, overdue SLA) when project selected

**Independent Test**: Select project from dropdown, verify KPI cards show correct values with appropriate units and visual highlighting for critical issues

### Tests for User Story 1 (TDD - Write FIRST, Ensure FAIL before implementation)

- [ ] T022 [P] [US1] Write unit test for KPICalculator.calculate_open_issues() in backend/tests/unit/test_kpi_calculator.py
- [ ] T023 [P] [US1] Write unit test for KPICalculator.calculate_critical_issues() in backend/tests/unit/test_kpi_calculator.py
- [ ] T024 [P] [US1] Write unit test for KPICalculator.calculate_avg_resolution_time() in backend/tests/unit/test_kpi_calculator.py
- [ ] T025 [P] [US1] Write unit test for KPICalculator.calculate_overdue_sla() in backend/tests/unit/test_kpi_calculator.py
- [ ] T026 [US1] Write integration test for GET /api/projects/{projectKey}/kpis endpoint in backend/tests/integration/test_api_endpoints.py
- [ ] T027 [US1] Run tests - verify all KPI tests FAIL (no implementation yet)

### Implementation for User Story 1

- [ ] T028 [US1] Create KPICalculator service class in backend/src/services/kpi_calculator.py with all four calculation methods
- [ ] T029 [US1] Create API Blueprint in backend/src/routes/api.py and register with Flask app
- [ ] T030 [US1] Implement GET /api/projects/{projectKey}/kpis endpoint in backend/src/routes/api.py
- [ ] T031 [US1] Run KPI unit tests - verify all PASS after implementation
- [ ] T032 [US1] Create KPI card component template in backend/src/templates/components/kpi_card.html with Tailwind styling
- [ ] T033 [US1] Create dashboard route Blueprint in backend/src/routes/dashboard.py with GET / endpoint
- [ ] T034 [US1] Create dashboard.html template in backend/src/templates/ with project selector and 4 KPI card placeholders
- [ ] T035 [US1] Create dashboard.js in backend/src/static/js/ to handle project selection and KPI data loading via AJAX
- [ ] T036 [US1] Add visual highlighting for critical issues KPI card (red background, warning icon) using Tailwind classes
- [ ] T037 [US1] Run integration tests - verify KPI endpoint returns correct values
- [ ] T038 [US1] Manual test: Select SUPP project, verify 4 KPI cards display with correct values and units

**Checkpoint**: At this point, User Story 1 should be fully functional and testable independently

---

## Phase 4: User Story 2 - Analyze Issue Distribution (Priority: P2)

**Goal**: Display bar chart for issue status distribution and pie chart for priority distribution

**Independent Test**: Select project, verify bar chart shows counts by status (In Progress, Resolved, Cancelled, Closed) and pie chart shows priority percentages with tooltips

### Tests for User Story 2 (TDD - Write FIRST)

- [ ] T039 [P] [US2] Write integration test for GET /api/projects/{projectKey}/charts/status endpoint in backend/tests/integration/test_api_endpoints.py
- [ ] T040 [P] [US2] Write integration test for GET /api/projects/{projectKey}/charts/priority endpoint in backend/tests/integration/test_api_endpoints.py
- [ ] T041 [US2] Run tests - verify chart endpoint tests FAIL

### Implementation for User Story 2

- [ ] T042 [P] [US2] Implement GET /api/projects/{projectKey}/charts/status endpoint in backend/src/routes/api.py
- [ ] T043 [P] [US2] Implement GET /api/projects/{projectKey}/charts/priority endpoint in backend/src/routes/api.py
- [ ] T044 [US2] Run chart endpoint tests - verify all PASS
- [ ] T045 [US2] Create charts component template in backend/src/templates/components/charts.html with canvas elements for Chart.js
- [ ] T046 [US2] Create charts.js in backend/src/static/js/ to initialize Chart.js bar chart for status distribution
- [ ] T047 [US2] Add Chart.js pie chart configuration in charts.js for priority distribution
- [ ] T048 [US2] Configure Chart.js tooltips to display exact counts and percentages on hover
- [ ] T049 [US2] Update dashboard.html to include charts component below KPI cards
- [ ] T050 [US2] Update dashboard.js to load chart data when project selected and call charts.js render functions
- [ ] T051 [US2] Add responsive sizing to charts using Chart.js responsive:true and Tailwind container classes
- [ ] T052 [US2] Manual test: Select SUPP project, verify both charts render with correct data and tooltips work

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently

---

## Phase 5: User Story 3 - Search and Filter Issue Details (Priority: P3)

**Goal**: Enable text search and priority filter on issues table with real-time client-side filtering

**Independent Test**: Enter search text and select priority filter, verify table shows only matching issues with "No issues found" message when no matches

### Tests for User Story 3 (TDD - Write FIRST)

- [ ] T053 [US3] Write unit test for FilterService.filter_by_text() in backend/tests/unit/test_filter_service.py
- [ ] T054 [US3] Write unit test for FilterService.filter_by_priority() in backend/tests/unit/test_filter_service.py
- [ ] T055 [US3] Run tests - verify filter tests FAIL

### Implementation for User Story 3

- [ ] T056 [US3] Create FilterService class in backend/src/services/filter_service.py with text and priority filter methods
- [ ] T057 [US3] Run filter unit tests - verify all PASS
- [ ] T058 [US3] Create filters component template in backend/src/templates/components/filters.html with search input and priority dropdown
- [ ] T059 [US3] Create table.js in backend/src/static/js/ with client-side filterTable() function for search and priority
- [ ] T060 [US3] Update dashboard.html to include filters component above issues table
- [ ] T061 [US3] Add event listeners in table.js for search input (keyup) and priority selector (change)
- [ ] T062 [US3] Implement "No issues found matching your criteria" empty state message in table component
- [ ] T063 [US3] Add filter reset logic when project changes in dashboard.js
- [ ] T064 [US3] Manual test: Search for "API", verify only matching issues shown; select "Critical" priority, verify filtering works

**Checkpoint**: At this point, User Stories 1, 2, AND 3 should all work independently

---

## Phase 6: User Story 4 - View Detailed Issue Information (Priority: P4)

**Goal**: Display sortable issues table with all columns (Key, Issue Type, Application, Summary, Assigned Group, Priority, Status) and pagination for >50 issues

**Independent Test**: Select project, verify table displays all issues with correct columns, sorting works on header click, pagination appears for large datasets

### Tests for User Story 4 (TDD - Write FIRST)

- [ ] T065 [US4] Write integration test for GET /api/projects/{projectKey}/issues endpoint with pagination in backend/tests/integration/test_api_endpoints.py
- [ ] T066 [US4] Run test - verify issues endpoint test FAILS

### Implementation for User Story 4

- [ ] T067 [US4] Implement GET /api/projects/{projectKey}/issues endpoint in backend/src/routes/api.py with limit/offset pagination
- [ ] T068 [US4] Run issues endpoint test - verify PASSES
- [ ] T069 [US4] Create issues_table component template in backend/src/templates/components/issues_table.html with table headers and body
- [ ] T070 [US4] Add sortTable() function in table.js to handle column sorting (ascending/descending toggle)
- [ ] T071 [US4] Add event listeners to table headers in table.js for sort on click
- [ ] T072 [US4] Implement pagination logic in table.js to show/hide rows and update pagination controls
- [ ] T073 [US4] Add pagination UI in issues_table.html with page numbers and next/previous buttons
- [ ] T074 [US4] Update dashboard.html to include issues_table component
- [ ] T075 [US4] Update dashboard.js to load issues data when project selected and populate table
- [ ] T076 [US4] Add responsive table styling with horizontal scroll on mobile using Tailwind overflow-x-auto
- [ ] T077 [US4] Manual test: Select SUPP project, verify table shows all columns, sorting works, pagination appears if >50 issues

**Checkpoint**: All core user stories (1-4) should now be independently functional

---

## Phase 7: User Story 5 - Apply Combined Filters (Priority: P5)

**Goal**: Enable combined filtering by project (required), issue type (optional), and AP Area (optional) with all dashboard elements updating

**Independent Test**: Select project + issue type + AP Area, verify KPIs/charts/table all update to show only matching subset; verify filters reset when project changes

### Tests for User Story 5 (TDD - Write FIRST)

- [ ] T078 [US5] Write integration test for combined filters (project + issueType + apArea query params) in backend/tests/integration/test_api_endpoints.py
- [ ] T079 [US5] Run test - verify combined filter test FAILS

### Implementation for User Story 5

- [ ] T080 [US5] Update GET /api/projects/{projectKey}/issues to accept issueType and apArea query parameters
- [ ] T081 [US5] Update GET /api/projects/{projectKey}/kpis to accept issueType and apArea query parameters
- [ ] T082 [US5] Update GET /api/projects/{projectKey}/charts/* endpoints to accept issueType and apArea query parameters
- [ ] T083 [US5] Run combined filter test - verify PASSES
- [ ] T084 [US5] Update filters.html component to include issue type dropdown and AP Area dropdown
- [ ] T085 [US5] Create filters.js in backend/src/static/js/ to handle combined filter state and trigger dashboard updates
- [ ] T086 [US5] Update dashboard.js to pass filter parameters (issueType, apArea) to all API calls (KPIs, charts, issues)
- [ ] T087 [US5] Implement filter reset logic: when project changes, reset issueType and apArea to "All"
- [ ] T088 [US5] Add empty state handling: when no issues match combined filters, show zero KPIs, empty charts with message, empty table
- [ ] T089 [US5] Manual test: Select SUPP + Bug + Mobile App, verify all dashboard elements filter correctly; change project, verify filters reset

**Checkpoint**: All user stories (1-5) should now be independently functional with full filter capabilities

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories and final touches

- [ ] T090 [P] Create custom.css in backend/src/static/css/ with Tailwind extensions for project-specific styles
- [ ] T091 [P] Add loading spinner component in templates/components/ displayed during AJAX requests
- [ ] T092 [P] Implement loading state in dashboard.js to show spinner while fetching data
- [ ] T093 [P] Create error handling in dashboard.js for API failures with user-friendly error messages
- [ ] T094 [P] Add ARIA labels to all interactive elements (dropdowns, buttons) for WCAG AA compliance
- [ ] T095 [P] Test color contrast of all KPI cards and charts using browser accessibility tools
- [ ] T096 [P] Add keyboard navigation support for table sorting and pagination
- [ ] T097 Create README.md in project root with setup instructions referencing quickstart.md
- [ ] T098 Add Python docstrings (Google style) to all public functions in services/ and routes/
- [ ] T099 Add type hints to all function signatures in backend/src/
- [ ] T100 Run black formatter on backend/src/ and backend/tests/
- [ ] T101 Run flake8 linter and fix any PEP 8 violations
- [ ] T102 Run mypy type checker and resolve any type errors
- [ ] T103 Run full test suite with coverage: pytest --cov=backend/src --cov-report=html
- [ ] T104 Verify test coverage ≥80% overall, 100% for kpi_calculator.py
- [ ] T105 Performance test: Verify dashboard loads in <2s with 100 issues
- [ ] T106 Performance test: Verify filter operations complete in <500ms
- [ ] T107 Create .flaskenv file with FLASK_APP=run.py for flask run command
- [ ] T108 Test on mobile viewport (375px width) and verify responsive design
- [ ] T109 Test on tablet viewport (768px width) and verify layout adapts
- [ ] T110 Test in Firefox, Chrome, Safari, Edge to ensure cross-browser compatibility
- [ ] T111 Final manual walkthrough of all 5 user stories from spec.md acceptance scenarios

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Story 1 (Phase 3)**: Depends on Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (Phase 4)**: Depends on Foundational (Phase 2) - No dependencies on other stories (independent)
- **User Story 3 (Phase 5)**: Depends on Foundational (Phase 2) AND User Story 4 (table must exist to filter)
- **User Story 4 (Phase 6)**: Depends on Foundational (Phase 2) - No dependencies on other stories (independent)
- **User Story 5 (Phase 7)**: Depends on User Stories 1, 2, 4 (combines KPIs, charts, table filtering)
- **Polish (Phase 8)**: Depends on all desired user stories being complete

### User Story Dependencies

```text
Setup (Phase 1)
    ↓
Foundational (Phase 2) ← BLOCKING GATE
    ↓
    ├─→ US1 (Phase 3) ────────┐
    ├─→ US2 (Phase 4) ────────┤
    ├─→ US4 (Phase 6) ───┐    ├─→ US5 (Phase 7)
    │                    │    │
    └─→ US3 (Phase 5) ←──┘────┘
            ↓
        Polish (Phase 8)
```

**Critical Path**: Setup → Foundational → US4 → US3 → US5 → Polish
**Parallel Opportunities**: US1, US2, US4 can all start simultaneously after Foundational

### Within Each User Story

1. **Tests MUST be written FIRST** (Constitutional Principle I - NON-NEGOTIABLE)
2. Tests must FAIL before implementation
3. Implementation proceeds to make tests pass
4. Tests are run after implementation to verify PASS
5. Models before services
6. Services before routes/endpoints
7. Backend endpoints before frontend integration
8. Story complete before moving to next priority

### Parallel Opportunities

**Setup Phase (Phase 1)**:
- T002, T004, T005, T006, T008, T009 can run in parallel (different files)

**Foundational Phase (Phase 2)**:
- T014, T015 (models) can run in parallel
- T019, T020, T021 (templates, tests, CSS) can run in parallel after T014-T015 complete

**User Story 1 Tests (Phase 3)**:
- T022, T023, T024, T025 (all unit tests) can run in parallel

**User Story 2 Tests (Phase 4)**:
- T039, T040 (integration tests) can run in parallel
- T042, T043 (endpoint implementations) can run in parallel

**User Story 3 Tests (Phase 5)**:
- T053, T054 (unit tests) can run in parallel

**User Story 4 (Phase 6)**:
- Limited parallelism (mostly sequential due to dependencies)

**User Story 5 (Phase 7)**:
- T080, T081, T082 (endpoint updates) can run in parallel

**Polish Phase (Phase 8)**:
- T090, T091, T092, T093, T094, T095, T096 can all run in parallel (different concerns)
- T100, T101, T102 (code quality tools) can run in parallel

**Parallel by User Story** (after Foundational):
```bash
# Launch all independent user stories together:
Task Group A: User Story 1 (T022-T038)
Task Group B: User Story 2 (T039-T052)
Task Group C: User Story 4 (T065-T077)
# Then sequentially: US3 → US5 → Polish
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (T001-T010)
2. Complete Phase 2: Foundational (T011-T021) ← CRITICAL GATE
3. Complete Phase 3: User Story 1 (T022-T038)
4. **STOP and VALIDATE**: Test User Story 1 independently per acceptance scenarios
5. Deploy/demo if ready (working dashboard with KPIs)

**Deliverable**: Minimal viable product with project selection and KPI display

### Incremental Delivery

1. **Foundation** (Phases 1-2): Setup + Foundational → Infrastructure ready
2. **MVP** (Phase 3): Add User Story 1 → Test independently → Deploy/Demo
3. **Charts** (Phase 4): Add User Story 2 → Test independently → Deploy/Demo
4. **Table** (Phase 6): Add User Story 4 → Test independently → Deploy/Demo
5. **Table Filters** (Phase 5): Add User Story 3 → Test independently → Deploy/Demo
6. **Combined Filters** (Phase 7): Add User Story 5 → Test independently → Deploy/Demo
7. **Polish** (Phase 8): Final touches → Full release

Each increment adds value without breaking previous functionality.

### Parallel Team Strategy

With multiple developers:

1. **Everyone**: Complete Setup + Foundational together (Phases 1-2)
2. Once Foundational is done:
   - **Developer A**: User Story 1 (T022-T038)
   - **Developer B**: User Story 2 (T039-T052)
   - **Developer C**: User Story 4 (T065-T077)
3. After US4 complete:
   - **Developer D**: User Story 3 (T053-T064) - depends on US4
4. After US1, US2, US4 complete:
   - **Developer E**: User Story 5 (T078-T089) - depends on US1, US2, US4
5. **Everyone**: Polish (Phase 8) together

Stories complete and integrate independently.

---

## Notes

- **[P] tasks** = different files, no dependencies - safe to run in parallel
- **[Story] label** maps task to specific user story for traceability
- Each user story should be independently completable and testable
- **TDD MANDATORY**: Verify tests fail before implementing (Constitutional Principle I)
- Commit after each task or logical group for clean git history
- Stop at any checkpoint to validate story independently
- **Performance targets**: Dashboard load <2s, filter ops <500ms, DB queries <100ms
- **Coverage target**: 80% overall, 100% for kpi_calculator.py
- Run quality gates before each commit: black, flake8, mypy, pytest

---

## Task Count Summary

- **Total Tasks**: 111
- **Phase 1 (Setup)**: 10 tasks
- **Phase 2 (Foundational)**: 11 tasks (BLOCKING)
- **Phase 3 (User Story 1)**: 17 tasks (7 tests + 10 implementation)
- **Phase 4 (User Story 2)**: 14 tasks (3 tests + 11 implementation)
- **Phase 5 (User Story 3)**: 12 tasks (3 tests + 9 implementation)
- **Phase 6 (User Story 4)**: 13 tasks (2 tests + 11 implementation)
- **Phase 7 (User Story 5)**: 12 tasks (2 tests + 10 implementation)
- **Phase 8 (Polish)**: 22 tasks

**Parallel Opportunities**: 35 tasks marked [P] can run in parallel within their phases
**Test Tasks**: 17 test tasks (TDD compliance with Constitution Principle I)

**MVP Scope** (Phases 1-3): 38 tasks
**Full Feature** (Phases 1-8): 111 tasks
