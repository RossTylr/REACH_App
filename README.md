# REACH App  
Response and Emergency Access Coverage — Interactive Planning Tool

REACH App is a Streamlit-based planning application for NHS analysts and service planners. It enables transparent exploration of emergency response coverage, supports scenario-based comparison, and highlights access gaps across defined geographies.

This repository contains the **application and presentation layer** only. It is designed to sit on top of pre-prepared datasets and modelling outputs produced elsewhere in the REACH ecosystem.

---

## Problem Statement

Strategic planning for emergency care requires answering a deceptively simple question:

> Given a set of service locations and response-time standards, which populations are realistically reachable — and where are the gaps?

REACH App provides a repeatable, auditable way to explore this question without embedding operational assumptions or opaque modelling logic.

---

## Design Principles

- **Planning, not operations**  
  The application supports strategic and tactical review. It is not a dispatch system and makes no real-time predictions.

- **Explainability over complexity**  
  Coverage logic and metrics are deliberately simple and transparent to support scrutiny, assurance, and governance.

- **Separation of concerns**  
  This repository focuses on user interaction and visualisation. Core analytics, optimisation, and data engineering are handled upstream.

- **Configuration over code changes**  
  Scenarios, thresholds, and datasets are intended to be adjusted through configuration rather than modification of application logic.

---

## Core Capabilities

- **Interactive coverage maps**  
  Small-area (e.g. LSOA-level) visualisation of response coverage under configurable time thresholds.

- **Scenario comparison**  
  Side-by-side comparison of baseline and alternative configurations (for example, enabling or disabling sites).

- **Demand and equity overlays**  
  Optional overlays to surface areas of high demand that fall outside target response times.

- **Clear planning metrics**  
  Headline indicators designed for decision-making rather than operational monitoring.

---

## What REACH App Is — and Is Not

| Aspect | In Scope | Out of Scope |
|---|---|---|
| Strategic planning | Yes | — |
| Scenario testing | Yes | — |
| Equity and access review | Yes | — |
| Live dispatch | — | No |
| Real-time forecasting | — | No |
| Automated decision-making | — | No |

---

## Application Workflow

1. Load prepared geospatial, population, and travel-time inputs  
2. Select response-time thresholds and scenario parameters  
3. Enable or disable service sites  
4. Review coverage metrics and geographic gaps  
5. Export insights for planning and assurance discussions  

All outputs are deterministic for a given configuration.

---

## Data Assumptions

REACH App assumes:

- Stable geographic units (e.g. ONS LSOAs)  
- Pre-computed or externally supplied travel-time matrices  
- Static assumptions per scenario run  

The application does not infer missing data or attempt to correct upstream quality issues.

---

## Technical Stack

- **Frontend:** Streamlit  
- **Data processing:** Pandas, GeoPandas  
- **Geospatial:** Shapely, PyProj  
- **Mapping:** PyDeck or Folium  
- **Runtime:** Python 3.11+  
- **Dependency management:** `pyproject.toml` (PEP 621 / PEP 517)

---

## Dependency Management

This project uses **`pyproject.toml`** as the single source of truth for:

- Project metadata (name, version, description)  
- Runtime dependencies  
- Development dependencies  
- Build system configuration  

`pyproject.toml` is the modern Python standard and replaces legacy files such as `requirements.txt` and `setup.py`.

### Expected Structure

Dependencies should be declared using **one** of the following approaches.

#### PEP 621 (setuptools)

```toml
[project]
name = "reach-app"
version = "0.1.0"
description = "Interactive planning tool for emergency access coverage"

dependencies = [
  "streamlit",
  "pandas",
  "geopandas",
  "shapely",
  "pyproj"
]

[project.optional-dependencies]
dev = [
  "pytest",
  "black",
  "mypy"
]
