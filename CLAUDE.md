# Al-Adhan Python Client - Project Documentation

## Project Overview

**Status:** Planning Phase
**Start Date:** 2025-10-26
**Goal:** Create a cutting-edge Python client for the AlAdhan.com API using Python 3.14, UV package manager, and modern best practices

This is the official Python client library for accessing Islamic prayer times, Hijri calendar data, and geocoding through the AlAdhan.com REST API. It's designed to be the most advanced prayer times client in any programming language.

## Strategic Context

### The Al-Adhan Ecosystem

This Python client is part of the larger Al-Adhan ecosystem:

1. **PHP Library** - Local astronomical calculations (math-based)
   - Repository: <https://1x.ax/islamic-network/libraries/prayer-times>
2. **PHP Client** - AlAdhan.com API wrapper
   - Repository: <https://1x.ax/islamic-network/aladhan/api-client-php>
3. **Python Client** - Modern Python API wrapper (THIS PROJECT)
4. **Python Library** (Future) - Python port of astronomical calculations

### Why Python?

**Market Gap:**

- PHP dominates web (WordPress, Laravel)
- Python dominates everywhere else (data science, ML, automation, IoT, modern web)
- **Problem:** Python developers have NO official way to access AlAdhan features

**Use Cases Python Enables:**

- IoT/Embedded (Raspberry Pi prayer displays)
- Modern web apps (Django, Flask, FastAPI)
- Data analysis (analyze prayer patterns with pandas)
- Automation (prayer notifications, scheduling)
- CLI tools (command-line utilities)
- Discord/Slack bots (prayer reminders)

### Strategic Approach

#### Phase (Current):\*\* API Client

- HTTP wrapper for AlAdhan.com API
- Both sync + async support
- Production-grade features (caching, retries, rate limiting)
- CLI tool

#### Phase (Future):\*\* Astronomical Library

- Port PHP library calculations to Python
- Enable offline prayer time calculations
- NumPy/SciPy integration for scientific computing

## Phase 0: Competitive Research (COMPLETE ✅)

**Status:** Week 1 research completed (2025-10-26)
**Outcome:** Validated build-new approach with clear differentiation strategy

### Research Conducted

**Gap Analysis Report:**

- Analyzed 7 existing Python packages for AlAdhan/prayer times
- Reviewed 641-line comprehensive market research report
- Studied AlAdhan API documentation (30+ endpoints)
- Examined download statistics and user preferences

**Codebase Analysis:**

- **aladhan.py** - Most mature competitor (async/sync support, 20% API coverage)
- **prayer-times-calculator** - Market leader (1,619 weekly downloads, simple but limited)

**Community Outreach:**

- Opened collaborative issue on aladhan.py: <https://github.com/HETHAT/aladhan.py/issues/8>
- Respectful inquiry about contributing vs building separate
- Waiting period: 7 days for maintainer response

### Key Findings: aladhan.py (Technical Excellence)

**Architecture Analysis:**

**✅ What They Do Well:**

1. **Dual sync/async pattern** via `is_async` flag:

   ```python
   # Sync mode
   client = aladhan.Client(is_async=False)
   times = client.get_timings_by_address("New York")

   # Async mode
   client = aladhan.Client(is_async=True)
   times = await client.get_timings_by_address("New York")
   ```

2. **Context manager support** for both modes:

   ```python
   with aladhan.Client() as client:  # sync
       pass

   async with aladhan.Client(is_async=True) as client:  # async
       pass
   ```

3. **Comprehensive parameter support** - Full AlAdhan API parameters
4. **Type-safe enums** - Constants for methods, schools, etc.
5. **Excellent documentation** - ReadTheDocs with examples
6. **Rate limit handling** - Automatic retry on 429 responses

**❌ Limitations Identified:**

1. **Type safety issue** - Returns `Union[T, Awaitable[T]]` which confuses type checkers:

   ```python
   TimingsR = Union[Timings, Awaitable[Timings]]  # Type checkers can't narrow this
   ```

2. **Only ~20% API coverage** - Missing:
   - Hijri calendar endpoints (6+ endpoints)
   - Qibla direction (2 endpoints)
   - Date conversions (4+ endpoints)
   - Asma Al-Husna (2 endpoints)
   - Utility endpoints (4+ endpoints)

3. **Custom classes instead of Pydantic** - No runtime validation:

   ```python
   @dataclass
   class Timings:  # Custom class, not Pydantic
       fajr: str   # String, not datetime.time
       ...
   ```

4. **No caching layer** - Every call hits API
5. **No retry logic** beyond rate limits
6. **No CLI tool**
7. **Small community** - 13 stars, 1 fork, unclear 2024-2025 activity

### Key Findings: prayer-times-calculator (Market Leader)

**Why It Dominates (1,619 weekly downloads = 60-65% market share):**

**✅ Success Factors:**

1. **Dead simple API** - One class, one method, no complexity
2. **Home Assistant integration** - Key driver of adoption
3. **Comprehensive parameters** - Supports all AlAdhan features (tuning, custom methods, etc.)
4. **Just works** - Reliable despite simplicity

