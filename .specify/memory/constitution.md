# SDD Constitution

## Core Principles

### I. Library-First Approach
Every feature MUST be implemented as a standalone library first. Libraries are the primary unit of delivery:

- All functionality begins as an independently deployable library
- Libraries must be self-contained with no hidden organizational dependencies
- Each library must be independently testable and documented
- Clear, narrowly-scoped purpose required—no organizational-only or utility libraries without defined contracts
- Library boundaries define system architecture

### II. Test-Driven Development (NON-NEGOTIABLE)
Test-Driven Development is mandatory for all development:

- Red-Green-Refactor cycle strictly enforced: tests written first → implementation follows → refactoring improves code quality
- User approval of test cases (specification) required before implementation begins
- Tests must comprehensively cover happy path, edge cases, and error conditions
- Library contracts verified through test suites as the definition of expected behavior
- No implementation commits without corresponding test coverage

### III. OOP and SOLID Principles
All code strictly adheres to Object-Oriented Programming and SOLID design principles:

- **Single Responsibility**: Each class/module has one reason to change
- **Open/Closed**: Open for extension, closed for modification
- **Liskov Substitution**: Derived types must be substitutable for base types
- **Interface Segregation**: Clients depend on specific interfaces, not broad contracts
- **Dependency Inversion**: Depend on abstractions, not concrete implementations
- Composition over inheritance; favor polymorphism through interfaces
- Clear separation of concerns across library modules and public contracts

## Development Workflow

### Library Creation & Testing Gate
1. Define library scope and public API contract
2. Write comprehensive test suite reflecting the contract
3. Get stakeholder approval of tests
4. Implement library to pass all tests
5. Review code for SOLID compliance before merge
6. Document public API and usage patterns

### Code Review Requirements
- Every PR must verify TDD adherence: tests written before implementation
- Library boundaries must be clear; no organizational-only code allowed
- SOLID principle violations trigger mandatory revision
- Library contracts must be backwards-compatible or version-bumped appropriately

## Governance

The Constitution supersedes all other practices and policies. All pull requests and code reviews MUST verify compliance with these three core principles:

- **Library-First**: Is this feature designed as a reusable library?
- **TDD**: Were tests written first and do they define the contract?
- **SOLID/OOP**: Does the code follow SOLID principles and proper OOP structure?

Amendments to this constitution require documentation of rationale, impact on existing libraries, and migration plan for non-compliant code. Complexity trade-offs MUST be justified against these principles.

**Version**: 1.0.0 | **Ratified**: 2026-05-18 | **Last Amended**: 2026-05-18
