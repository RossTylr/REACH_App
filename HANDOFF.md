# REACH App - Claude Chat Handoff Document

> **REACH** = Response and Emergency Access Coverage

This document provides all essential context for Claude Chat (or any developer) to understand and continue development of the REACH App.

---

## Quick Start

```bash
# Clone and setup
cd /home/user/REACH_App
poetry install
poetry shell

# Run the app
streamlit run src/streamlit_app/app.py

# Run tests
pytest

# Code quality
black src/ tests/
ruff check src/ tests/
mypy src/
```

---

## What Is REACH App?

REACH App is a **Streamlit-based interactive planning tool** designed for **NHS analysts and service planners** to analyze emergency response coverage across defined geographies.

### Core Problem Solved

> "Given a set of service locations (e.g., ambulance stations) and response-time standards, which populations are realistically reachable — and where are the gaps?"

### Key Use Cases

1. **Coverage Analysis** - Visualize which areas are covered within target response times
2. **Scenario Comparison** - Compare baseline vs. alternative service configurations
3. **Equity Assessment** - Identify populations outside acceptable response times
4. **Planning Support** - Provide transparent, auditable metrics for decision-making

### What It Is NOT

- ❌ Not a real-time dispatch system
- ❌ Not a forecasting/prediction tool
- ❌ Not an automated decision-maker
- ✅ A **planning and analysis** tool for strategic/tactical review

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     STREAMLIT UI LAYER                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │ Coverage Map│  │  Optimizer  │  │      Analytics          │  │
│  │   (Page 1)  │  │   (Page 3)  │  │       (Page 4)          │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  Components: map_viewer | kpi_cards | filters | site_selector│
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       CORE LOGIC LAYER                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │   coverage   │  │  optimization│  │      metrics         │   │
│  │  (coverage.py)│  │(optimization.py)│(metrics.py)          │   │
│  └──────────────┘  └──────────────┘  └──────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        DATA LAYER                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │   loaders    │  │  validators  │  │     transforms       │   │
│  └──────────────┘  └──────────────┘  └──────────────────────┘   │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │  geo/spatial │  │  geo/coords  │  │   geo/isochrones     │   │
│  └──────────────┘  └──────────────┘  └──────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     INPUT DATA SOURCES                          │
│  • LSOA boundaries (GeoPackage)                                 │
│  • Service stations (CSV)                                       │
│  • Travel time matrix (Parquet)                                 │
│  • Population data                                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## Directory Structure

```
REACH_App/
├── src/
│   ├── reach/                    # Core business logic
│   │   ├── core/                 # Coverage, optimization, metrics
│   │   ├── data/                 # Loading, validation, transforms
│   │   ├── models/               # Pydantic data models
│   │   ├── geo/                  # Geospatial operations
│   │   ├── utils/                # Logging, caching, errors
│   │   └── config.py             # Central configuration
│   │
│   └── streamlit_app/            # UI/Presentation layer
│       ├── app.py                # Main entry point
│       ├── pages/                # Multi-page navigation
│       ├── components/           # Reusable UI components
│       └── state/                # Session management
│
├── tests/
│   ├── unit/                     # Unit tests
│   ├── integration/              # End-to-end tests
│   └── fixtures/                 # Sample data for tests
│
├── data/logo/                    # Branding assets
├── docs/                         # Additional documentation
├── notebooks/                    # Exploration notebooks
├── scripts/                      # Utility scripts
└── .github/workflows/            # CI/CD pipelines
```

---

## Tech Stack

| Category | Technology | Purpose |
|----------|------------|---------|
| **UI Framework** | Streamlit ^1.36 | Interactive web application |
| **Mapping** | PyDeck ^0.9 | WebGL geospatial visualization |
| **Data** | Pandas ^2.2, GeoPandas ^0.14 | Data manipulation |
| **Optimization** | PuLP ^2.8, NetworkX ^3.3 | Linear programming, graph analysis |
| **Validation** | Pydantic ^2.7 | Data model validation |
| **Config** | pydantic-settings, python-dotenv | Environment management |
| **Logging** | Loguru ^0.7 | Structured logging |
| **Testing** | pytest ^8.2, pytest-cov | Test framework |
| **Code Quality** | black, mypy, ruff, pre-commit | Formatting, types, linting |

---

## Data Model

### Expected Input Data

1. **Geographic Boundaries** (LSOA level)
   - Format: GeoPackage (`.gpkg`) or Shapefile
   - Contains: LSOA polygons with identifiers
   - Example: `tests/fixtures/synthetic_lsoas.gpkg`

