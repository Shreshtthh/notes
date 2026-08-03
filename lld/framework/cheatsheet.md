# Lesson 8

## Cheat Sheet
Every LLD interview follows the same arc: scope, discover entities, design top-down from the orchestrator, implement core logic, then discuss extensibility.

### Key Concepts

| Concept | Definition |
|---|---|
| 5-step framework | Requirements (5 min) -> Entities (3 min) -> Class Design (10-15 min) -> Implementation (10 min) -> Extensibility (5 min) |
| Scoping themes | Four question categories: primary capabilities, rules/completion, error handling, scope boundaries |
| Entity discovery | Extract nouns from requirements, then filter: only keep nouns that maintain changing state or enforce rules |
| Entity demotion | Most candidate nouns become fields or enums on another class, not standalone classes |
| Orchestrator pattern | One top-level coordinator that owns domain objects, exposes the public API, and routes inputs |
| Derive vs track state | Prefer computing values from a single source of truth over maintaining redundant copies |
| Temporal resource modeling | Store bookings as intervals (start, end), not as pre-populated per-slot rows |
| Half-open intervals | `[check_in, check_out)` -- guest checking out on day X does not conflict with guest checking in on day X |
| Architecture as DAG | Class dependency graph must be acyclic; cycles mean a missing interface or event |

### Core Formulas

| Name | Formula |
|---|---|
| Entity filter question | "Does it maintain changing state or enforce rules? If not, it is a field, not a class." |
| Optimal class count | 3-5 core classes for most LLD problems; deep-dive on 2-3 beats shallow overview of 10 |
| Interval overlap test | Two intervals `[a,b)` and `[c,d)` overlap iff `a < d` AND `c < b` |
| Single source of truth | If a value can be computed from existing data, do not store it separately |
| Slot table cost | `O(duration)` storage and writes, bounded time windows, timezone pain -- use intervals instead |
| DAG construction order | Topological sort of the dependency graph gives you valid instantiation order |

### Decision Rules

| Situation | Action |
|---|---|
| Prompt is vague ("Design Uber") | Spend 5 minutes asking scoping questions across the 4 themes before designing |
| Noun extracted from requirements does not track changing state | Demote to a field or enum on another class |
| Need a "starting class" for the design | Start with the orchestrator (`ParkingLot`, `Game`, `BookingSystem`, `Library`) |
| Tracking `availableSeats` set alongside reservations list | Derive availability from reservations -- single source of truth |
| Resource reserved for a time range (hotel room, meeting room) | Store as an interval (start, end); query with overlap test |
| Two classes depend on each other | Introduce an interface or event to break the cycle and restore DAG property |

### Common Pitfalls

| Pitfall | Why It Fails |
|---|---|
| Diving into code without asking clarifying questions | Builds the wrong thing; wastes time on features not in scope |
| Promoting every noun to a class | Leads to 10+ trivial classes with no real behavior; interviewers penalize this |
| Tracking state in two places (e.g., reservations list + `bookedSeats` set) | The two representations drift out of sync, producing impossible states |
| Pre-populating slot tables for temporal resources | `O(duration)` storage, bounded windows, and timezone fragility for no gain in most LLD problems |
| Coding silently for 20 minutes | Communication is a scored dimension; announce each step and explain tradeoffs |
| Skipping extensibility discussion | Misses the chance to show the design bends without breaking for new requirements |
