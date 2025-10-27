# Implementation Plan: AlAdhan Python Client Library

**Branch**: `001-python-client` | **Date**: 2025-10-27 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-python-client/spec.md`

## Summary

Build a modern, type-safe Python client library for the AlAdhan Prayer Times API with 100% endpoint coverage (30+ endpoints), built-in HTTP-aware caching, and separate synchronous/asynchronous clients. The library will provide full type safety through Pydantic v2 models, respect constitutional principles (behavior-driven testing, no mocks for in-process components, performance measurement), and deliver sub-millisecond cached responses while never exceeding API rate limits (12 req/s).

**Technical Approach**: Hand-write Pydantic models from validated OpenAPI schema, implement separate `AlAdhanClient` (sync) and `AsyncAlAdhanClient` (async) classes using httpx transport layer with hishel for RFC 9111-compliant HTTP caching, maintain <5s test suite using pytest with real API integration tests (no mocks except for network layer), and target Python 3.14+ for cutting-edge features.

## Technical Context

**Language/Version**: Python 3.14+
**Primary Dependencies**:

- httpx (0.27+) - Modern HTTP client with sync/async support
- hishel (0.0.29+) - RFC 9111-compliant HTTP caching for httpx
- pydantic (2.9+) - Data validation with Rust core (v2 required)
- pytest (8.3+) - Testing framework
- pytest-asyncio (0.24+) - Async test support
- pytest-benchmark (4.0+) - Performance benchmarking for PDD

**Storage**: HTTP cache backends (in-memory default, optional: filesystem, Redis, SQLite)
**Testing**: pytest with behavior-driven integration tests (no mocks for in-process code per constitution)
**Target Platform**: Cross-platform (Linux, macOS, Windows) - Python package for any OS
**Project Type**: Single library project (importable package, not application)
**Performance Goals**:

- Cached requests: <1ms (p95)
- Uncached requests: <500ms (p95)
- Test suite execution: <5s total
- Cache hit rate: >95% for typical usage

**Constraints**:

- Zero mock usage for in-process components (constitution Principle II)
- Separate sync/async clients (constitution Principle III - no Union types)
- 100% API endpoint coverage required (constitution Principle IV)
- 80%+ test coverage minimum (constitution Quality Standards)
- > 90% test resistance to refactoring (constitution Quality Standards)
- Never exceed 12 req/s API rate limit (spec FR-026)

**Scale/Scope**:

- 30+ API endpoints to implement
- ~15-20 Pydantic models (Prayer Times, Hijri/Gregorian Date, Method Info, etc.)
- Target: Developer library (not end-user application)
- Expected usage: IoT devices, web backends, mobile app backends

## Constitution Check

_GATE: Must pass before Phase 0 research. Re-check after Phase 1 design._

### Principle I: Type Safety ✅ PASS

- **Requirement**: All data crossing boundaries MUST be validated
- **Implementation**: Pydantic v2 models for all API responses with runtime validation
- **Verification**: mypy strict mode in CI, type checker must pass with zero errors

### Principle II: Behavior-Driven Testing ✅ PASS

- **Requirement**: Tests verify observable behavior, not implementation
- **Implementation**: Integration tests against real API (mocked network layer only)
- **Test Value**: Maximize `v = p × r × f × m` through behavior-focused sociable tests
- **Anti-Pattern Avoided**: No mocks for Pydantic models, cache layer, or client logic
- **Verification**: >90% tests stay green during refactoring, <5s suite execution

### Principle III: Clear Separation of Concerns ✅ PASS

- **Requirement**: Sync and async are separate components
- **Implementation**: `AlAdhanClient` (sync) and `AsyncAlAdhanClient` (async) as distinct classes
- **Type Safety**: No `Union[T, Awaitable[T]]` - each client has unambiguous return types
- **Verification**: Type checker verifies no dual-mode patterns

### Principle IV: Completeness ✅ PASS

- **Requirement**: 100% API endpoint coverage
- **Implementation**: All 30+ endpoints from validated OpenAPI schema
- **Verification**: Contract tests ensure every documented endpoint has corresponding client method

### Principle V: Performance as Design Constraint ✅ PASS

- **Requirement**: Performance measured, not assumed
- **Implementation**: pytest-benchmark for PDD, hishel for HTTP-aware caching
- **Measurement**: Performance tests in CI track <1ms cached, <500ms uncached targets
- **Verification**: Benchmark suite tracks regressions, cache hit rate monitored

### Foundational Constraints ✅ PASS

- **Python Language**: All code in Python 3.14+
- **Modern Features**: Uses PEP 649 (lazy annotations), PEP 779 (free-threading if beneficial)
- **Trunk-Based Development**: Small atomic commits via Graphite (gt)

### Quality Standards ✅ PASS

- **Test Suite Quality**: <5s execution, >90% refactoring resistance, zero in-process mocks
- **Code Quality**: Zero type errors (mypy strict), zero lint warnings (ruff), docstrings required
- **Performance**: <1ms cached, <500ms uncached, never exceed rate limits

**GATE STATUS**: ✅ **ALL CHECKS PASS** - Proceed to Phase 0

## Project Structure

### Documentation (this feature)

```text
specs/001-python-client/
├── plan.md              # This file
├── spec.md              # Feature specification (complete)
├── research.md          # Phase 0 research decisions
├── data-model.md        # Phase 1 Pydantic models design
├── quickstart.md        # Phase 1 developer quick-start guide
├── contracts/           # Phase 1 API endpoint contracts
└── checklists/
    └── requirements.md  # Spec validation checklist (complete)