**❌ Critical Weaknesses:**

1. **Synchronous only** - Blocks with `requests.get(url, timeout=10)`
2. **No type hints** - Returns plain `dict[str, Any]`
3. **No caching** - Every call hits API
4. **No retry logic** - Single 10-second timeout, no exponential backoff
5. **No validation** - Raw dictionary responses
6. **Abandoned** - No commits in 12+ months (2024-2025 inactive)
7. **20-parameter constructor** - Awkward API design

**Code Example:**

```python
# Their approach - constructor-heavy
calculator = PrayerTimesCalculator(
    latitude=51.5074,
    longitude=-0.1278,
    calculation_method="isna",
    date="2025-10-26",
    school="hanafi",
    midnightMode="standard",
    # ... 14 more parameters
)
times = calculator.fetch_prayer_times()  # Returns dict
```

### Our Competitive Advantages (Validated)

Based on competitive research, our differentiation is **real and valuable**:

| Feature            | prayer-times-calculator | aladhan.py         | **Our Client**                         |
| ------------------ | ----------------------- | ------------------ | -------------------------------------- |
| **Market Share**   | 60-65% (1,619/week)     | ~4% (moderate)     | Target: 25% in Y1                      |
| **API Coverage**   | Prayer times only       | ~20%               | **100% (all 30+ endpoints)**           |
| **Async Support**  | ❌ Blocks               | ✅ Yes (dual mode) | **✅ Separate clients (better types)** |
| **Data Models**    | Plain dict              | Custom classes     | **✅ Pydantic v2 (validation)**        |
| **Caching**        | ❌ No                   | ❌ No              | **✅ In-memory + Redis**               |
| **Retry Logic**    | ❌ 10s timeout          | Rate limits only   | **✅ Exponential backoff**             |
| **Type Safety**    | ❌ No hints             | ⚠️ Union[T, Aw[T]] | **✅ Full type hints**                 |
| **CLI Tool**       | ❌ No                   | ❌ No              | **✅ Full CLI**                        |
| **Maintenance**    | ❌ Abandoned 12mo       | ❓ Unclear 2024-25 | **✅ Committed monthly**               |
| **Python Version** | 3.8+                    | 3.7+               | **✅ 3.14+ (modern)**                  |
| **Home Assistant** | ✅ Integrated           | ❌ No              | **📋 Phase 5 target**                  |

### Strategic Decisions Based on Research

#### 1. Build Separate (Not Fork/Contribute)

**Rationale:**

- Would need to rebuild 80% of aladhan.py anyway (Pydantic refactor + 20 endpoints + caching)
- Retrofitting Pydantic = breaking change for existing users
- Better to design for Pydantic v2 + caching + Python 3.14 from day 1
- Awaiting maintainer response, but preparing to build independently

#### 2. Target Python 3.14+

**Rationale:**

- Lazy annotations (PEP 649) - no more `from __future__ import annotations`
- Free-threading (PEP 779) - optional no-GIL mode
- Experimental JIT - 5-10% performance boost
- Market already abandoned (no 2024-2025 updates), so breaking with 3.7-3.13 is acceptable
- Position as "modern, cutting-edge" client

#### 3. Separate Sync/Async Clients (Not Dual Mode)

**Problem with aladhan.py's approach:**

```python
TimingsR = Union[Timings, Awaitable[Timings]]  # Type checkers confused
```

**Our approach:**

```python
# Clear, type-safe
from aladhan import AlAdhanClient, AsyncAlAdhanClient

client = AlAdhanClient()  # Returns Timings directly
times: Timings = client.get_timings_by_city("Dubai", "UAE")

async_client = AsyncAlAdhanClient()  # Returns awaitable
times: Timings = await async_client.get_timings_by_city("Dubai", "UAE")
```

**Benefits:**

- Type checkers understand immediately (no Union confusion)
- Simpler mental model
- Better IDE autocomplete
- Follows modern Python async conventions

#### 4. Target Home Assistant in Phase 5

**Insight:** 60-65% of market comes from Home Assistant integration (prayer-times-calculator)

**Strategy:**

- Phase 1-4: Build comprehensive, production-ready client
- Phase 5: Create Home Assistant custom component
- Market entry: "Better maintained alternative with async support"

### What We Learned (Actionable)

**From aladhan.py (Study & Emulate):**

- ✅ Context manager design (`__enter__`, `__aenter__`)
- ✅ Type-safe enums for constants (Method, School)
- ✅ ReadTheDocs documentation structure
- ✅ Comprehensive parameter support (Parameters class)
- ✅ Rate limit auto-handling pattern

**From prayer-times-calculator (Study & Avoid):**

- ✅ Why simplicity matters (their success despite limitations)
- ❌ Constructor parameter explosion (our approach: builder pattern)
- ❌ Blocking sync calls (we provide async)
- ❌ No validation (we use Pydantic)

**Market Validation:**

- ✅ 10,000 monthly downloads = sustainable niche
- ✅ AlAdhan API serves 16M requests/day globally = demand proven
- ✅ ZERO 2024-2025 maintenance = massive opportunity
- ✅ No Pydantic/caching/async combination exists = clear differentiation

