# Feature Specification: AlAdhan Python Client Library

**Feature Branch**: `001-python-client`
**Created**: 2025-10-27
**Status**: Draft
**Input**: User description: "Modern, type-safe Python client for the AlAdhan Prayer Times API with 100% API coverage, built-in caching, and separate sync/async clients"

## User Scenarios & Testing _(mandatory)_

### User Story 1 - Get Prayer Times for a Location (Priority: P1)

A Muslim developer building a mobile app needs to retrieve accurate prayer times for their user's location so the app can notify users when it's time to pray.

**Why this priority**: Core value proposition - prayer times retrieval is the fundamental capability. Without this, the library has no purpose. This single feature delivers immediate value to developers.

**Independent Test**: Can be fully tested by providing coordinates and a date, then verifying prayer times are returned with correct values for all 5 daily prayers. Delivers a working MVP that solves the primary use case.

**Acceptance Scenarios**:

1. **Given** developer has user's coordinates (latitude, longitude), **When** they request prayer times for today, **Then** they receive accurate times for Fajr, Dhuhr, Asr, Maghrib, and Isha
2. **Given** developer specifies a calculation method (e.g., ISNA, MWL), **When** they request prayer times, **Then** times are calculated using that method's parameters
3. **Given** developer requests prayer times for a specific past or future date, **When** they provide a valid date, **Then** they receive prayer times for that exact date
4. **Given** developer requests prayer times, **When** request completes successfully, **Then** response includes Hijri calendar information for that date
5. **Given** developer makes identical prayer time requests repeatedly, **When** subsequent requests occur, **Then** responses are delivered in under 1 millisecond (from cache)

---

### User Story 2 - Get Prayer Times Without Coordinates (Priority: P2)

A developer whose users don't want to share precise GPS coordinates needs alternative ways to get prayer times using city names or addresses so their app respects user privacy.

**Why this priority**: Critical for user privacy and ease of use. Many users prefer not to share precise location data. City/address lookup is more user-friendly than asking for coordinates.

**Independent Test**: Can be tested by providing city/country or a street address, then verifying prayer times are returned accurately. Works independently of P1 (uses same underlying data model).

**Acceptance Scenarios**:

1. **Given** developer has user's city and country, **When** they request prayer times by city, **Then** they receive accurate prayer times for that location
2. **Given** developer has a street address, **When** they request prayer times by address, **Then** address is geocoded and prayer times are returned
3. **Given** city name is ambiguous (e.g., "London" exists in UK and Canada), **When** developer provides country, **Then** correct location is used
4. **Given** address geocoding fails, **When** request is made, **Then** clear error message explains the issue (invalid address, service unavailable)

---

### User Story 3 - Get Monthly/Yearly Prayer Calendar (Priority: P2)

A developer building a calendar feature needs to retrieve prayer times for an entire month or year at once so they can pre-populate a UI calendar without making 30+ separate API calls.

**Why this priority**: Essential for calendar UIs and batch operations. Prevents rate limit violations and improves performance for calendar views. Common use case for mosque apps and Islamic calendar applications.

**Independent Test**: Can be tested by requesting a month/year of prayer times, then verifying all days are returned with correct data. Delivers value for calendar features independently of single-day lookups.

**Acceptance Scenarios**:

1. **Given** developer needs a full month's prayer times, **When** they request calendar for October 2025, **Then** they receive prayer times for all 31 days
2. **Given** developer needs a full year's prayer times, **When** they request annual calendar for 2025, **Then** they receive prayer times for all 12 months (365 days)
3. **Given** developer requests monthly calendar, **When** response is received, **Then** each day includes Hijri date, prayer times, and metadata
4. **Given** developer makes repeated calendar requests for same month, **When** cached data exists, **Then** subsequent requests complete in under 1 millisecond

---

### User Story 4 - Hijri Calendar Conversions and Islamic Dates (Priority: P3)

A developer building an Islamic calendar application needs to convert between Gregorian and Hijri dates, and identify Islamic holidays, so users can see important Islamic dates and plan accordingly.