```

### Source Code (repository root)

```text
# Single library project structure
aladhan/                 # Main package (importable as: from aladhan import ...)
├── __init__.py          # Package exports: AlAdhanClient, AsyncAlAdhanClient
├── client.py            # Synchronous client implementation
├── async_client.py      # Asynchronous client implementation
├── models/              # Pydantic v2 data models
│   ├── __init__.py
│   ├── prayer_times.py  # PrayerTimes, Timings models
│   ├── dates.py         # HijriDate, GregorianDate models
│   ├── methods.py       # CalculationMethod, MethodInfo models
│   ├── qibla.py         # QiblaDirection model
│   ├── names.py         # AllahName model (99 names)
│   └── common.py        # Shared models (Location, Cache metadata)
├── cache/               # HTTP caching layer
│   ├── __init__.py
│   ├── backends.py      # In-memory, filesystem, Redis backends
│   └── transport.py     # hishel integration with httpx
├── exceptions.py        # Custom exceptions (ValidationError, RateLimitError, etc.)
└── py.typed             # PEP 561 marker for type checking

tests/                   # Test suite (behavior-driven, sociable tests)
├── contract/            # API contract tests (verify all endpoints exist)
│   └── test_endpoints.py
├── integration/         # Integration tests (real API, mocked network only)
│   ├── test_prayer_times.py
│   ├── test_hijri_calendar.py
│   ├── test_qibla.py
│   └── test_caching.py
├── performance/         # Performance benchmarks (pytest-benchmark)
│   └── test_benchmarks.py
├── fixtures/            # Test fixtures (sample API responses)
│   └── api_responses/
└── conftest.py          # Pytest configuration

docs/                    # mkdocs documentation (Phase 3)
├── index.md
├── quickstart.md
├── api/
└── examples/