### Updated Roadmap Post-Research

#### Phase 0: Research ✅ COMPLETE: 1)

- ✅ Analyzed existing packages
- ✅ Studied competitor codebases
- ✅ Validated market gap
- ✅ Opened collaborative dialogue
- ✅ Confirmed build-separate strategy

#### Phase : MVP - Prayer Times Parity: s 2-3)\*\*

- [ ] Match aladhan.py's prayer times coverage (10 endpoints)
- [ ] Exceed with Pydantic v2 models + caching
- [ ] Both sync + async (separate clients)
- [ ] Target: Drop-in replacement for prayer-times-calculator

#### Phase : Complete API Coverage: s 4-6)\*\*

- [ ] Add missing 60-80% of endpoints
- [ ] Target: Only Python client with 100% coverage

#### Phase : Production Features: s 7-9)\*\*

- [ ] Retry logic, circuit breaker, Redis cache
- [ ] 90%+ test coverage

#### Phase : Developer Experience: s 10-11)\*\*

- [ ] CLI tool, migration guides, ReadTheDocs

#### Phase : Ecosystem & Adoption: 12+)\*\*

- [ ] Home Assistant custom component (key!)
- [ ] FastAPI/Django integrations

## Technology Stack (Bleeding Edge 2025)

### Core Technologies

**Python 3.14** (Released October 7, 2025)

- **Why:** Latest features, performance improvements, modern syntax
- **Key Features:**
  - PEP 649: Lazy annotations (no more `from __future__ import annotations`)
  - PEP 750: Template strings for safer string processing
  - PEP 758: Simpler exception syntax
  - PEP 779: Free-threaded Python (optional no-GIL mode)
  - Experimental JIT compiler (5-10% performance boost)
- **Documentation:** <https://docs.python.org/3.14/whatsnew/3.14.html>

**UV Package Manager** (Astral)

- **Why:** 10-100x faster than pip, all-in-one tooling
- **Key Features:**
  - Rust-powered performance
  - Replaces pip, pip-tools, pipx, poetry, pyenv, virtualenv
  - Workspace support (like Cargo)
  - Lock files for reproducible builds
  - Python version management
  - PEP 723 script support (inline dependencies)
- **Documentation:** <https://docs.astral.sh/uv/>

**Pydantic v2.12+** (Rust-powered validation)

- **Why:** Type safety, validation, IDE autocomplete
- **Key Features:**
  - Rust core for extreme performance
  - Python 3.14 lazy annotation support
  - Immutable models with frozen config
  - JSON schema generation
  - model_validate_json() for direct JSON parsing
- **Documentation:** <https://docs.pydantic.dev/>

**httpx** (Modern HTTP client)

- **Why:** Both sync + async, HTTP/2 support
- **Key Features:**
  - Single library for sync and async
  - HTTP/2 support (better performance)
  - Connection pooling
  - Timeout management
  - Type hints throughout
- **Documentation:** <https://www.python-httpx.org/>

### Additional Dependencies

**Production:**

- **tenacity** - Retry logic with exponential backoff
- **ratelimit** - Request rate limiting
- **typer** - Modern CLI framework (from Pydantic team)
- **rich** - Beautiful terminal output

**Optional:**

- **redis** - Distributed caching support

**Development:**

- **pytest** - Testing framework
- **pytest-asyncio** - Async test support
- **pytest-cov** - Code coverage
- **trunk.io** - Mega-linter (replaces ruff, mypy, black, isort, flake8, etc.)

## Architecture Design

### Project Structure

```text
python-client/
├── uv.toml                      # UV workspace configuration
├── pyproject.toml               # Main project metadata
├── uv.lock                      # Dependency lock file (auto-generated)
├── README.md                    # User-facing documentation
├── CLAUDE.md                    # This file - development documentation
├── LICENSE                      # GPL-3.0
├── src/
│   └── aladhan/
│       ├── __init__.py          # Public API exports
│       ├── client.py            # Synchronous client
│       ├── async_client.py      # Asynchronous client
│       ├── models.py            # Pydantic data models
│       ├── enums.py             # Enums (Method, School, etc.)
│       ├── exceptions.py        # Custom exceptions
│       ├── cache.py             # TTL caching with optional Redis
│       ├── retry.py             # Exponential backoff logic
│       ├── ratelimit.py         # Rate limiting implementation
│       └── cli.py               # CLI tool (Typer-based)
├── tests/
│   ├── conftest.py              # Pytest configuration
│   ├── test_client.py           # Sync client tests
│   ├── test_async_client.py     # Async client tests
│   ├── test_models.py           # Pydantic model tests
│   ├── test_cache.py            # Caching tests
│   └── test_cli.py              # CLI tests
├── examples/
│   ├── basic.py                 # Simple usage example (PEP 723)
│   ├── async_usage.py           # Async example
│   ├── advanced.py              # Caching, retries, rate limiting
│   └── cli_usage.sh             # CLI examples
└── docs/
    ├── api_reference.md         # API documentation
    ├── user_guide.md            # User guide
    └── migration.md             # Migration from PHP client
```

### API Design Philosophy