**Why this priority**: Important for Islamic apps but not essential for basic prayer times. Enhances user experience for calendar features and cultural/religious planning.

**Independent Test**: Can be tested by converting dates between calendars and requesting Islamic holidays, verifying accuracy against known dates. Works independently as a utility feature.

**Acceptance Scenarios**:

1. **Given** developer has a Gregorian date, **When** they convert to Hijri, **Then** they receive accurate Hijri date with month name in Arabic and English
2. **Given** developer has a Hijri date, **When** they convert to Gregorian, **Then** they receive accurate Gregorian date
3. **Given** developer requests Islamic holidays for a year, **When** data is retrieved, **Then** all major holidays (Ramadan, Eid, etc.) are included with dates
4. **Given** developer requests current Islamic date, **When** request is made, **Then** they receive today's Hijri date

---

### User Story 5 - Qibla Direction Calculation (Priority: P3)

A developer building a compass feature needs to calculate the Qibla direction from any location so their users know which direction to face for prayer.

**Why this priority**: Valuable utility feature but not core to prayer times. Common requirement for Islamic apps but serves a different use case.

**Independent Test**: Can be tested by providing coordinates and verifying Qibla direction angle is returned accurately. Completely independent feature from prayer times.

**Acceptance Scenarios**:

1. **Given** developer has user's coordinates, **When** they request Qibla direction, **Then** they receive bearing angle in degrees from North
2. **Given** developer is in North America (west of Mecca), **When** Qibla is calculated, **Then** direction is northeast (accurate to ±1 degree)
3. **Given** developer is in Southeast Asia (east of Mecca), **When** Qibla is calculated, **Then** direction is northwest (accurate to ±1 degree)

---

### User Story 6 - Access 99 Names of Allah (Priority: P4)

A developer building an Islamic educational app needs access to the 99 Names of Allah (Asma al-Husna) with translations and meanings so users can learn and reflect on these names.

**Why this priority**: Educational/spiritual feature. Nice to have but not related to prayer times core functionality. Lower priority than operational features.

**Independent Test**: Can be tested by requesting the names and verifying all 99 are returned with accurate Arabic, transliteration, and English meaning.

**Acceptance Scenarios**:

1. **Given** developer requests all 99 names, **When** data is retrieved, **Then** all names are returned with Arabic text, transliteration, and English meaning
2. **Given** developer requests a specific name by number (1-99), **When** request is made, **Then** that specific name's details are returned
3. **Given** names are requested multiple times, **When** cached data exists, **Then** subsequent requests complete instantly from cache

---

### Edge Cases