pyproject.toml           # Project metadata, dependencies, tool config
uv.lock                  # UV lock file (dependency resolution)
.trunk/                  # trunk.io configuration
README.md                # User-facing documentation (exists)
CLAUDE.md                # Architecture documentation (exists)
openapi.yaml             # Validated OpenAPI schema (exists)
.specify/                # Specify framework config (exists)
```

**Structure Decision**: Single library project - This is an importable Python package, not an application. No backend/frontend split needed. Simple flat structure with `aladhan/` as main package, `tests/` for behavior-driven tests, `docs/` for mkdocs. Follows standard Python packaging conventions (PEP 517/518 via pyproject.toml with UV as build backend).

## Complexity Tracking

> **No constitution violations** - Empty table (all gates passed)

## Phase 0: Research & Technical Decisions

### Research Decisions

Based on extensive prior research (Phase 0 competitive analysis, OpenAPI validation, HTTP header analysis), the following technical decisions are finalized:

#### 1. HTTP Client: httpx

**Decision**: Use httpx (not requests)

**Rationale**:

- Supports both sync and async from single codebase
- Pluggable transport architecture enables hishel integration
- HTTP/2 support, connection pooling
- Modern, actively maintained (requests is in maintenance mode)
- Constitution forbids requests (sync-only, blocking)

**Alternatives Considered**:

- `requests` - Rejected: sync-only, no async support, no transport plugins
- `aiohttp` - Rejected: async-only, would need separate sync library
- Raw `urllib` - Rejected: too low-level, missing features

#### 2. HTTP Caching: hishel

**Decision**: Use hishel for RFC 9111-compliant HTTP caching

**Rationale**:

- RFC 9111 compliant (respects Cache-Control, ETag, Expires)
- Built specifically for httpx via transport layer
- Supports multiple backends (in-memory, filesystem, Redis, SQLite)
- Handles conditional requests (If-None-Match with ETags)
- Handles 304 Not Modified responses correctly
- API provides explicit Cache-Control: max-age=7200, ETag headers

**Alternatives Considered**:

- `requests-cache` - Rejected: requires requests library (forbidden)
- Custom cache - Rejected: reinventing RFC 9111 is complex and error-prone
- No caching - Rejected: violates constitution Principle V and spec FR-023

#### 3. Validation: Pydantic v2

**Decision**: Use Pydantic v2 (Rust core)

**Rationale**:

- Runtime validation of all API responses (constitution Principle I)
- Type-safe models with IDE autocomplete (spec SC-009, SC-011)
- Excellent error messages for validation failures
- Performance: Rust core is 5-17x faster than Pydantic v1
- Industry standard for Python data validation
- Supports Python 3.14 features (lazy annotations via PEP 649)

**Alternatives Considered**:

- Plain dataclasses - Rejected: no runtime validation
- attrs - Rejected: less ergonomic than Pydantic for API modeling
- marshmallow - Rejected: older, slower, less type-safe
- Manual validation - Rejected: error-prone, no IDE support

#### 4. Testing: pytest with pytest-asyncio and pytest-benchmark

**Decision**: pytest ecosystem for all testing

**Rationale**:

- Industry standard Python testing framework
- pytest-asyncio enables async client testing
- pytest-benchmark enables PDD (Performance-Driven Development)
- Rich plugin ecosystem (coverage, fixtures, parametrization)
- Constitution requires <5s test suite (pytest is fast)

**Alternatives Considered**:

- unittest - Rejected: more verbose, less powerful fixtures
- nose2 - Rejected: less maintained, smaller ecosystem

#### 5. Package Management: UV

**Decision**: Use UV (not pip, poetry, or pipenv)

**Rationale**:

- 10-100x faster than pip (Rust-powered)
- PEP 517/518 compliant
- Workspace support for monorepos (future-proof)
- Lock files for reproducible builds (uv.lock)
- PEP 723 script support (inline metadata)
- Constitution emphasizes cutting-edge tooling

**Alternatives Considered**:

- pip - Rejected: slow, no lock files by default
- poetry - Rejected: slower than UV, more complex
- pipenv - Rejected: slow, less maintained

#### 6. Linting: trunk.io

**Decision**: trunk.io orchestrating ruff, mypy, black

**Rationale**:

- Orchestrates multiple tools (ruff for linting, mypy for types, black for formatting)
- Still uses individual tool configs (ruff in pyproject.toml, mypy.ini)
- Fast, parallel execution
- Constitution requires zero type errors, zero lint warnings

**Alternatives Considered**:

- Individual tools - Rejected: manual orchestration is tedious
- pre-commit - Rejected: slower than trunk, less integrated

#### 7. Documentation: mkdocs with material theme

**Decision**: mkdocs-material for documentation site

**Rationale**:

- Beautiful, responsive theme
- Markdown-based (easy to maintain)
- Excellent Python API documentation support
- Search, navigation, dark mode built-in

**Alternatives Considered**:

- Sphinx - Rejected: RST is less ergonomic than Markdown
- Read the Docs - Rejected: not a docs generator (hosting platform)

### No Further Research Needed

All technical stack decisions are complete based on:

1. Constitution principles and quality standards
2. Specification requirements (35 functional requirements)
3. Competitive analysis (aladhan.py, prayer-times-calculator)
4. OpenAPI schema validation (9 endpoints tested)
5. HTTP header analysis (Cache-Control, ETag, rate limits observed)

## Phase 1: Design & Contracts

### Data Model Overview

(See `data-model.md` for complete Pydantic model definitions)

**Core Models**:

1. **PrayerTimes** - Daily prayer times (Fajr, Dhuhr, Asr, Maghrib, Isha + additional)
2. **Timings** - Individual prayer time strings with timezone info
3. **HijriDate** - Islamic calendar date (day, month, year, weekday, holidays)
4. **GregorianDate** - Standard calendar date
5. **DateInfo** - Combined Hijri + Gregorian date info
6. **CalculationMethod** - Method parameters (fajr angle, isha angle, etc.)
7. **MethodInfo** - Full method details (ID, name, location, params)
8. **QiblaDirection** - Bearing angle from North to Kaaba
9. **AllahName** - One of 99 Names (Arabic, transliteration, English meaning)
10. **Location** - Latitude/longitude coordinates
11. **CacheMetadata** - ETag, timestamp, expiration info

**Model Hierarchy**:

```text
TimingsResponse
├── data
│   ├── timings: Timings
│   ├── date: DateInfo
│   │   ├── hijri: HijriDate
│   │   └── gregorian: GregorianDate
│   └── meta: MetaInfo
│       ├── method: CalculationMethod
│       └── location: Location