**Pythonic Over PHP-like:**

- Use Python conventions (snake_case, not camelCase)
- Client object pattern, not class-per-endpoint
- Type hints everywhere (thanks to Pydantic)
- Context managers for async (`async with`)

**Example Comparison:**

**PHP Client:**

```php
$times = new TimesByCity('Dubai', 'UAE');
$result = $times->get();
```

**Python Client:**

```python
client = AlAdhanClient()
times = client.get_times_by_city('Dubai', 'UAE')
# times is a typed Pydantic model, not a dict!
print(times.timings.fajr)  # datetime.time object
```

### Core Components

#### 1. Synchronous Client (`client.py`)

**Responsibility:** Blocking HTTP requests for simple use cases

**API:**

```python
class AlAdhanClient:
    def __init__(
        self,
        base_url: str = "https://api.aladhan.com",
        method: Method = Method.ISNA,
        school: School = School.SHAFI,
        cache_ttl: int | None = 3600,
        max_retries: int = 3,
        rate_limit: int = 10,
        timeout: int = 30,
        verify_ssl: bool = True,
    ):
        """Initialize synchronous client."""

    def get_times(
        self,
        timestamp: int,
        latitude: float,
        longitude: float,
        timezone: str,
    ) -> PrayerTimesResponse:
        """Get prayer times by coordinates and timestamp."""

    def get_times_by_city(
        self,
        city: str,
        country: str,
        state: str | None = None,
    ) -> PrayerTimesResponse:
        """Get prayer times by city name (easiest method)."""

    def get_times_by_address(
        self,
        address: str,
    ) -> PrayerTimesResponse:
        """Get prayer times by address (with geocoding)."""

    def get_calendar(
        self,
        month: int,
        year: int,
        latitude: float,
        longitude: float,
        timezone: str,
    ) -> list[PrayerTimesResponse]:
        """Get monthly calendar of prayer times."""

    def get_calendar_by_city(
        self,
        month: int,
        year: int,
        city: str,
        country: str,
    ) -> list[PrayerTimesResponse]:
        """Get monthly calendar by city."""

    def hijri_to_gregorian(
        self,
        hijri_date: str,
    ) -> GregorianDate:
        """Convert Hijri date to Gregorian (format: DD-MM-YYYY)."""

    def gregorian_to_hijri(
        self,
        gregorian_date: str,
    ) -> HijriDate:
        """Convert Gregorian date to Hijri (format: DD-MM-YYYY)."""

    def get_next_holiday(self) -> Holiday:
        """Get next Islamic holiday."""

    def get_holidays_by_year(
        self,
        hijri_year: int,
    ) -> list[Holiday]:
        """Get all holidays in a Hijri year."""
```

#### 2. Asynchronous Client (`async_client.py`)

**Responsibility:** Non-blocking HTTP requests for high-performance applications

**API:**

```python
class AsyncAlAdhanClient:
    """Async version of AlAdhanClient - same methods, async/await."""

    async def __aenter__(self):
        """Context manager support."""
        return self

    async def __aexit__(self, *args):
        """Clean up resources."""
        await self.http.aclose()

    async def get_times_by_city(
        self,
        city: str,
        country: str,
    ) -> PrayerTimesResponse:
        """Async version of get_times_by_city."""

    # ... all other methods as async
```

**Usage:**

```python
async with AsyncAlAdhanClient() as client:
    # Concurrent requests
    dubai, london, new_york = await asyncio.gather(
        client.get_times_by_city("Dubai", "UAE"),
        client.get_times_by_city("London", "UK"),
        client.get_times_by_city("New York", "USA"),
    )
```

#### 3. Pydantic Models (`models.py`)

**Responsibility:** Type-safe data models with validation

**Key Models:**

```python
from datetime import time, date
from pydantic import BaseModel, Field, ConfigDict

class PrayerTimings(BaseModel):
    """Prayer times for a single day."""
    model_config = ConfigDict(frozen=True)  # Immutable

    fajr: time
    sunrise: time
    dhuhr: time
    asr: time
    sunset: time
    maghrib: time
    isha: time
    midnight: time
    imsak: time | None = None
    firstthird: time | None = None
    lastthird: time | None = None

class GregorianDate(BaseModel):
    """Gregorian date information."""
    date: date
    format: str
    day: int
    month: str | int
    year: int
    weekday: str

class HijriDate(BaseModel):
    """Hijri (Islamic) date information."""
    date: str
    format: str
    day: int
    month: str | int
    year: int
    weekday: str
    designation: str
    holidays: list[str] = Field(default_factory=list)

class DateInfo(BaseModel):
    """Combined date information."""
    readable: str
    timestamp: int
    gregorian: GregorianDate
    hijri: HijriDate

class Meta(BaseModel):
    """Request metadata."""
    latitude: float
    longitude: float
    timezone: str
    method: dict
    latitudeAdjustmentMethod: str
    midnightMode: str
    school: str
    offset: dict

class PrayerTimesResponse(BaseModel):
    """Complete API response."""
    timings: PrayerTimings
    date: DateInfo
    meta: Meta

    @classmethod
    def from_api_json(cls, json_str: str):
        """Parse JSON directly (Pydantic v2 performance optimization)."""
        return cls.model_validate_json(json_str)

class Holiday(BaseModel):
    """Islamic holiday information."""
    name: str
    date: HijriDate
    gregorian_date: GregorianDate | None = None
```