2. **Service Stations**
   - Format: CSV
   - Columns: id, name, latitude, longitude, capacity, status
   - Example: `tests/fixtures/synthetic_stations.csv`

3. **Travel Time Matrix**
   - Format: Parquet
   - Structure: Origin (LSOA) → Destination (Station) → Travel time
   - Example: `tests/fixtures/travel_matrix_sample.parquet`

### Core Entities

```python
# src/reach/models/site.py
class ServiceSite:
    """A service location (e.g., ambulance station)"""
    id: str
    name: str
    coordinates: tuple[float, float]  # lat, lon
    capacity: int
    enabled: bool  # For scenario testing

# src/reach/models/coverage.py
class CoverageResult:
    """Coverage calculation output"""
    site_id: str
    lsoa_id: str
    travel_time_minutes: float
    is_covered: bool  # Based on threshold

# src/reach/models/optimization.py
class OptimizationResult:
    """Optimization output"""
    selected_sites: list[str]
    total_coverage: float
    coverage_gaps: list[str]
```

---

## Current Implementation State

### ✅ Completed / Scaffolded

- [x] Project structure and directory layout
- [x] Poetry-based dependency management
- [x] pyproject.toml with all configurations
- [x] Test framework with fixtures
- [x] Pre-commit hooks setup
- [x] README documentation
- [x] Logo/branding assets
- [x] Multi-page Streamlit structure

### 🔲 Needs Implementation

- [ ] Core coverage calculation engine (`src/reach/core/coverage.py`)
- [ ] Optimization algorithms (`src/reach/core/optimization.py`)
- [ ] Metrics computation (`src/reach/core/metrics.py`)
- [ ] Data loaders (`src/reach/data/loaders.py`)
- [ ] Data validators (`src/reach/data/validators.py`)
- [ ] Geospatial operations (`src/reach/geo/`)
- [ ] Streamlit page implementations
- [ ] UI components
- [ ] Docker configuration
- [ ] CI/CD workflows
- [ ] Additional documentation

---

## Key Development Patterns

### 1. Configuration-Driven Design

All thresholds, scenarios, and parameters should be configurable:

```python
# src/reach/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    response_time_threshold_minutes: int = 8
    default_scenario: str = "baseline"
    log_level: str = "INFO"

    class Config:
        env_file = ".env"
```

### 2. Pydantic Models for Validation

Use Pydantic for all data structures:

```python
from pydantic import BaseModel, Field

class Site(BaseModel):
    id: str
    name: str
    lat: float = Field(ge=-90, le=90)
    lon: float = Field(ge=-180, le=180)
```

### 3. Streamlit Session State

Manage state across pages:

```python
# src/streamlit_app/state/session.py
import streamlit as st

def init_session_state():
    if "selected_sites" not in st.session_state:
        st.session_state.selected_sites = []
    if "threshold" not in st.session_state:
        st.session_state.threshold = 8
```

### 4. Separation of Concerns

- **UI Layer** (`streamlit_app/`): Only handles display and user interaction
- **Core Layer** (`reach/core/`): Business logic, calculations
- **Data Layer** (`reach/data/`): Loading, validation, transformation

---

## Testing Approach

### Run Tests

```bash
# All tests with coverage
pytest

# Unit tests only
pytest tests/unit/

# Integration tests only
pytest tests/integration/

# Specific test
pytest tests/unit/test_coverage.py -v
```

### Test Structure

```python
# tests/unit/test_coverage.py
import pytest
from reach.core.coverage import calculate_coverage

def test_coverage_within_threshold():
    """Sites within threshold should be marked as covered"""
    result = calculate_coverage(
        site_id="station_1",
        travel_time=5,
        threshold=8
    )
    assert result.is_covered is True

def test_coverage_outside_threshold():
    """Sites outside threshold should not be covered"""
    result = calculate_coverage(
        site_id="station_1",
        travel_time=12,
        threshold=8
    )
    assert result.is_covered is False
```

### Fixtures

Test data is in `tests/fixtures/`:
- `synthetic_lsoas.gpkg` - Sample geographic boundaries
- `synthetic_stations.csv` - Sample service locations
- `travel_matrix_sample.parquet` - Sample travel times

---

## Common Development Tasks

### Add a New Streamlit Page

1. Create file in `src/streamlit_app/pages/` with numbered prefix (e.g., `5_new_page.py`)
2. Add page layout and components
3. Connect to core logic through imports from `reach.core`

