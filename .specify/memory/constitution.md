# AlAdhan Python Client Constitution

## SYNC IMPACT REPORT

Version Change: N/A → 1.0.0 (Initial Constitution)
Rationale: Initial constitution focusing on principles and governance,
not implementation details

Modified Principles: N/A (initial version)
Added Sections:

- Core Principles (I-V) - Timeless, technology-agnostic
- Foundational Constraints - Project identity constraints
- Quality Standards - Abstract quality requirements
- Governance - Amendment and compliance processes
- Testing Philosophy Foundations - Intellectual lineage

Templates Status:
✅ plan-template.md - Compatible (principles-focused)
✅ spec-template.md - Compatible (behavior-driven)
✅ tasks-template.md - Compatible (test-first aligned)
⚠ No command files exist yet to validate

Follow-up TODOs:

- Create technical specification document with implementation details
- Define technology stack in separate standards document

## Preamble

This constitution establishes the **non-negotiable principles** and **governance framework** for the AlAdhan Python Client project. It defines WHAT we value and WHY, not HOW to implement it.

Technical specifications, tool choices, and implementation details are documented separately and may evolve. This constitution governs those decisions but does not prescribe them.

## Core Principles

### I. Type Safety (NON-NEGOTIABLE)

**Principle**: All data crossing system boundaries MUST be validated and type-safe.

**Intent**: Prevent runtime errors, enable compile-time verification, and provide excellent developer experience through IDE support and autocomplete.

**Requirements:**

- External data (API responses, user input) MUST be validated at entry points
- Type annotations MUST be comprehensive and accurate
- Type checker MUST pass in strict mode with zero errors
- Runtime validation MUST occur for all external data

**Anti-Patterns (FORBIDDEN):**

- ❌ Unvalidated dictionaries or dynamic typing for external data
- ❌ Type hints ignored by type checkers (`# type: ignore` without justification)
- ❌ Runtime type errors due to missing validation

**Rationale**: Competitor analysis revealed that weak typing leads to runtime errors, poor IDE support, and maintenance nightmares. Type safety is our primary competitive advantage.

### II. Behavior-Driven Testing (NON-NEGOTIABLE)

**Principle**: Tests MUST verify observable behavior that users care about, not implementation details.

**Intent**: Create a test suite that protects against regressions while remaining resistant to refactoring. Maximize test ROI through high-value, low-maintenance tests.

**The Test Value Equation:**

Every test MUST maximize: `value = p × r × f × m`

Where:

- `p` = Protection against regressions (catches real bugs)
- `r` = Resistance to refactoring (doesn't break when internals change)
- `f` = Fast feedback (executes quickly)
- `m` = Maintainability (simple, clear, easy to understand)

**Requirements:**

- **Test-First Development**: Tests written before implementation (Red-Green-Refactor)
- **Behavior Focus**: Tests verify WHAT the system does, not HOW it does it
- **Classicist/Sociable Style**: Test components with their real collaborators
- **Fast Feedback**: Tests integrated into development workflow, execute in seconds
- **Minimum Coverage**: 80%+ (but coverage alone is meaningless)

**Anti-Patterns (FORBIDDEN):**

- ❌ Mock-heavy tests coupling to implementation details
- ❌ Tests breaking during refactoring when behavior is unchanged
- ❌ Solitary/Mockist tests isolating every collaborator
- ❌ Treating code coverage as primary quality metric
- ❌ Slow test suites breaking developer flow

**Correct Patterns (REQUIRED):**

- ✅ Test observable behavior through real component collaboration
- ✅ Use test doubles ONLY for "awkward" collaborators (network, filesystem, external APIs)
- ✅ Focus on user-facing contracts, not internal class structure
- ✅ Design architecture to minimize mocking needs
- ✅ Prefer integration tests over mock-heavy unit tests
- ✅ Run tests continuously during development

**Rationale**: Prayer time calculations affect millions of Muslims daily. Wrong times can lead to invalid prayers. Mock-heavy solitary tests create brittle suites that break during refactoring, slow development velocity, and erode trust. Behavior-centric, sociable tests provide superior ROI: higher confidence, faster feedback, dramatically lower maintenance burden.

### III. Clear Separation of Concerns (NON-NEGOTIABLE)

**Principle**: Distinct concerns MUST be implemented as separate, independently understandable components.

**Intent**: Enable independent testing, clear reasoning, and type-safe interfaces.

**Application to This Project:**

- Synchronous and asynchronous operations are separate concerns
- They MUST be implemented as separate components with clear type signatures
- No dual-mode patterns that create type ambiguity

**Requirements:**

- Components serving different concerns MUST have distinct implementations
- Type signatures MUST be unambiguous (no `Union[T, Awaitable[T]]`)
- Each component MUST be independently testable

**Rationale**: Dual-mode patterns create type confusion where static analysis tools cannot provide guarantees. Separate components provide crystal-clear types, better IDE support, and eliminate entire classes of bugs.

### IV. Completeness (NON-NEGOTIABLE)

**Principle**: This client MUST provide complete access to all AlAdhan API capabilities.

**Intent**: Users should never need to make raw HTTP requests or use multiple libraries to access full API functionality.

**Requirements:**

- All documented API endpoints MUST be implemented
- All endpoint parameters MUST be supported
- All response fields MUST be modeled

**Rationale**: Partial implementations force users to either make raw HTTP requests (losing type safety), use multiple libraries (dependency hell), or fork and extend (maintenance burden). Complete coverage is our competitive differentiator.

### V. Performance as a Design Constraint (NON-NEGOTIABLE)

**Principle**: Performance MUST be measured, not assumed. Optimization decisions MUST be data-driven.

**Intent**: Deliver fast, efficient software through systematic measurement and benchmarking, not premature optimization.

**Requirements:**

- **Critical Paths**: Identify and benchmark performance-sensitive operations
- **Deterministic Results**: Cache results that cannot change (prayer times are immutable)
- **Data-Driven Optimization**: When choosing implementations, measure with benchmarks
- **Performance Testing**: Track performance regressions in CI pipeline
- **Rate Limit Compliance**: Never exceed external API rate limits

**HTTP Caching Principle:**

- Results that are deterministic MUST be cached
- HTTP semantic caching MUST respect server directives (Cache-Control, ETag)
- Cache implementation MUST be configurable (in-memory, persistent, distributed)

**Rationale**: Prayer times for a specific date and location are deterministic and will never change. Repeated API calls waste bandwidth, slow responses, and risk rate limit violations. Performance-Driven Development (PDD) ensures optimization decisions are based on measurements, not assumptions.

## Foundational Constraints

These constraints define the project's identity. Changing them would mean building a different project.

### Language

**Constraint**: This is a Python client library.

**Implication**: All implementation code MUST be Python. The client targets the Python ecosystem.

### Modern Language Features

**Constraint**: This project targets cutting-edge Python features, not legacy compatibility.

**Intent**: Position as the modern, technically advanced option for early adopters and new projects.

**Implication**: Use latest language features. Backward compatibility with older versions is explicitly NOT a goal.

### Trunk-Based Development

**Constraint**: Development MUST use trunk-based workflow with small, atomic commits.

**Intent**: Enable fast iteration, independent code review, and clear history.

**Implication**:

- Small, focused commits that build on each other
- Each commit is independently reviewable and revertible
- Direct commits to long-lived feature branches are forbidden

## Quality Standards

### Test Suite Quality

- **Resistance to Refactoring**: >90% of tests MUST remain green when internal implementation changes without behavior changes
- **Mock Usage**: Zero mocks for in-process collaborators (only for external dependencies)
- **Execution Speed**: Core compile suite MUST execute in <5 seconds
- **Coverage**: Minimum 80% (necessary but not sufficient for quality)

### Code Quality

- **Type Safety**: Zero type errors in strict mode
- **Linting**: Zero warnings from configured linters
- **Documentation**: Public APIs MUST have docstrings with usage examples
- **Commit Quality**: Each commit MUST be atomic, focused, and independently functional

### Performance Standards

- **Cached Responses**: <1ms for in-memory cache hits
- **Network Requests**: <500ms p95 for cache misses (including network)
- **Rate Limit Compliance**: MUST never exceed API rate limits
- **Validation Overhead**: Minimal performance cost for type validation

## Governance

### Constitutional Authority

This constitution SUPERSEDES all other development practices, coding guidelines, or personal preferences. When conflicts arise, constitutional principles win.

### Amendment Process

**PATCH Version** (Clarifications):

- Wording improvements, typo fixes, formatting
- No semantic changes to principles
- **Required**: Document in commit message

**MINOR Version** (Expansions):

- New principles added
- Expanded guidance on existing principles
- **Required**: Update dependent templates, sync impact report

**MAJOR Version** (Breaking Changes):

- Principle removal or redefinition
- Backward-incompatible governance changes
- **Required**: Explicit justification, migration plan, community discussion

### Compliance Verification

**Pre-Commit Requirements:**

- All tests MUST pass
- Type checking MUST pass (strict mode, zero errors)
- Linting MUST pass (zero warnings)
- Code MUST be formatted per project standards

**Pre-PR Requirements:**

- Constitution principles MUST be verified (no forbidden patterns)
- All commits MUST follow trunk-based workflow
- Performance benchmarks MUST not show regressions
- Documentation MUST be updated for public API changes

**Continuous Review:**

- Monthly: Dependency updates and security audit
- Quarterly: Performance benchmark review
- Annually: Constitution review for relevance

### Principle Violation Justification

If extraordinary circumstances require a principle violation:

1. **Document** in implementation plan's "Complexity Tracking" section
2. **Explain** why the violation is necessary
3. **Justify** why simpler, compliant alternatives were rejected
4. **Get approval** before implementation
5. **Create TODO** for future refactoring to compliance

**Example:**

```markdown
| Violation             | Why Needed                                        | Simpler Alternative Rejected Because                                                            |
| --------------------- | ------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Use mock for database | Current architecture couples DB to business logic | Refactoring entire data layer would delay critical bugfix by 2 weeks; scheduled for Q2 refactor |
```

## Testing Philosophy: Foundations

Our testing principles synthesize insights from industry thought leaders:

**Kent Beck** (Test-Driven Development, 1999):

- Red-Green-Refactor cycle as design discipline
- Tests as executable specifications for behavior
- Original TDD used sociable/classicist tests, not extreme isolation

**Martin Fowler** (Test Pyramid, Sociable vs Solitary):

- Test Pyramid: economic model balancing speed and confidence
- Sociable unit tests: verify components with real collaborators
- Solitary unit tests: isolate with mocks (leads to brittleness)
- **We adopt the classicist/sociable approach**

**Vladimir Khorikov** (Unit Testing Principles, 2020):

- Test value equation: `v = p × r × f × m`
- Resistance to refactoring (r) as critical attribute
- Mocking as primary cause of brittle tests
- Behavior-over-implementation testing

**ThePrimeagen** (Pragmatic Performance Engineering):

- Developer velocity and flow as first principles
- "Mocks are a code smell" - architectural solution, not testing trick
- Fast feedback loops integrated into editor workflow
- Performance-Driven Development (PDD)
- Coverage is a "vanity metric" without quality assertions

**Our Synthesis:**

We reject dogmatic TDD (mockist style) that creates brittle, slow test suites. We embrace pragmatic TDD (classicist style) that maximizes test ROI through behavior-centric verification, real collaborator integration, fast feedback, and resistance to refactoring.

Every testing decision is evaluated through Khorikov's value equation and ThePrimeagen's velocity lens.

## Relationship to Other Documents

This constitution establishes principles. Implementation details are documented in:

- **Technical Specifications**: Technology stack, library versions, architectural patterns
- **CLAUDE.md**: Architecture decisions, competitive analysis, development context
- **README.md**: User-facing documentation, API examples, installation
- **plan.md**: Feature-specific implementation plans
- **spec.md**: Feature requirements and acceptance criteria
- **tasks.md**: Task breakdowns for implementation

All documents MUST comply with this constitution. In case of conflict, the constitution prevails.

---

**Version**: 1.0.0 | **Ratified**: 2025-10-27 | **Last Amended**: 2025-10-27