**Benefits:**

- ✅ Full type safety
- ✅ IDE autocomplete
- ✅ Automatic validation
- ✅ JSON schema generation
- ✅ Immutable (frozen=True)
- ✅ Clear documentation

#### 4. Enums (`enums.py`)

**Responsibility:** Type-safe constants

```python
from enum import IntEnum, StrEnum

class Method(IntEnum):
    """Prayer calculation methods."""
    SHIA_ITHNA_ASHARI = 0
    UNIVERSITY_OF_ISLAMIC_SCIENCES_KARACHI = 1
    ISNA = 2  # Islamic Society of North America
    MWL = 3   # Muslim World League
    MAKKAH = 4  # Umm Al-Qura, Makkah
    EGYPT = 5
    TEHRAN = 7
    GULF = 8
    KUWAIT = 9
    QATAR = 10
    SINGAPORE = 11
    FRANCE = 12
    TURKEY = 13
    RUSSIA = 14
    DUBAI = 15
    JAKIM = 16  # Malaysia
    TUNISIA = 17
    ALGERIA = 18
    KEMENAG = 19  # Indonesia
    MOROCCO = 20
    PORTUGAL = 21
    JORDAN = 22

class School(IntEnum):
    """Asr jurisprudential schools."""
    SHAFI = 0   # Standard - shadow length = object height
    HANAFI = 1  # Shadow length = 2x object height

class LatitudeAdjustmentMethod(IntEnum):
    """High latitude adjustment methods."""
    MIDDLE_OF_THE_NIGHT = 1
    ONE_SEVENTH = 2
    ANGLE_BASED = 3  # Recommended
```

#### 5. Caching System (`cache.py`)

**Responsibility:** Reduce API calls with TTL caching

**Features:**

- In-memory cache (default)
- Optional Redis backend for distributed caching
- TTL (time-to-live) expiration
- Automatic cache key generation

```python
from typing import Protocol
from datetime import datetime, timedelta

class CacheBackend(Protocol):
    """Cache backend protocol."""
    def get(self, key: str) -> Any | None: ...
    def set(self, key: str, value: Any, ttl: int) -> None: ...
    def delete(self, key: str) -> None: ...
    def clear(self) -> None: ...

class InMemoryCache:
    """Simple in-memory cache with TTL."""
    def __init__(self):
        self._cache: dict[str, tuple[datetime, Any]] = {}

    def get(self, key: str) -> Any | None:
        if key in self._cache:
            expiry, value = self._cache[key]
            if datetime.now() < expiry:
                return value
            del self._cache[key]
        return None

    def set(self, key: str, value: Any, ttl: int) -> None:
        expiry = datetime.now() + timedelta(seconds=ttl)
        self._cache[key] = (expiry, value)

class RedisCache:
    """Redis-backed cache for distributed systems."""
    def __init__(self, redis_url: str = "redis://localhost:6379"):
        import redis
        self.redis = redis.from_url(redis_url)
```

#### 6. Retry Logic (`retry.py`)

**Responsibility:** Handle transient network failures

**Strategy:** Exponential backoff with jitter

- 1st retry: 1 second
- 2nd retry: 2 seconds
- 3rd retry: 4 seconds
- Max: 10 seconds

```python
from tenacity import (
    retry,
    stop_after_attempt,
    wait_exponential,
    retry_if_exception_type,
)
import httpx

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=1, max=10),
    retry=retry_if_exception_type((httpx.NetworkError, httpx.TimeoutException)),
    reraise=True,
)
def make_request_with_retry(client: httpx.Client, url: str, params: dict):
    """Make HTTP request with automatic retries."""
    response = client.get(url, params=params)
    response.raise_for_status()
    return response
```

#### 7. Rate Limiting (`ratelimit.py`)

**Responsibility:** Respect API rate limits

**Strategy:** Token bucket algorithm

- 10 requests per second (default)
- Configurable limit
- Thread-safe implementation

```python
from ratelimit import limits, sleep_and_retry

class RateLimiter:
    """Rate limiter using token bucket algorithm."""

    @sleep_and_retry
    @limits(calls=10, period=1)  # 10 calls per second
    def allow_request(self):
        """Check if request is allowed (blocks if rate exceeded)."""
        pass
```

#### 8. CLI Tool (`cli.py`)

**Responsibility:** Command-line interface using Typer + Rich

**Commands:**

```bash
# Get prayer times
aladhan get-times --city "Dubai" --country "UAE"
aladhan get-times --lat 25.2048 --lng 55.2708

# Monthly calendar
aladhan calendar --city "London" --month 12 --year 2025

# Hijri conversions
aladhan convert-hijri "01-01-1447"
aladhan convert-gregorian "01-01-2025"

# Next holiday
aladhan next-holiday

# All holidays in year
aladhan holidays --year 1447
```