CalendarResponse
└── data: List[TimingsResponse data]

MethodsResponse
└── data: Dict[str, MethodInfo]

QiblaResponse
└── data: QiblaDirection
```

### API Contracts

(See `contracts/openapi.yaml` - already exists and validated)

**Endpoint Categories**:

1. **Prayer Times** (6 endpoints): By coordinates, city, address; with date variants
2. **Calendar** (4 endpoints): Monthly, yearly, Hijri-based
3. **Hijri Calendar** (6 endpoints): Conversions, holidays, current date
4. **Qibla** (2 endpoints): Direction calculation
5. **Asma Al-Husna** (2 endpoints): All names, specific name
6. **Methods** (4 endpoints): Method info, list all methods
7. **Utilities** (6+ endpoints): Timestamps, date helpers

**Client Method Mapping** (examples):

```python
# Sync Client
client.get_timings(date, lat, lon, method) → TimingsResponse
client.get_timings_by_city(date, city, country, method) → TimingsResponse
client.get_calendar(year, month, lat, lon, method) → CalendarResponse
client.get_qibla(lat, lon) → QiblaResponse
client.convert_gregorian_to_hijri(date) → DateInfo
client.get_asma_al_husna() → List[AllahName]

# Async Client (identical API surface)
await async_client.get_timings(...)
await async_client.get_timings_by_city(...)
# ... same methods, all async
```

### Quickstart Example

(See `quickstart.md` for full developer guide)

```python
# Install
# $ uv add aladhan-client

# Synchronous usage
from aladhan import AlAdhanClient
from datetime import date

client = AlAdhanClient()
times = client.get_timings(
    date=date.today(),
    latitude=25.2854,
    longitude=51.5310,
    method="ISNA"
)

print(f"Fajr: {times.data.timings.Fajr}")
print(f"Dhuhr: {times.data.timings.Dhuhr}")

# Asynchronous usage
from aladhan import AsyncAlAdhanClient
import asyncio

async def main():
    async with AsyncAlAdhanClient() as client:
        times = await client.get_timings(
            date=date.today(),
            latitude=25.2854,
            longitude=51.5310,
            method="ISNA"
        )
        print(f"Fajr: {times.data.timings.Fajr}")

asyncio.run(main())
```

## Next Steps

This plan is complete through Phase 1. Ready for:

1. **`/speckit.tasks`** - Generate task breakdown for implementation
2. **Implementation** - Begin coding with tasks.md as guide
3. **Continuous validation** - Verify constitution compliance throughout

**Files Generated**:

- ✅ `plan.md` (this file)
- ⏳ `research.md` (research decisions documented above, formal file next)
- ⏳ `data-model.md` (Pydantic model specifications, formal file next)
- ⏳ `contracts/` (reference openapi.yaml, formal contracts next)
- ⏳ `quickstart.md` (developer guide, formal file next)
