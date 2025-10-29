# Implementation Tasks: AlAdhan Python Client Library

**Feature**: AlAdhan Python Client Library
**Branch**: `001-python-client`
**Spec**: [spec.md](./spec.md) | **Plan**: [plan.md](./plan.md)
**Created**: 2025-10-28

## Overview

This document breaks down the implementation of the AlAdhan Python Client into atomic, executable tasks organized by user story priority. Each user story represents an independently testable increment that delivers value.

**Tech Stack**: Python 3.14+, httpx 0.27+, hishel 0.0.29+, Pydantic v2.9+, pytest 8.3+, UV package manager

**Validation Philosophy**: Minimal client-side validation (basic sanity checks only). Let API handle complex business rules. This approach:

- Catches 95% of user errors immediately (null checks, basic ranges, types)
- Preserves rate limit quota (invalid requests don't hit API)
- Minimizes maintenance burden (simple checks don't evolve with API)
- Relies on Pydantic for automatic type validation
- Defers complex rules to API (calculation methods, city existence, Hijri calendar rules)

**Implementation Strategy**:

- **MVP**: User Story 1 (P1) - Prayer times by coordinates
- **Incremental Delivery**: Each user story builds on foundations without breaking previous stories
- **Parallel Execution**: Tasks marked with `[P]` can be executed in parallel within each phase
- **Independent Testing**: Each user story phase is independently testable

## Task Format

```text
- [ ] [TaskID] [P?] [Story?] Description with file path
```

- **TaskID**: Sequential number (T001, T002, etc.)
- **[P]**: Parallelizable (can execute concurrently with other [P] tasks in same phase)
- **[Story]**: User story label (e.g., [US1], [US2]) for traceability

## Phase 1: Project Setup

**Goal**: Initialize project structure, dependencies, and tooling

### Setup Tasks

- [ ] T001 Initialize UV project with pyproject.toml in repository root
- [ ] T002 Configure pyproject.toml with project metadata (name: aladhan-client, version: 0.1.0, Python 3.14+)
- [ ] T003 Add core dependencies to pyproject.toml (httpx >=0.27, hishel >=0.0.29, pydantic >=2.9)
- [ ] T004 Add dev dependencies to pyproject.toml (pytest >=8.3, pytest-asyncio >=0.24, pytest-benchmark >=4.0, mypy, ruff)
- [ ] T005 Create aladhan/ package directory in repository root
- [ ] T006 Create aladhan/\_\_init\_\_.py with package exports (empty for now)
- [ ] T007 Create aladhan/py.typed marker file for PEP 561 type checking support
- [ ] T008 Create tests/ directory in repository root
- [ ] T009 Create tests/conftest.py with pytest configuration
- [ ] T010 Create .python-version file with "3.14" for UV
- [ ] T011 Configure mypy in pyproject.toml with strict professional settings (strict=true, disallow_untyped_defs=true, disallow_any_generics=true, warn_return_any=true, warn_unused_ignores=true, no_implicit_optional=true, implicit_reexport=false, strict_equality=true, enable_error_code=["ignore-without-code","redundant-expr","truthy-bool"], plugins=["pydantic.mypy"])
- [ ] T012 Configure ruff in pyproject.toml with professional rules (line-length=100, target-version="py314", select=["E","F","I","N","UP","S","B","A","C4","DTZ","T10","EM","ISC","ICN","PIE","PT","Q","RET","SIM","TID","ARG","PTH","PD","PL","TRY","NPY","RUF"], ignore=["S101","PLR2004","TRY003","EM101"], per-file-ignores for tests, isort config, mccabe max-complexity=10)
- [ ] T013 Configure pytest in pyproject.toml ([tool.pytest.ini_options] section)
- [ ] T014 Run `uv sync` to create virtual environment and install dependencies
- [ ] T015 Verify mypy runs with zero errors on empty package (mypy aladhan/)
- [ ] T016 Verify ruff runs with zero warnings (ruff check aladhan/)
- [ ] T017 Verify pytest discovers tests directory (pytest --collect-only)

**Completion Criteria**: Project structure matches plan.md, all tools run successfully, dependencies installed

---

## Phase 2: Foundational Components

**Goal**: Implement shared infrastructure needed by all user stories

### Exception Hierarchy

- [ ] T018 [P] Create aladhan/exceptions.py with base AlAdhanError exception class
- [ ] T019 [P] Add AlAdhanAPIError exception (API-level errors) in aladhan/exceptions.py
- [ ] T020 [P] Add ValidationError exception (input validation errors) in aladhan/exceptions.py
- [ ] T021 [P] Add NetworkError exception (connection issues) in aladhan/exceptions.py
- [ ] T022 [P] Add RateLimitError exception (12 req/s exceeded) in aladhan/exceptions.py
- [ ] T023 [P] Add CacheError exception (caching failures) in aladhan/exceptions.py

### Core Data Models (Shared)

- [ ] T024 [P] Create aladhan/models/ package directory
- [ ] T025 [P] Create aladhan/models/\_\_init\_\_.py (empty for now)
- [ ] T026 [P] Create aladhan/models/common.py with Location model (latitude, longitude fields with validation)
- [ ] T027 [P] Add GeoCoordinates validator to common.py (latitude ±90, longitude ±180)
- [ ] T028 [P] Create aladhan/models/dates.py with HijriDate model (day, month, year, weekday, holidays)
- [ ] T029 [P] Create GregorianDate model in aladhan/models/dates.py (day, month, year, weekday)
- [ ] T030 [P] Create DateInfo model in aladhan/models/dates.py (hijri: HijriDate, gregorian: GregorianDate)

### HTTP Client Foundation

- [ ] T031 Create aladhan/\_http.py with BaseHTTPClient class (shared logic for sync/async)
- [ ] T032 Add \_build_url() method to BaseHTTPClient in aladhan/\_http.py (constructs API URLs)
- [ ] T033 Add \_handle_response() method to BaseHTTPClient in aladhan/\_http.py (validates status codes, handles errors)
- [ ] T034 Add API_BASE_URL constant ("<https://api.aladhan.com/v1>") in aladhan/\_http.py
- [ ] T035 Add rate limit tracking logic (12 req/s) to BaseHTTPClient in aladhan/\_http.py

### Caching Foundation

- [ ] T036 [P] Create aladhan/cache/ package directory
- [ ] T037 [P] Create aladhan/cache/\_\_init\_\_.py with cache backend exports
- [ ] T038 [P] Create aladhan/cache/backends.py with InMemoryCacheBackend class (using hishel)
- [ ] T039 [P] Add FileSystemCacheBackend class in aladhan/cache/backends.py (optional backend)
- [ ] T040 [P] Create aladhan/cache/transport.py with CachedHTTPTransport wrapper for httpx
- [ ] T041 [P] Integrate hishel CacheTransport in aladhan/cache/transport.py (RFC 9111 compliance)

**Completion Criteria**: Exceptions defined, shared models created, HTTP client foundation ready, caching infrastructure in place

---

## Phase 3: User Story 1 (P1) - Get Prayer Times by Coordinates

**Goal**: Enable developers to retrieve prayer times using geographic coordinates

**Independent Test**: Provide coordinates (25.2854, 51.5310) and date, verify Fajr/Dhuhr/Asr/Maghrib/Isha times returned

### Models

- [ ] T042 [P] [US1] Create aladhan/models/prayer_times.py with Timings model (all 11 prayer times as fields)
- [ ] T043 [P] [US1] Add field validation to Timings model (time format: HH:MM or ISO8601)
- [ ] T044 [P] [US1] Create TimingsResponse model in aladhan/models/prayer_times.py (data: {timings, date, meta})
- [ ] T045 [P] [US1] Create aladhan/models/methods.py with CalculationMethod model (fajr_angle, isha_angle, etc.)
- [ ] T046 [P] [US1] Add MethodInfo model in aladhan/models/methods.py (id, name, params: CalculationMethod)
- [ ] T047 [P] [US1] Create MetaInfo model in aladhan/models/prayer_times.py (method: MethodInfo, location: Location)

### Synchronous Client - Prayer Times by Coordinates

- [ ] T048 [US1] Create aladhan/client.py with AlAdhanClient class
- [ ] T049 [US1] Add \_\_init\_\_() to AlAdhanClient with cache_backend parameter (default: InMemory)
- [ ] T050 [US1] Initialize httpx.Client with CachedHTTPTransport in AlAdhanClient.\_\_init\_\_()
- [ ] T051 [US1] Implement get_timings() method in AlAdhanClient (date, latitude, longitude, method) → TimingsResponse
- [ ] T052 [US1] Add 'school' parameter to get_timings() in AlAdhanClient (default: None, options: None|'Hanafi')
- [ ] T053 [US1] Pass school parameter to API endpoint in \_fetch_timings() (query: ?school=1 for Hanafi)
- [ ] T054 [P] [US1] Add 'school' parameter support to async get_timings() in AsyncAlAdhanClient
- [ ] T055 [P] [US1] Add test_get_timings_with_hanafi_school() in test_prayer_times.py (verify Asr time differs)
- [ ] T056 [US1] Add 'tune' parameter to get_timings() in AlAdhanClient (dict mapping prayer names to minute offsets)
- [ ] T057 [US1] Implement tune parameter serialization in \_fetch_timings() (query: ?tune=fajr,3,dhuhr,-2,...)
- [ ] T058 [P] [US1] Add 'tune' parameter support to async get_timings() in AsyncAlAdhanClient
- [ ] T059 [P] [US1] Add test_get_timings_with_tuning() in test_prayer_times.py (verify adjusted times)
- [ ] T060 [US1] Add basic input validation for get_timings() (non-null parameters, use GeoCoordinates validator)
- [ ] T061 [US1] Add error handling for get_timings() (network errors, API errors with clear messages)
- [ ] T062 [US1] Implement \_fetch_timings() private method in AlAdhanClient (HTTP GET /timings/{date})
- [ ] T063 [US1] Add response parsing with Pydantic validation in \_fetch_timings()

### Asynchronous Client - Prayer Times by Coordinates

- [ ] T064 [P] [US1] Create aladhan/async_client.py with AsyncAlAdhanClient class
- [ ] T065 [P] [US1] Add \_\_init\_\_() to AsyncAlAdhanClient with cache_backend parameter
- [ ] T066 [P] [US1] Initialize httpx.AsyncClient with CachedHTTPTransport in AsyncAlAdhanClient.\_\_init\_\_()
- [ ] T067 [P] [US1] Implement async get_timings() method in AsyncAlAdhanClient (same signature as sync)
- [ ] T068 [P] [US1] Add async context manager support (**aenter**, **aexit**) to AsyncAlAdhanClient
- [ ] T069 [P] [US1] Implement async \_fetch_timings() private method in AsyncAlAdhanClient
- [ ] T070 [P] [US1] Add identical input validation and error handling to async get_timings()

### Package Exports

- [ ] T071 [US1] Update aladhan/\_\_init\_\_.py to export AlAdhanClient and AsyncAlAdhanClient
- [ ] T072 [US1] Update aladhan/models/\_\_init\_\_.py to export TimingsResponse, Timings, DateInfo models

### Integration Tests (Behavior-Driven)

- [ ] T073 [P] [US1] Create tests/integration/ directory
- [ ] T074 [P] [US1] Create tests/integration/test_prayer_times.py
- [ ] T075 [P] [US1] Add test_get_timings_by_coordinates_sync() in test_prayer_times.py (real API call)
- [ ] T076 [P] [US1] Add test_get_timings_by_coordinates_async() in test_prayer_times.py (real API call)
- [ ] T077 [P] [US1] Add test_get_timings_with_invalid_coordinates() for validation errors
- [ ] T078 [P] [US1] Add test_get_timings_with_different_methods() (ISNA, MWL, MAKKAH)
- [ ] T079 [P] [US1] Add test_get_timings_cached_response() to verify <1ms performance

### Performance Benchmarks

- [ ] T080 [P] [US1] Create tests/performance/ directory
- [ ] T081 [P] [US1] Create tests/performance/test_benchmarks.py
- [ ] T082 [P] [US1] Add benchmark_cached_request() using pytest-benchmark (<1ms target)
- [ ] T083 [P] [US1] Add benchmark_uncached_request() using pytest-benchmark (<500ms target)

**Completion Criteria**: AlAdhanClient and AsyncAlAdhanClient implement get_timings(), all US1 acceptance scenarios pass, <5s test execution

---

## Phase 4: User Story 2 (P2) - Get Prayer Times by City/Address

**Goal**: Enable prayer time retrieval using city names or addresses (privacy-friendly)

**Independent Test**: Provide "Dubai, UAE" or street address, verify prayer times returned accurately

### Client Methods - City Lookup

- [ ] T084 [P] [US2] Implement get_timings_by_city() in AlAdhanClient (date, city, country, method) → TimingsResponse
- [ ] T085 [P] [US2] Add basic validation to get_timings_by_city() (non-null, non-empty strings)
- [ ] T086 [P] [US2] Implement \_fetch_timings_by_city() in AlAdhanClient (HTTP GET /timingsByCity)
- [ ] T087 [P] [US2] Add error handling for API errors (parse API error messages for city not found, etc.)

### Client Methods - Address Lookup

- [ ] T088 [P] [US2] Implement get_timings_by_address() in AlAdhanClient (date, address, method) → TimingsResponse
- [ ] T089 [P] [US2] Add basic validation to get_timings_by_address() (non-null, non-empty string)
- [ ] T090 [P] [US2] Implement \_fetch_timings_by_address() in AlAdhanClient (HTTP GET /timingsByAddress)
- [ ] T091 [P] [US2] Add error handling for API errors (parse API messages for geocoding failures)

### Async Client Parity (US2)

- [ ] T092 [P] [US2] Implement async get_timings_by_city() in AsyncAlAdhanClient
- [ ] T093 [P] [US2] Implement async get_timings_by_address() in AsyncAlAdhanClient
- [ ] T094 [P] [US2] Add identical validation and error handling to async methods

### Integration Tests (US2)

- [ ] T095 [P] [US2] Add test_get_timings_by_city_sync() in test_prayer_times.py (Dubai, UAE)
- [ ] T096 [P] [US2] Add test_get_timings_by_city_async() in test_prayer_times.py
- [ ] T097 [P] [US2] Add test_get_timings_by_address_sync() in test_prayer_times.py
- [ ] T098 [P] [US2] Add test_ambiguous_city_name() (London with country specification)
- [ ] T099 [P] [US2] Add test_invalid_city_error_message() (non-existent city)
- [ ] T100 [P] [US2] Add test_geocoding_failure_error() (invalid address)

**Completion Criteria**: City and address lookup methods work in both sync/async clients, US2 acceptance scenarios pass

---

## Phase 5: User Story 3 (P2) - Monthly/Yearly Prayer Calendar

**Goal**: Retrieve prayer times for entire month/year at once (batch operations)

**Independent Test**: Request October 2025 calendar, verify all 31 days returned with complete data

### Models (US3)

- [ ] T101 [P] [US3] Create CalendarResponse model in aladhan/models/prayer_times.py (data: List[TimingsResponse])
- [ ] T102 [P] [US3] Add calendar-specific validation (all days present, chronological order)

### Client Methods - Calendar

- [ ] T103 [P] [US3] Implement get_calendar() in AlAdhanClient (year, month, latitude, longitude, method) → CalendarResponse
- [ ] T104 [P] [US3] Add basic validation to get_calendar() (month 1-12, non-null parameters, use GeoCoordinates)
- [ ] T105 [P] [US3] Implement \_fetch_calendar() in AlAdhanClient (HTTP GET /calendar/{year}/{month})
- [ ] T106 [P] [US3] Add batch response parsing with Pydantic validation in \_fetch_calendar()

### Client Methods - Yearly Calendar

- [ ] T107 [P] [US3] Implement get_calendar_yearly() in AlAdhanClient (year, latitude, longitude, method) → CalendarResponse
- [ ] T108 [P] [US3] Implement \_fetch_calendar_yearly() in AlAdhanClient (HTTP GET /calendar/{year})
- [ ] T109 [P] [US3] Add Pydantic parsing for annual calendar response (handles 365/366 days automatically)

### Async Client Parity (US3)

- [ ] T110 [P] [US3] Implement async get_calendar() in AsyncAlAdhanClient
- [ ] T111 [P] [US3] Implement async get_calendar_yearly() in AsyncAlAdhanClient

### Integration Tests (US3)

- [ ] T112 [P] [US3] Add test_get_monthly_calendar_sync() in test_prayer_times.py (31 days)
- [ ] T113 [P] [US3] Add test_get_monthly_calendar_async() in test_prayer_times.py
- [ ] T114 [P] [US3] Add test_get_yearly_calendar_sync() in test_prayer_times.py (365 days)
- [ ] T115 [P] [US3] Add test_calendar_caching_performance() (verify faster than 30 individual requests)
- [ ] T116 [P] [US3] Add test_calendar_includes_hijri_dates() (metadata validation)

**Completion Criteria**: Calendar methods retrieve month/year data correctly, caching improves batch performance, US3 acceptance scenarios pass

---

## Phase 6: User Story 4 (P3) - Hijri Calendar Conversions

**Goal**: Convert between Gregorian/Hijri dates and retrieve Islamic holidays

**Independent Test**: Convert known Gregorian date to Hijri, verify accuracy; retrieve 2025 Islamic holidays

### Models (US2)

- [ ] T117 [P] [US4] Create HijriConversionResponse model in aladhan/models/dates.py (hijri: HijriDate, gregorian: GregorianDate)
- [ ] T118 [P] [US4] Create IslamicHoliday model in aladhan/models/dates.py (name_ar, name_en, hijri_date, gregorian_date)
- [ ] T119 [P] [US4] Create HolidaysResponse model in aladhan/models/dates.py (data: List[IslamicHoliday])

### Client Methods - Conversions

- [ ] T120 [P] [US4] Implement gregorian_to_hijri() in AlAdhanClient (date: datetime.date) → HijriConversionResponse
- [ ] T121 [P] [US4] Implement hijri_to_gregorian() in AlAdhanClient (day, month, year) → HijriConversionResponse
- [ ] T122 [P] [US4] Implement \_fetch_gregorian_to_hijri() in AlAdhanClient (HTTP GET /gToH/{dd-mm-yyyy})
- [ ] T123 [P] [US4] Implement \_fetch_hijri_to_gregorian() in AlAdhanClient (HTTP GET /hToG/{dd-mm-yyyy})

### Client Methods - Holidays

- [ ] T124 [P] [US4] Implement get_islamic_holidays() in AlAdhanClient (hijri_year: int) → HolidaysResponse
- [ ] T125 [P] [US4] Implement get_current_hijri_date() in AlAdhanClient () → HijriDate
- [ ] T126 [P] [US4] Implement get_next_islamic_holiday() in AlAdhanClient () → IslamicHoliday
- [ ] T127 [P] [US4] Implement \_fetch_holidays() in AlAdhanClient (HTTP GET /islamicHolidays/{year})

### Async Client Parity (US4)

- [ ] T128 [P] [US4] Implement all Hijri conversion methods in AsyncAlAdhanClient (async versions)
- [ ] T129 [P] [US4] Implement all holiday methods in AsyncAlAdhanClient (async versions)

### Integration Tests (US4)

- [ ] T130 [P] [US4] Create tests/integration/test_hijri_calendar.py
- [ ] T131 [P] [US4] Add test_gregorian_to_hijri_conversion() (known date verification)
- [ ] T132 [P] [US4] Add test_hijri_to_gregorian_conversion() (bidirectional check)
- [ ] T133 [P] [US4] Add test_get_islamic_holidays() (verify Ramadan, Eid included)
- [ ] T134 [P] [US4] Add test_current_hijri_date() (reasonable date check)
- [ ] T135 [P] [US4] Add test_api_error_for_invalid_dates() (verify API returns clear errors for edge cases)

**Completion Criteria**: Hijri conversions accurate (±1 day), Islamic holidays retrieved correctly, US4 acceptance scenarios pass

---

## Phase 7: User Story 5 (P3) - Qibla Direction Calculation

**Goal**: Calculate Qibla direction from any location (bearing angle to Mecca)

**Independent Test**: Provide New York coordinates (40.7128, -74.0060), verify northeast direction (±1°)

### Models (US4)

- [ ] T136 [P] [US5] Create aladhan/models/qibla.py with QiblaDirection model (direction: float, latitude, longitude)
- [ ] T137 [P] [US5] Add bearing angle validation (0-360 degrees) to QiblaDirection model
- [ ] T138 [P] [US5] Create QiblaResponse model in aladhan/models/qibla.py (data: QiblaDirection)

### Client Methods (US5)

- [ ] T139 [P] [US5] Implement get_qibla() in AlAdhanClient (latitude, longitude) → QiblaResponse
- [ ] T140 [P] [US5] Add basic validation to get_qibla() (non-null parameters, use GeoCoordinates validator)
- [ ] T141 [P] [US5] Implement \_fetch_qibla() in AlAdhanClient (HTTP GET /qibla/{lat}/{lon})

### Async Client Parity (US5)

- [ ] T142 [P] [US5] Implement async get_qibla() in AsyncAlAdhanClient

### Integration Tests (US5)

- [ ] T143 [P] [US5] Create tests/integration/test_qibla.py
- [ ] T144 [P] [US5] Add test_qibla_from_north_america() (New York → northeast, ±1° accuracy)
- [ ] T145 [P] [US5] Add test_qibla_from_southeast_asia() (Singapore → northwest, ±1° accuracy)
- [ ] T146 [P] [US5] Add test_qibla_caching() (verify cached response performance)

**Completion Criteria**: Qibla calculation accurate (±1°), works for locations worldwide, US5 acceptance scenarios pass

---

## Phase 8: User Story 6 (P4) - 99 Names of Allah

**Goal**: Provide access to Asma al-Husna (99 Names) with Arabic, transliteration, English meaning

**Independent Test**: Request all 99 names, verify count and data completeness (Arabic + transliteration + meaning)

### Models (US5)

- [ ] T147 [P] [US6] Create aladhan/models/names.py with AllahName model (number, name_ar, transliteration, meaning_en)
- [ ] T148 [P] [US6] Add validation to AllahName (number 1-99, non-empty fields)
- [ ] T149 [P] [US6] Create NamesResponse model in aladhan/models/names.py (data: List[AllahName])

### Client Methods (US6)

- [ ] T150 [P] [US6] Implement get_asma_al_husna() in AlAdhanClient () → NamesResponse
- [ ] T151 [P] [US6] Implement get_allah_name() in AlAdhanClient (number: int) → AllahName
- [ ] T152 [P] [US6] Add basic validation to get_allah_name() (number in 1-99 range)
- [ ] T153 [P] [US6] Implement \_fetch_names() in AlAdhanClient (HTTP GET /asmaAlHusna)

### Async Client Parity (US6)

- [ ] T154 [P] [US6] Implement async get_asma_al_husna() in AsyncAlAdhanClient
- [ ] T155 [P] [US6] Implement async get_allah_name() in AsyncAlAdhanClient

### Integration Tests (US6)

- [ ] T156 [P] [US6] Add test_get_all_99_names() in test_prayer_times.py (verify count = 99)
- [ ] T157 [P] [US6] Add test_get_specific_name() (retrieve name #1: Ar-Rahman)
- [ ] T158 [P] [US6] Add test_names_include_arabic_transliteration_english()
- [ ] T159 [P] [US6] Add test_names_caching() (instant response from cache)

**Completion Criteria**: All 99 names retrievable, data complete (Arabic/transliteration/meaning), US6 acceptance scenarios pass

---

## Phase 9: Additional Endpoints & Completeness

**Goal**: Implement remaining API endpoints for 100% coverage

### Calculation Methods Info

- [ ] T160 [P] Implement get_all_methods() in AlAdhanClient () → Dict[str, MethodInfo]
- [ ] T161 [P] Implement get_method_info() in AlAdhanClient (method_id: str) → MethodInfo
- [ ] T162 [P] Implement async versions in AsyncAlAdhanClient

### Timezone & Time Utilities

- [ ] T163 [P] Implement get_current_timestamp() in AlAdhanClient (zone: str) → int
- [ ] T164 [P] Implement get_current_time() in AlAdhanClient (zone: str) → datetime
- [ ] T165 [P] Implement async versions in AsyncAlAdhanClient

### Contract Tests (Endpoint Coverage)

- [ ] T166 Create tests/contract/ directory
- [ ] T167 Create tests/contract/test_endpoints.py
- [ ] T168 Add test_all_endpoints_implemented() (verify 30+ endpoints have client methods)
- [ ] T169 Add test_sync_async_parity() (verify identical API surface)

**Completion Criteria**: 100% API endpoint coverage achieved, contract tests verify completeness

---

## Phase 10: Polish & Cross-Cutting Concerns

**Goal**: Documentation, final optimizations, production readiness

### Documentation

- [ ] T170 [P] Add docstrings to all public methods in AlAdhanClient (Google-style)
- [ ] T171 [P] Add docstrings to all public methods in AsyncAlAdhanClient
- [ ] T172 [P] Add docstrings to all Pydantic models (field descriptions)
- [ ] T173 [P] Update README.md with installation instructions (uv add aladhan-client)
- [ ] T174 [P] Add usage examples to README.md (sync and async code snippets)
- [ ] T175 [P] Create docs/ directory with mkdocs structure
- [ ] T176 [P] Create docs/quickstart.md with getting started guide
- [ ] T177 [P] Create docs/api/ with API reference documentation

### Type Checking & Linting

- [ ] T178 Run mypy on entire codebase (mypy aladhan/ tests/) - verify zero errors
- [ ] T179 Run ruff linting (ruff check aladhan/ tests/) - verify zero warnings
- [ ] T180 Run ruff formatting (ruff format aladhan/ tests/) - ensure consistent style
- [ ] T181 Configure mypy strict mode checks in pyproject.toml
- [ ] T182 Configure ruff line length and import ordering in pyproject.toml

### Test Suite Optimization

- [ ] T183 Run full test suite (pytest) - verify <5s execution time
- [ ] T184 Measure test coverage (pytest --cov=aladhan) - verify >80% coverage
- [ ] T185 Create tests/fixtures/api_responses/ directory
- [ ] T186 Add sample API response fixtures for offline testing (optional)
- [ ] T187 Optimize slow tests (identify and improve tests >100ms)

### Caching Validation

- [ ] T188 [P] Add test_cache_hit_rate() in tests/integration/test_caching.py (verify >95%)
- [ ] T189 [P] Add test_etag_conditional_requests() (verify 304 Not Modified handling)
- [ ] T190 [P] Add test_cache_bypass_option() (verify fresh API calls when requested)
- [ ] T191 [P] Add test_multiple_cache_backends() (in-memory, filesystem)

### Performance Validation

- [ ] T192 Run benchmark suite (pytest tests/performance/) - verify targets met
- [ ] T193 Validate cached request performance (<1ms p95)
- [ ] T194 Validate uncached request performance (<500ms p95)
- [ ] T195 Validate rate limit compliance (stress test with 100 requests)

### Final Validation

- [ ] T196 Run constitutional compliance checks (zero mocks in-process, type safety)
- [ ] T197 Verify all 35 functional requirements (FR-001 to FR-035) implemented
- [ ] T198 Verify all 21 success criteria (SC-001 to SC-021) met
- [ ] T199 Run checklist validation (checklists/comprehensive.md)
- [ ] T200 Update CLAUDE.md with implementation notes
- [ ] T201 Create CHANGELOG.md with v0.1.0 release notes

**Completion Criteria**: All documentation complete, tests pass, performance targets met, production-ready

---

## Task Summary

**Total Tasks**: 201
**By Phase**:

- Phase 1 (Setup): 17 tasks
- Phase 2 (Foundational): 24 tasks
- Phase 3 (US1 - P1): 42 tasks
- Phase 4 (US2 - P2): 17 tasks
- Phase 5 (US3 - P2): 14 tasks
- Phase 6 (US4 - P3): 19 tasks (simplified validation)
- Phase 7 (US5 - P3): 11 tasks
- Phase 8 (US6 - P4): 13 tasks
- Phase 9 (Additional): 10 tasks
- Phase 10 (Polish): 32 tasks

**Parallelization Opportunities**: 101 tasks marked [P] (52.3% parallelizable)

**Validation Approach**: Minimal client-side validation (basic sanity checks only). Complex business rules delegated to API for reduced maintenance burden.

**By User Story**:

- US1 (P1): 42 tasks (Prayer times by coordinates including school & tune parameters)
- US2 (P2): 17 tasks (City/address lookup)
- US3 (P2): 14 tasks (Calendar)
- US4 (P3): 19 tasks (Hijri conversions - simplified validation)
- US5 (P3): 11 tasks (Qibla)
- US6 (P4): 13 tasks (99 Names)

---

## Dependency Graph

```text
Phase 1 (Setup)
    ↓
Phase 2 (Foundational) - blocking for all user stories
    ↓
┌───────┬───────┬───────┬───────┬───────┐
│  US1  │  US2  │  US3  │  US4  │  US5  │  US6
│  (P1) │  (P2) │  (P2) │  (P3) │  (P3) │  (P4)
│       │   ↓   │   ↓   │       │       │
│       │  US1  │  US1  │       │       │
└───────┴───────┴───────┴───────┴───────┴───────┘
    ↓
Phase 9 (Additional Endpoints)
    ↓
Phase 10 (Polish)
```

**Dependencies**:

- **US2 depends on US1**: Uses same TimingsResponse model and client infrastructure
- **US3 depends on US1**: Uses same models, extends with calendar responses
- **US4, US5, US6**: Independent (can be implemented in any order after foundational)

---

## Parallel Execution Examples

### Phase 2 (Foundational) - Parallel Execution

Execute in parallel:

- Batch A: T018-T023 (all exception classes)
- Batch B: T024-T030 (all data models)
- Batch C: T036-T041 (all caching infrastructure)

Sequential: T031-T035 (HTTP client foundation - blocking for Batch C)

### Phase 3 (US1) - Parallel Execution

Execute in parallel:

- Batch A: T042-T047 (all models)
- Batch B: T048-T055 (sync client)
- Batch C: T056-T062 (async client)
- Batch D: T065-T071 (integration tests)
- Batch E: T072-T075 (performance benchmarks)

Sequential: T063-T064 (package exports - after Batch B and C complete)

### Phase 10 (Polish) - Parallel Execution

Execute in parallel:

- Batch A: T163-T170 (all documentation tasks)
- Batch B: T178-T180 (test suite optimization)
- Batch C: T181-T184 (caching validation tests)

Sequential: T171-T177 (type checking/linting), T185-T188 (performance validation), T189-T194 (final validation)

---

## Implementation Strategy

### MVP Scope (Minimum Viable Product)

**Target**: User Story 1 (P1) only - Prayer times by coordinates

**Includes**:

- Phase 1: Setup (T001-T017)
- Phase 2: Foundational (T018-T041)
- Phase 3: US1 (T042-T083)

**MVP Task Count**: 83 tasks (Phases 1-3: Setup + Foundational + US1 including school & tune parameters)
**Estimated Effort**: 1-2 weeks for experienced Python developer
**Deliverable**: Functional sync/async client for prayer times by coordinates with caching, basic input validation (coordinates, non-null params)

### Incremental Delivery

**Iteration 1 (MVP)**: US1 - Prayer times by coordinates (83 tasks including school & tune)
**Iteration 2**: US2 - City/address lookup (extends MVP, adds 17 tasks)
**Iteration 3**: US3 - Calendar (adds batch operations, 14 tasks)
**Iteration 4**: US4 - Hijri conversions (independent utility, 19 tasks - simplified validation)
**Iteration 5**: US5, US6 - Qibla + Names (independent features, 24 tasks)
**Iteration 6**: Phase 9-10 - Completeness + Polish (42 tasks)

### Test-First Approach (Optional)

If following TDD:

1. Write integration test (e.g., T067 for US1)
2. Implement models (T042-T047)
3. Implement client methods (T048-T062)
4. Watch tests pass
5. Refactor if needed (tests protect against regressions)

**Constitution Compliance**: Zero mocks for in-process components - integration tests use real API (mocked network layer only for offline testing)

---

## Validation Checklist

Before marking phase complete, verify:

- [ ] All tasks in phase completed
- [ ] Integration tests pass for that user story
- [ ] Type checking passes (mypy aladhan/)
- [ ] Linting passes (ruff check aladhan/)
- [ ] Performance targets met for cached/uncached requests
- [ ] User story acceptance scenarios verified
- [ ] Independent test criteria satisfied
- [ ] No regressions in previous user stories

---

**Ready for Implementation**: Tasks are atomic, file paths are explicit, dependencies are clear, parallel execution opportunities identified. Start with Phase 1 (Setup) and proceed sequentially through phases, parallelizing within each phase as marked.