**Implementation:**

```python
import typer
from rich.console import Console
from rich.table import Table

app = typer.Typer(help="AlAdhan CLI - Islamic Prayer Times")
console = Console()

@app.command()
def get_times(
    city: str = typer.Option(..., help="City name"),
    country: str = typer.Option(..., help="Country name"),
    method: Method = typer.Option(Method.ISNA, help="Calculation method"),
):
    """Get prayer times for a city."""
    client = AlAdhanClient(method=method)
    times = client.get_times_by_city(city, country)

    # Beautiful table output with Rich
    table = Table(title=f"Prayer Times - {city}, {country}")
    table.add_column("Prayer", style="cyan", no_wrap=True)
    table.add_column("Time", style="green")

    table.add_row("Fajr", times.timings.fajr.strftime("%H:%M"))
    table.add_row("Sunrise", times.timings.sunrise.strftime("%H:%M"))
    table.add_row("Dhuhr", times.timings.dhuhr.strftime("%H:%M"))
    table.add_row("Asr", times.timings.asr.strftime("%H:%M"))
    table.add_row("Maghrib", times.timings.maghrib.strftime("%H:%M"))
    table.add_row("Isha", times.timings.isha.strftime("%H:%M"))

    console.print(table)
```

## Python 3.14 Features Utilized

### PEP 649: Lazy Annotations

**Before (Python <3.14):**

```python
from __future__ import annotations
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from aladhan.models import PrayerTimesResponse

def get_times() -> "PrayerTimesResponse":  # Quotes needed
    ...
```

**After (Python 3.14):**

```python
from aladhan.models import PrayerTimesResponse

def get_times() -> PrayerTimesResponse:  # No quotes, no future import!
    ...
```

**Benefit:** Cleaner code, no circular import workarounds

### PEP 750: Template Strings

**Use Case:** Safe URL construction

```python
from typing import Template

def build_api_url(endpoint: str, params: dict) -> str:
    # Template strings prevent injection attacks
    base = t"https://api.aladhan.com"
    return t"{base}/{endpoint}?{query_string}"
```

### PEP 758: Simpler Exception Syntax

**Before:**

```python
except (ValueError,):  # Parentheses required
    ...
```

**After:**

```python
except ValueError:  # Clean!
    ...
```

### PEP 779: Free-Threaded Mode

**Use Case:** Parallel API requests without GIL

```python
# Run with: PYTHON_GIL=0 python script.py
import concurrent.futures

with concurrent.futures.ThreadPoolExecutor() as executor:
    # True parallelism - no GIL!
    futures = [
        executor.submit(client.get_times_by_city, city, country)
        for city, country in cities
    ]
    results = [f.result() for f in futures]
```

### Experimental JIT

**Benefit:** 5-10% performance improvement for hot code paths
**Usage:** Automatic - no code changes needed

## UV Package Manager Integration

### Project Configuration (`pyproject.toml`)

```toml
[project]
name = "aladhan"
version = "1.0.0"
description = "Modern Python client for AlAdhan.com API - Islamic Prayer Times"
authors = [
    {name = "Your Name", email = "your@email.com"}
]
requires-python = ">=3.14"
dependencies = [
    "httpx>=0.27.0",
    "pydantic>=2.12.0",
    "tenacity>=9.0.0",
    "ratelimit>=2.2.1",
    "typer>=0.15.0",
    "rich>=13.0.0",
]
readme = "README.md"
license = {text = "GPL-3.0"}
keywords = ["prayer", "times", "islamic", "muslim", "hijri", "calendar"]
classifiers = [
    "Development Status :: 4 - Beta",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: GNU General Public License v3 (GPLv3)",
    "Programming Language :: Python :: 3.14",
    "Topic :: Religion",
]

[project.urls]
Homepage = "https://github.com/islamic-network/aladhan-python-client"
Documentation = "https://aladhan-python.readthedocs.io"
Repository = "https://github.com/islamic-network/aladhan-python-client"
Issues = "https://github.com/islamic-network/aladhan-python-client/issues"

[project.optional-dependencies]
redis = ["redis>=5.0.0"]
dev = [
    "pytest>=8.0.0",
    "pytest-asyncio>=0.24.0",
    "pytest-cov>=6.0.0",
    "ruff>=0.8.0",
    "mypy>=1.13.0",
]

[project.scripts]
aladhan = "aladhan.cli:app"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.uv]
dev-dependencies = [
    "pytest>=8.0.0",
    "pytest-asyncio>=0.24.0",
    "ruff>=0.8.0",
    "mypy>=1.13.0",
]

[tool.ruff]
target-version = "py314"
line-length = 100
select = ["E", "F", "I", "N", "W", "UP", "ANN", "B", "C4"]
ignore = ["ANN101", "ANN102"]

[tool.ruff.per-file-ignores]
"tests/*" = ["ANN"]

[tool.mypy]
python_version = "3.14"
strict = true
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = "test_*.py"
python_classes = "Test*"
python_functions = "test_*"
asyncio_mode = "auto"
```

### Development Workflow