- **What happens when coordinates are invalid** (latitude > 90, longitude > 180)? Clear validation error with helpful message
- **What happens when date is far in the future** (year 3000)? Accept and calculate (prayer times are deterministic)
- **What happens when API rate limit is exceeded** (12 req/s)? Automatic caching prevents this; if exceeded, wait and retry with exponential backoff
- **What happens when network is unavailable**? Return cached data if available; otherwise clear error about network issue
- **What happens when city name doesn't exist**? Clear error message indicating city not found, suggest checking spelling
- **What happens when calculation method is invalid**? Clear error listing valid calculation method names
- **How does system handle extreme latitudes** (near poles where sun doesn't set)? Use high-latitude adjustment methods as specified by calculation method
- **What happens when requesting Hijri date before Islamic calendar era**? Clear error indicating date is before Hijri calendar start (622 CE)

## Requirements _(mandatory)_

### Functional Requirements

**Core Prayer Times:**

- **FR-001**: System MUST retrieve prayer times for any valid geographic coordinates (latitude/longitude)
- **FR-002**: System MUST support all 24 documented AlAdhan API calculation methods (MWL, ISNA, EGYPT, MAKKAH, etc.)
- **FR-003**: System MUST allow selection of jurisprudential school (Shafi/Standard or Hanafi) for Asr calculation
- **FR-004**: System MUST support prayer time adjustments/tuning (add/subtract minutes per prayer)
- **FR-005**: System MUST return all prayer times: Fajr, Sunrise, Dhuhr, Asr, Maghrib, Sunset, Isha, Imsak, Midnight, First Third, Last Third
- **FR-006**: System MUST retrieve prayer times by city and country name
- **FR-007**: System MUST retrieve prayer times by street address (with automatic geocoding)
- **FR-008**: System MUST provide monthly calendar of prayer times (all days in a month)
- **FR-009**: System MUST provide yearly calendar of prayer times (all days in a year)

**Hijri Calendar:**

- **FR-010**: System MUST convert Gregorian dates to Hijri dates
- **FR-011**: System MUST convert Hijri dates to Gregorian dates
- **FR-012**: System MUST provide Islamic holidays for any Hijri year
- **FR-013**: System MUST provide current Hijri date
- **FR-014**: System MUST identify the next upcoming Islamic holiday

**Qibla & Utilities:**

- **FR-015**: System MUST calculate Qibla direction for any coordinates
- **FR-016**: System MUST provide information about all available calculation methods
- **FR-017**: System MUST provide access to the 99 Names of Allah with Arabic, transliteration, and English meaning
- **FR-018**: System MUST provide current timestamp for any timezone

**Type Safety & Validation:**

- **FR-019**: System MUST validate all API responses against defined data models before returning to users
- **FR-020**: System MUST provide type-safe interfaces (compile-time and runtime type checking)
- **FR-021**: System MUST provide IDE autocomplete for all API methods and response fields
- **FR-022**: System MUST reject invalid inputs (bad coordinates, invalid dates, unknown methods) with clear error messages

**Performance & Caching:**

- **FR-023**: System MUST cache deterministic results (prayer times for specific date/location/method never change)
- **FR-024**: System MUST respect HTTP caching headers from API (Cache-Control, ETag)
- **FR-025**: System MUST support conditional requests using ETags to minimize bandwidth
- **FR-026**: System MUST never exceed API rate limits (12 requests/second)
- **FR-027**: System MUST provide cache bypass option for users who need fresh API calls
- **FR-028**: System MUST support multiple cache backends (in-memory, persistent storage, distributed cache)

**Client Separation:**

- **FR-029**: System MUST provide separate synchronous client for blocking operations
- **FR-030**: System MUST provide separate asynchronous client for non-blocking operations
- **FR-031**: System MUST ensure synchronous and asynchronous clients have identical API surface (same methods, parameters)
- **FR-032**: System MUST have unambiguous type signatures (no dual-mode Union types)

**Testing & Quality:**

- **FR-033**: System MUST have automated tests for all API endpoints
- **FR-034**: System MUST verify behavior against actual API responses, not mocks
- **FR-035**: System MUST maintain test suite that executes in under 5 seconds for rapid feedback

### Key Entities

- **Prayer Times**: Collection of prayer times for a specific day (Fajr, Dhuhr, Asr, Maghrib, Isha, plus additional times)
- **Calculation Method**: Algorithm parameters for determining prayer times (fajr angle, isha angle, maghrib adjustment, etc.)
- **Hijri Date**: Islamic calendar date with day, month, year, weekday names in Arabic and English
- **Gregorian Date**: Standard calendar date with day, month, year, weekday
- **Location Coordinates**: Latitude and longitude representing a geographic position
- **Qibla Direction**: Bearing angle in degrees from North pointing toward the Kaaba in Mecca
- **Islamic Holiday**: Named religious observance with Hijri and Gregorian dates
- **Allah's Name**: One of the 99 Names with Arabic text, transliteration, and English meaning/translation
- **Cache Entry**: Stored API response with key (request parameters) and metadata (timestamp, ETag, expiration)
- **Method Information**: Details about a calculation method including ID, name, location, and calculation parameters

## Success Criteria _(mandatory)_

### Measurable Outcomes

**Performance:**

- **SC-001**: Cached prayer time requests MUST complete in under 1 millisecond
- **SC-002**: Uncached prayer time requests MUST complete in under 500 milliseconds at p95
- **SC-003**: Batch calendar requests (30 days) MUST complete faster than 30 individual requests (via caching)
- **SC-004**: System MUST maintain 95%+ cache hit rate for typical usage patterns

**Correctness:**

- **SC-005**: Prayer time calculations MUST match AlAdhan API reference data with 100% accuracy
- **SC-006**: Hijri calendar conversions MUST match Islamic astronomical calculations within ±1 day
- **SC-007**: Qibla direction MUST be accurate within ±1 degree for all tested locations

**Developer Experience:**

- **SC-008**: Developers can retrieve prayer times in under 5 minutes from library installation to first working code
- **SC-009**: IDE autocomplete provides accurate suggestions for 100% of public API methods and response fields
- **SC-010**: 90% of developers successfully complete primary use case (get prayer times) on first attempt without reading docs
- **SC-011**: Type checker catches 100% of parameter/response type mismatches at compile time

**Reliability:**

- **SC-012**: System handles 1,000 consecutive API requests without rate limit violations (via caching and throttling)
- **SC-013**: System gracefully handles network failures with clear error messages (no cryptic stack traces)
- **SC-014**: 100% of API endpoints are covered by the client library (no gaps requiring raw HTTP calls)

**Quality & Maintainability:**

- **SC-015**: Test suite maintains 80%+ code coverage
- **SC-016**: Test suite completes in under 5 seconds for continuous feedback during development
- **SC-017**: 90%+ of tests remain green when internal implementation changes (resistance to refactoring)
- **SC-018**: Zero mocks used for in-process components (only external API calls are doubled)

**Adoption:**

- **SC-019**: Library becomes the most feature-complete Python client for AlAdhan API (100% endpoint coverage vs competitors' 20-40%)
- **SC-020**: Library is the only Python client with full type safety and validation
- **SC-021**: Library provides fastest cached response times among all Python prayer time libraries

## Assumptions

1. **API Stability**: AlAdhan API contract (endpoints, parameters, response formats) remains stable as documented in validated OpenAPI schema
2. **Network Availability**: Users have internet connectivity for initial API calls (caching enables offline operation afterward)
3. **Rate Limits**: AlAdhan API maintains current rate limit of 12 requests/second
4. **Calculation Accuracy**: AlAdhan API's astronomical calculations are authoritative and accurate
5. **Caching Validity**: Prayer times for a specific date/location/method are deterministic and safe to cache indefinitely
6. **Target Users**: Primary users are Python developers building Islamic applications (mobile apps, web services, IoT devices)
7. **User Environment**: Users have modern development environments capable of running latest Python versions
8. **Default Behaviors**: When not specified, use industry-standard Islamic defaults (Shafi school for Asr, Standard midnight mode)
9. **Error Handling**: Network failures should return cached data when available; otherwise fail with clear error messages
10. **Documentation Language**: English is acceptable for API documentation and error messages (Arabic support is for data, not UI)

## Dependencies

**External Systems:**

- AlAdhan API (<https://api.aladhan.com/v1>)
- Network connectivity for initial API calls
- System clock for determining "current" date/time requests

**Existing Assets:**

- Validated OpenAPI 3.1.0 schema (`openapi.yaml`) documenting all 30+ endpoints
- Competitive research identifying market gaps and opportunities
- HTTP header analysis confirming caching behavior and rate limits

**Future Enhancements** (explicitly out of scope for this feature):

- Offline-capable astronomical calculation library (local computation without API)
- CLI tool for command-line prayer time queries
- Configuration file support for default settings
- Background job scheduling for notifications

## Scope Boundaries

**In Scope:**

- Python client library (importable package)
- Synchronous and asynchronous API clients
- All 30+ AlAdhan API endpoints
- Type-safe data models with validation
- HTTP-aware caching with multiple backends
- Comprehensive test suite
- Developer documentation and examples

**Out of Scope:**

- Command-line interface (CLI tool) - future enhancement
- Desktop/mobile UI applications - not a library responsibility
- Prayer time calculations without API (offline mode) - future library
- Notifications/alarms - application-level concern
- User authentication/accounts - library is anonymous API wrapper
- Database storage - users implement their own persistence
- Real-time updates/websockets - API is request-response only
- Multi-language support for error messages - English only for now