```python
# src/streamlit_app/pages/5_comparison.py
import streamlit as st
from reach.core.metrics import compute_comparison

st.title("Scenario Comparison")

# Use session state
scenario_a = st.session_state.get("scenario_a", "baseline")
scenario_b = st.selectbox("Compare with:", ["alternative_1", "alternative_2"])

if st.button("Compare"):
    results = compute_comparison(scenario_a, scenario_b)
    st.dataframe(results)
```

### Add a New Core Algorithm

1. Create or update file in `src/reach/core/`
2. Add Pydantic models in `src/reach/models/` if needed
3. Write unit tests in `tests/unit/`
4. Add integration test if it connects multiple components

### Add a New Data Source

1. Define loader in `src/reach/data/loaders.py`
2. Add validator in `src/reach/data/validators.py`
3. Create Pydantic model for the data structure
4. Add sample data to `tests/fixtures/`

---

## Environment Setup

### Required Environment Variables

Create `.env` file from template:

```bash
cp .env.example .env
```

Expected variables:
```env
# Application Settings
LOG_LEVEL=INFO
RESPONSE_TIME_THRESHOLD=8

# Data Paths (optional - can use defaults)
DATA_DIR=/path/to/data
LSOA_FILE=lsoas.gpkg
STATIONS_FILE=stations.csv
TRAVEL_MATRIX_FILE=travel_times.parquet
```

### Running Locally

```bash
# Activate environment
poetry shell

# Run Streamlit
streamlit run src/streamlit_app/app.py

# App available at http://localhost:8501
```

---

## Important Contextual Notes

### NHS Domain Context

- **LSOA** = Lower Layer Super Output Area (UK census geography, ~1,500 people each)
- **Response time standards** = Target times for emergency services (e.g., 8 minutes for Category 1)
- **Isochrones** = Areas reachable within a given travel time

### Design Decisions

1. **Deterministic outputs** - Same inputs always produce same results (no randomness)
2. **Explainability** - Simple, transparent metrics over complex black-box algorithms
3. **Auditability** - All assumptions and calculations must be traceable
4. **No data inference** - App does not guess or fill in missing data

### Assumptions

- Geographic units are stable (ONS LSOA definitions)
- Travel times are pre-computed externally
- Static assumptions per scenario run
- Authentication handled at infrastructure level

---

## Git Workflow

### Current Branch

```
claude/create-handoff-docs-XHTuQ
```

### Commit Conventions

```bash
# Feature
git commit -m "feat: add coverage calculation engine"

# Bug fix
git commit -m "fix: correct travel time parsing"

# Documentation
git commit -m "docs: update handoff documentation"

# Refactor
git commit -m "refactor: extract geospatial utilities"
```

---

## Key Files Quick Reference

| Purpose | File |
|---------|------|
| Project config | `pyproject.toml` |
| Main app entry | `src/streamlit_app/app.py` |
| Core config | `src/reach/config.py` |
| Coverage logic | `src/reach/core/coverage.py` |
| Optimization | `src/reach/core/optimization.py` |
| Data loading | `src/reach/data/loaders.py` |
| Test config | `pytest.ini` |
| Sample data | `tests/fixtures/` |

---

## Next Steps for Development

### Priority 1: Core Engine
1. Implement `coverage.py` - coverage calculation logic
2. Implement `loaders.py` - data loading functions
3. Implement `validators.py` - data validation

### Priority 2: UI Implementation
1. Complete `1_coverage_map.py` - main coverage visualization
2. Build `map_viewer.py` component with PyDeck
3. Implement `kpi_cards.py` for metrics display

### Priority 3: Optimization
1. Implement optimization algorithms in `optimization.py`
2. Build optimizer UI page
3. Add scenario comparison features

### Priority 4: Polish
1. Complete Docker configuration
2. Set up CI/CD workflows
3. Fill in documentation files
4. Add pre-commit hook definitions

---

## Questions for Future Development

When continuing work on REACH App, consider:

1. **Data sources** - Where will production LSOA/travel time data come from?
2. **Deployment** - What infrastructure will host the app? (Streamlit Cloud, Azure, AWS?)
3. **Authentication** - What auth mechanism is required for NHS deployment?
4. **Integration** - Does this need to connect with other NHS systems?
5. **Scale** - How many concurrent users expected?

---

## Contact & Resources

- **Repository Owner**: Ross Taylor
- **License**: MIT (2025)
- **Repository**: `RossTylr/REACH_App`

---

*Document created: 2025-12-30*
*For use with Claude Chat or any developer continuing REACH App development*