```bash
# Install UV (one-time setup)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create project
cd python-client
uv init aladhan

# Install dependencies (creates venv + lock file)
uv sync

# Add new dependency
uv add httpx pydantic

# Add dev dependency
uv add --dev pytest ruff mypy

# Run tests
uv run pytest

# Run with coverage
uv run pytest --cov=aladhan --cov-report=html

# Format code
uv run ruff format .

# Lint code
uv run ruff check .

# Type check
uv run mypy src/

# Run CLI during development
uv run aladhan get-times --city Dubai --country UAE

# Build package
uv build

# Publish to PyPI
uv publish

# Run example scripts (PEP 723)
uv run examples/basic.py
```

### PEP 723 Script Support

**Example Script:**

```python
#!/usr/bin/env -S uv run
# /// script
# requires-python = ">=3.14"
# dependencies = [
#     "aladhan>=1.0.0",
# ]
# ///

"""
Self-contained example - UV handles dependencies automatically!
Usage: uv run examples/basic.py
"""

from aladhan import AlAdhanClient

client = AlAdhanClient()
times = client.get_times_by_city("Mecca", "Saudi Arabia")
print(f"Next Fajr: {times.timings.fajr}")
```

**Benefits:**

- No virtual environment needed
- Dependencies declared inline
- Shareable single-file scripts
- Perfect for documentation examples

## Security Improvements

### Over PHP Client

| Security Issue          | PHP Client                    | Python Client                 |
| ----------------------- | ----------------------------- | ----------------------------- |
| **SSL Verification**    | 🚨 Disabled in Location class | ✅ Always enabled             |
| **Input Validation**    | ❌ Minimal                    | ✅ Pydantic validation        |
| **Type Safety**         | ❌ Arrays only                | ✅ Full type hints            |
| **URL Construction**    | ⚠️ String concatenation       | ✅ Template strings (PEP 750) |
| **Dependency Security** | ⚠️ Manual updates             | ✅ UV lock file + dependabot  |

### Implementation

```python
class AlAdhanClient:
    def __init__(
        self,
        base_url: str = "https://api.aladhan.com",  # HTTPS by default
        verify_ssl: bool = True,  # NEVER disable
        timeout: int = 30,
    ):
        if not verify_ssl:
            raise SecurityError(
                "SSL verification cannot be disabled. "
                "This is a security requirement."
            )

        self.http = httpx.Client(
            base_url=base_url,
            verify=True,  # Always enforced
            timeout=timeout,
            headers={
                "User-Agent": "AlAdhanPythonClient/1.0",
                "Accept": "application/json",
            }
        )
```

## Performance Benchmarks (Projected)

| Operation         | PHP Client      | Python Client   | Improvement             |
| ----------------- | --------------- | --------------- | ----------------------- |
| **Install time**  | ~10s (composer) | ~1s (uv)        | **10x faster**          |
| **First request** | ~200ms          | ~180ms (HTTP/2) | 10% faster              |
| **With caching**  | N/A             | ~0.1ms          | **2000x faster**        |
| **Type checking** | Runtime errors  | Compile-time    | **Catch bugs early**    |
| **Async support** | ❌ No           | ✅ Yes          | **Concurrent requests** |
| **CLI tool**      | ❌ No           | ✅ Yes          | **Better UX**           |

## Testing Strategy

### Test Coverage Goals

- **Unit tests:** 90%+ coverage
- **Integration tests:** All API endpoints
- **Async tests:** All async methods
- **CLI tests:** All commands

### Test Structure

```python
import pytest
from aladhan import AlAdhanClient, AsyncAlAdhanClient

# Sync client tests
def test_get_times_by_city():
    client = AlAdhanClient()
    times = client.get_times_by_city("Dubai", "UAE")

    assert times.timings.fajr is not None
    assert isinstance(times.timings.fajr, time)
    assert times.date.gregorian.year >= 2025

# Async client tests
@pytest.mark.asyncio
async def test_async_get_times():
    async with AsyncAlAdhanClient() as client:
        times = await client.get_times_by_city("Dubai", "UAE")
        assert times is not None

# Caching tests
def test_cache_hit():
    client = AlAdhanClient(cache_ttl=60)

    # First call - cache miss
    times1 = client.get_times_by_city("Dubai", "UAE")

    # Second call - cache hit
    times2 = client.get_times_by_city("Dubai", "UAE")

    assert times1 == times2
    assert client._cache_stats["hits"] == 1

# Retry tests
def test_retry_on_network_error(mocker):
    mocker.patch("httpx.Client.get", side_effect=[
        httpx.NetworkError("Connection failed"),
        httpx.NetworkError("Connection failed"),
        httpx.Response(200, json={"data": {...}}),  # Success on 3rd try
    ])

    client = AlAdhanClient(max_retries=3)
    times = client.get_times_by_city("Dubai", "UAE")
    assert times is not None

# Rate limiting tests
def test_rate_limit():
    client = AlAdhanClient(rate_limit=2)  # 2 requests/second

    start = time.time()
    for _ in range(3):
        client.get_times_by_city("Dubai", "UAE")
    duration = time.time() - start

    # Should take at least 1 second (3 requests at 2/sec)
    assert duration >= 1.0
```

