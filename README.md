# AlAdhan Python Client

> **Modern, type-safe Python client for the AlAdhan Prayer Times API**

[![Python 3.14+](https://img.shields.io/badge/python-3.14+-blue.svg)](https://www.python.org/downloads/)
[![License: GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-yellow.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Status: Planning](https://img.shields.io/badge/Status-Planning-orange.svg)](https://github.com/inventrohyder/aladhan-python-client)

**🚧 Project Status:** Currently in **Phase 0 - Planning & Research**

This is a planned open-source Python client for the [AlAdhan.com API](https://aladhan.com), designed to provide:

- **100% API coverage** (30+ endpoints vs 20% in existing clients)
- **Type-safe Pydantic models** (validated responses, full IDE autocomplete)
- **Separate sync/async clients** (no type confusion)
- **Built-in caching** (prayer times are deterministic)
- **Python 3.14+** (cutting-edge features: lazy annotations, free-threading, JIT)

## OpenAPI Schema

This project includes a **complete, validated OpenAPI 3.1.0 schema** (`openapi.yaml`) for the AlAdhan API:

- ✅ **All 30+ endpoints** documented
- ✅ **Validated against live API** (9+ endpoints tested)
- ✅ **Ready for code generation** (Pydantic models, clients)
- ✅ **Free for community use** (GPL-3.0)

The schema was built through systematic API exploration and validated against actual responses to ensure accuracy. It can be used with tools like `openapi-python-client`, `datamodel-code-generator`, or any OpenAPI-compatible tooling.

**Community Contribution:** This schema is shared with the open-source community to benefit all Python prayer time projects.

## Why This Project?

### Current State of Python Prayer Time Libraries

| Package                   | Weekly Downloads | Last Update | API Coverage       | Type Safety        | Async Support |
| ------------------------- | ---------------- | ----------- | ------------------ | ------------------ | ------------- |
| `prayer-times-calculator` | 1,619            | 2023        | 1 endpoint         | ❌ Dict            | ❌ Sync only  |
| `aladhan.py`              | 293              | 2024        | ~20% (6 endpoints) | ⚠️ Union confusion | ⚠️ Dual-mode  |
| Other packages            | <100             | 2020-2023   | Varies             | ❌ No              | ❌ No         |

**Total market:** ~10,000 monthly downloads across all packages
**Problem:** Zero maintenance in 2024-2025, missing 60-80% of API functionality

### What Makes This Different

| Feature             | This Client               | Existing Clients                 |
| ------------------- | ------------------------- | -------------------------------- |
| **API Coverage**    | 100% (30+ endpoints)      | 20-40% (1-6 endpoints)           |
| **Type Safety**     | ✅ Full Pydantic models   | ❌ Plain dicts or ⚠️ Union types |
| **Validation**      | ✅ Runtime + compile-time | ❌ None                          |
| **IDE Support**     | ✅ Full autocomplete      | ❌ Limited                       |
| **Sync Client**     | ✅ Separate class         | ⚠️ Dual-mode confusion           |
| **Async Client**    | ✅ Separate class         | ⚠️ Dual-mode confusion           |
| **Caching**         | ✅ Built-in (planned)     | ❌ None                          |
| **Python Version**  | 3.14+                     | 3.8-3.11                         |
| **Package Manager** | UV (10-100x faster)       | pip/poetry                       |
| **Maintenance**     | 2025+                     | 2020-2024                        |

## Planned Features

### Prayer Times

- ✅ By coordinates, city, address
- ✅ Single day or full calendar (monthly/yearly)
- ✅ Next prayer time
- ✅ Custom calculation methods
- ✅ Tuning and adjustments
- ✅ Multiple schools (Shafi, Hanafi)
- ✅ High latitude adjustments

### Hijri Calendar

- ✅ Gregorian ↔ Hijri conversion
- ✅ Islamic holidays
- ✅ Current Hijri date
- ✅ Special Islamic dates

### Other Features

- ✅ Qibla direction calculation
- ✅ 99 Names of Allah (Asma al-Husna)
- ✅ Current time/date utilities
- ✅ Calculation methods info

### Technical Features

- ✅ **Type-safe Pydantic models** (full validation)
- ✅ **Separate sync/async clients** (no type confusion)
- ✅ **Built-in caching** (deterministic results)
- ✅ **Comprehensive error handling**
- ✅ **100% test coverage**
- ✅ **Full documentation**
- ✅ **Examples for all use cases**

## Planned API

### Synchronous Client

```python
from aladhan import AlAdhanClient
from datetime import date

# Create client
client = AlAdhanClient()

# Get prayer times by coordinates
times = client.get_timings_by_coordinates(
    date=date(2025, 10, 27),
    latitude=25.2854,
    longitude=51.5310,
    method="ISNA"
)

# Type-safe access (validated Pydantic models)
print(f"Fajr: {times.timings.Fajr}")  # Full IDE autocomplete
print(f"Dhuhr: {times.timings.Dhuhr}")
print(f"Hijri: {times.date.hijri.date}")

# Get prayer times by city
times = client.get_timings_by_city(
    date=date(2025, 10, 27),
    city="Dubai",
    country="United Arab Emirates",
    method="MAKKAH"
)

# Get monthly calendar
calendar = client.get_calendar(
    year=2025,
    month=10,
    latitude=25.2854,
    longitude=51.5310,
    method="ISNA"
)

for day in calendar:
    print(f"{day.date.readable}: Fajr at {day.timings.Fajr}")

# Qibla direction
qibla = client.get_qibla(latitude=40.7128, longitude=-74.0060)
print(f"Qibla direction from NYC: {qibla.direction}°")

# Hijri calendar conversion
hijri = client.gregorian_to_hijri(date(2025, 10, 27))
print(f"Gregorian: {hijri.gregorian.date}")
print(f"Hijri: {hijri.hijri.date}")
```

### Asynchronous Client

```python
from aladhan import AsyncAlAdhanClient
from datetime import date
import asyncio

async def main():
    # Create async client
    async with AsyncAlAdhanClient() as client:
        # All methods return awaitables
        times = await client.get_timings_by_coordinates(
            date=date(2025, 10, 27),
            latitude=25.2854,
            longitude=51.5310,
            method="ISNA"
        )

        print(f"Fajr: {times.timings.Fajr}")  # Same type-safe access

        # Concurrent requests
        dubai_times, makkah_times = await asyncio.gather(
            client.get_timings_by_city(date.today(), "Dubai", "UAE", "DUBAI"),
            client.get_timings_by_city(date.today(), "Makkah", "Saudi Arabia", "MAKKAH")
        )

asyncio.run(main())
```

### Why Separate Sync/Async Clients?

**Problem with dual-mode pattern** (used by `aladhan.py`):

```python
# Confusing type signature
def get_timings(self, ...) -> Union[Timings, Awaitable[Timings]]:
    ...

# Type checker can't help you
times = client.get_timings(...)  # What type is this?
await times  # Runtime error if sync mode!
times.Fajr  # Runtime error if async mode!
```

**Our solution:**

```python
# Clear, type-safe
client = AlAdhanClient()  # Sync
times: Timings = client.get_timings(...)  # No await needed

async_client = AsyncAlAdhanClient()  # Async
times: Timings = await async_client.get_timings(...)  # Clear awaitable
```

## Installation (Planned)

```bash
# Using UV (recommended - 10-100x faster than pip)
uv pip install aladhan-client

# Using pip
pip install aladhan-client

# Development installation
git clone https://github.com/inventrohyder/aladhan-python-client
cd aladhan-python-client
uv sync --all-extras
```

## Requirements

- **Python 3.14+** (uses cutting-edge features)
- **httpx** (modern HTTP client)
- **Pydantic v2** (validation and type safety)

## Development Status

### Phase 0: Planning & Research ✅ (Completed)

- [x] Competitive analysis of existing Python packages
- [x] Gap analysis report (641 lines)
- [x] OpenAPI schema validation (9 endpoints tested)
- [x] Architecture documentation (CLAUDE.md)
- [x] User-facing documentation (README.md)

### Phase 1: Core Implementation 🚧 (Next)

- [ ] Project scaffolding with UV
- [ ] Pydantic models from OpenAPI schema
- [ ] Base HTTP client (sync + async)
- [ ] Prayer times endpoints
- [ ] Error handling
- [ ] Unit tests (>80% coverage)

### Phase 2: Extended Features

- [ ] Calendar endpoints
- [ ] Hijri calendar conversions
- [ ] Qibla direction
- [ ] Asma al-Husna
- [ ] Utility endpoints
- [ ] Integration tests

### Phase 3: Production Ready

- [ ] Built-in caching
- [ ] Rate limiting
- [ ] Retry logic
- [ ] Comprehensive documentation
- [ ] Examples and tutorials
- [ ] 100% test coverage
- [ ] PyPI publication

## Contributing

This project welcomes contributions from the Muslim developer community! We're using:

- **Graphite (gt)** for stacked commits (trunk-based development)
- **trunk.io** for linting (orchestrates ruff, mypy, black)
- **UV** for package management
- **pytest** for testing
- **mkdocs** for documentation

See `CLAUDE.md` for detailed architecture and development guidelines.

## License

GPL-3.0 - Same as AlAdhan.com API

This is a client library only. The AlAdhan.com API and its data are provided by [Islamic Network](https://islamic.network).

## Acknowledgments

- **Islamic Network** - AlAdhan.com API and infrastructure
- **Hamid Zarrabi-Zadeh** - Original PrayTimes.js algorithm
- **Muslim Developer Community** - Inspiration and feedback

## Related Projects

- [AlAdhan.com](https://aladhan.com) - Web interface
- [AlAdhan API](https://aladhan.com/api) - REST API documentation
- [PHP Client](https://github.com/islamic-network/aladhan-api-client-php) - Official PHP client
- [Prayer Times Library](https://github.com/islamic-network/prayer-times) - Core calculation engine

---

السلام عليكم ورحمة الله وبركاته

Built with ❤️ for the Muslim Ummah