## Migration Guide (from PHP Client)

### For Existing Users

**PHP Client:**

```php
use AlAdhanApi\TimesByCity;

$times = new TimesByCity('Dubai', 'United Arab Emirates');
$result = $times->get();

echo $result['data']['timings']['Fajr'];  // String
```

**Python Client:**

```python
from aladhan import AlAdhanClient

client = AlAdhanClient()
times = client.get_times_by_city('Dubai', 'United Arab Emirates')

print(times.timings.fajr)  # datetime.time object
```

### Key Differences

| Aspect          | PHP Client         | Python Client                 |
| --------------- | ------------------ | ----------------------------- |
| **API Style**   | Class per endpoint | Single client object          |
| **Return Type** | Arrays             | Pydantic models               |
| **Async**       | Not supported      | Full async/await              |
| **Caching**     | Not built-in       | Built-in with TTL             |
| **Retries**     | Not built-in       | Automatic exponential backoff |
| **CLI**         | Not available      | Full CLI tool                 |
| **Type Safety** | Runtime only       | Compile-time + runtime        |

## Development Roadmap

### Phase 1: MVP: s 1-2)

- [x] Project structure setup with UV
- [ ] Core models (Pydantic)
- [ ] Synchronous client
- [ ] Basic tests
- [ ] README documentation

### Phase 2: Production Features: s 3-4)

- [ ] Asynchronous client
- [ ] Caching system (in-memory)
- [ ] Retry logic
- [ ] Rate limiting
- [ ] Comprehensive tests (90% coverage)

### Phase 3: CLI & Polish: 5)

- [ ] CLI tool (Typer + Rich)
- [ ] Redis cache backend
- [ ] API documentation
- [ ] User guide
- [ ] PyPI publication

### Phase 4: Advanced Features (Future)

- [ ] Webhook support
- [ ] GraphQL client (if API adds GraphQL)
- [ ] Data export (CSV, JSON, iCal)
- [ ] Prayer time notifications
- [ ] Integration with calendar apps

### Phase 5: Library Port (Future)

- [ ] Port PHP astronomical calculations to Python
- [ ] NumPy/SciPy integration
- [ ] Offline calculation support
- [ ] Performance optimization
- [ ] Scientific computing features

## Comparison: Python vs PHP Client

### Advantages of Python Client

✅ **Modern Language Features**

- Type hints with Pydantic (compile-time safety)
- Async/await for concurrent requests
- Pattern matching (Python 3.10+)
- Context managers for resource cleanup

✅ **Better Developer Experience**

- Single client object (not class per endpoint)
- IDE autocomplete with type hints
- Cleaner API design
- CLI tool included

✅ **Production-Grade Features**

- Built-in caching (in-memory + Redis)
- Automatic retry logic
- Rate limiting
- Better error messages

✅ **Performance**

- HTTP/2 support (faster)
- Caching (2000x faster for repeated requests)
- UV package manager (10-100x faster installs)
- Python 3.14 JIT (5-10% boost)

✅ **Security**

- SSL always verified (not disabled)
- Input validation via Pydantic
- Template strings for injection prevention
- Modern dependency management

✅ **Testing**

- Higher test coverage goal (90% vs PHP's ~60%)
- Async test support
- Better mocking with pytest
- Coverage reports

### When to Use Each

**Use Python Client When:**

- Building modern web apps (Django, Flask, FastAPI)
- Need async/await for performance
- Building CLI tools
- Data analysis with pandas
- IoT/embedded systems (Raspberry Pi)
- Type safety is important

**Use PHP Client When:**

- Already using PHP (WordPress, Laravel)
- Legacy codebase integration
- Shared hosting without Python
- Team only knows PHP

## Next Steps

1. ✅ **Documentation Complete** - This file
2. **Setup Project Structure** - UV workspace, pyproject.toml
3. **Implement Core Models** - Pydantic data models
4. **Build Sync Client** - Main functionality
5. **Add Tests** - Achieve 90% coverage
6. **Build Async Client** - Async/await support
7. **Add Production Features** - Caching, retries, rate limiting
8. **Create CLI Tool** - Typer-based CLI
9. **Write User Documentation** - README, user guide
10. **Publish to PyPI** - Make it available via `uv add aladhan`

---

## Resources

### Official Documentation

- Python 3.14: <https://docs.python.org/3.14/>
- UV: <https://docs.astral.sh/uv/>
- Pydantic: <https://docs.pydantic.dev/>
- httpx: <https://www.python-httpx.org/>
- Typer: <https://typer.tiangolo.com/>
- Rich: <https://rich.readthedocs.io/>

### API Documentation

- AlAdhan API: <https://aladhan.com/prayer-times-api>
- Calculation Methods: <https://aladhan.com/calculation-methods>

### Related Al-Adhan Projects

- **PHP Prayer Times Library:** <https://1x.ax/islamic-network/libraries/prayer-times>
- **PHP API Client:** <https://1x.ax/islamic-network/aladhan/api-client-php>

---

_Last Updated: 2025-10-26_
_Status: Planning Phase - Ready for Implementation_
