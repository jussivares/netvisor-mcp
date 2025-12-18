# PROCESS: Testing Strategy

> **Version:** 1.5
> **Updated:** 2025-12-16  
> **File:** PROCESS_Testing.md  
> **Purpose:** Practical guide for implementing tests in our development model  
> **Based on:** RESEARCH_Testing_Strategy.md, Session #13 methodology updates, Obra Superpowers

---

## Changes v1.0 → v1.1

| Change | Description | Source |
|--------|-------------|--------|
| ✅ **Audit Trail Hierarchy** | REQ → AC → Task → TS → test_xxx() | Session #13 |
| ✅ **TS-XX.Y naming convention** | Aligned with PROCESS_SPEC_Writing v1.7 | E8, E9 |
| ✅ **Lightweight scenario format** | Simplified format in TECH_SPEC | Session #13 |
| ✅ **Traceability in docstrings** | REQ + AC + TS references in test code | E10 |
| ✅ **TECH_SPEC Phase added** | New phase between SPEC and CODE | Session #13 |

---

## Overview

This document defines HOW testing is implemented across our 11-phase development model.

```
KEY PRINCIPLE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   SPEC Phase      →  Define WHAT to test (REQ + AC)            │
│   TECH_SPEC Phase →  Define scenarios (Task + TS)              │
│   CODE Phase      →  Execute HOW to test (TDD, pytest)         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Audit Trail Hierarchy (v1.1) ⭐ NEW

### From Requirement to Test

```
REQ-01 (Requirement)           ← SPEC: What the system must do
  │
  ├── AC-01 (Acceptance Criterion)  ← SPEC: How we know it works
  │     │
  │     └── Task-01            ← TECH_SPEC: Technical work item
  │           │
  │           ├── TS-01.1 (Happy Path)    ← TECH_SPEC: Test scenario
  │           ├── TS-01.2 (Edge Case)
  │           └── TS-01.3 (Error Case)
  │                 │
  │                 └── test_xxx()  ← CODE: Actual test implementation
```

### Naming Convention

| Element | Format | Example |
|---------|--------|---------|
| Requirement | REQ-XX | REQ-01, REQ-02 |
| Acceptance Criterion | AC-XX | AC-01, AC-02 |
| Task | Task-XX | Task-01, Task-02 |
| Test Scenario | TS-XX.Y | TS-01.1, TS-01.2 |

Where:
- XX = sequential number
- Y = scenario number within that AC

---

## 1. Test Types and Distribution

### 1.1 Test Pyramid/Trophy Hybrid

| Test Type | Target % | Purpose | Tools |
|-----------|----------|---------|-------|
| **Unit** | 40% | Individual functions/methods | pytest |
| **Integration** | 45% | Module interactions, API contracts | pytest + fixtures |
| **E2E** | 10% | Complete user workflows | pytest + mocks |
| **Static** | 5% | Type checking, linting | mypy, ruff |

### 1.2 When to Write Each Type

| Test Type | Written In | Triggered By |
|-----------|------------|--------------|
| Unit | CODE phase | Each function implementation |
| Integration | CODE phase | Module completion |
| E2E | CODE phase | Feature completion |
| Static | Continuous | Every save/commit |

---

## 2. SPEC Phase: Defining Acceptance Criteria

### 2.1 AC in SPEC Template

In SPEC_XX.md, define Acceptance Criteria for each requirement:

```markdown
### REQ-01: Memory Storage 🟢

Users can store memories with categories.

**Acceptance Criteria:**

| AC-ID | Criterion | Type |
|-------|-----------|------|
| AC-01 | Store returns MemoryItem with unique ID | Functional |
| AC-02 | File is created on disk | Functional |
| AC-03 | Invalid input raises clear error | Error |
```

### 2.2 Checklist for SPEC Acceptance Criteria

Before marking SPEC complete:

- [ ] All requirements have at least one AC
- [ ] Each AC is testable (clear pass/fail)
- [ ] Error cases are covered
- [ ] Performance criteria defined (if applicable)

---

## 3. TECH_SPEC Phase: Defining Test Scenarios (v1.1) ⭐ NEW

### 3.1 Lightweight Scenario Format

In TECH_SPEC_XX.md, expand each AC into Test Scenarios using the **lightweight format**:

```markdown
### Task-01: Implement store() method

**Implements:** AC-01, AC-02
**Estimated:** 3h

**Test Scenarios:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-01.1 | HP | Valid MemoryItem → returns item with UUID |
| TS-01.2 | HP | Item with metadata → metadata preserved |
| TS-01.3 | EC | Empty content → stores successfully |
| TS-01.4 | ER | None input → raises ValueError |

*Claude Code may add technical edge cases during implementation.*
```

#### Basic format (when action is obvious from context)

One arrow: `input → output`

```markdown
| TS-01.1 | HP | Valid MemoryItem → returns UUID |
```

#### Extended format (when action needs to be explicit)

Two arrows: `precondition → action → expected result`

```markdown
| TS-02.1 | HP | Empty database → search('test') → returns empty list |
| TS-02.2 | EC | 1000 items in DB → search('x') → returns in <100ms |
```

**Selection guide:** If the reader cannot infer the action from context, use two arrows.

### 3.2 Scenario Types

| Type | Code | Description |
|------|------|-------------|
| Happy Path | HP | Normal, expected behavior |
| Edge Case | EC | Boundary conditions, special situations |
| Error Case | ER | Error conditions, exceptions |
| Performance | PF | Performance tests (if applicable) |

### 3.3 Example: MemoryService Test Scenarios

```markdown
## Test Scenarios

### Feature: Memory Storage

#### Happy Path
| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-01.1 | HP | Valid MemoryItem → returns item with UUID |
| TS-01.2 | HP | Existing memory → get_by_id() returns correct item |
| TS-01.3 | HP | Search term exists → search() returns matching items |

#### Edge Cases
| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-01.4 | EC | Empty content → stores successfully |
| TS-01.5 | EC | Very long content (1MB) → chunks and stores |
| TS-01.6 | EC | Unicode content → handles correctly |

#### Error Cases
| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-01.7 | ER | None as input → raises ValueError |
| TS-01.8 | ER | Invalid ID format → raises ValueError |
| TS-01.9 | ER | DB connection lost → raises ConnectionError |
```

### 3.4 Why Lightweight Format?

- **Faster to write** than full Given-When-Then
- **Still unambiguous** - clear input → output
- **Claude Code expands** to full test during TDD (Arrange-Act-Assert structure)

---

## 4. CODE Phase: TDD Execution

### 4.0 Briefing Validation (Before Writing Tests)

**Why this step exists:** Task-07 revealed that briefing examples may contain logical errors. Example used 1M tokens but didn't account for premium multiplier (>200K threshold).

**Validation Checklist:**

| Check | Question | Action if Fails |
|-------|----------|-----------------|
| **Numeric Examples** | Are calculations mathematically correct? | Hand-calculate, ask orchestrator |
| **Rule Interactions** | Do multiple rules apply simultaneously? | Document rule interactions |
| **Thresholds** | Are edge cases near thresholds correct? | Test boundary values |
| **Consistency** | Do all examples use same assumptions? | Flag inconsistencies |

**Gate Function:**

```
BEFORE writing first test from briefing:

  1. Hand-calculate at least ONE numeric example
  2. Verify ALL applicable rules are accounted for
  3. Check threshold boundaries (200K, 1M, etc.)

  IF calculation doesn't match briefing:
    → STOP - Ask orchestrator for clarification
    → Don't assume either is correct
```

**Example - Task-07:**

```
Briefing: 1M tokens × $3/M = $3.00
Reality:  1M > 200K → premium multiplier applies
Correct:  1M × $3/M × 1.25 = $3.75
```

**Pre-flight Checklist (CC täyttää ennen toteutusta):**

Task-08 validointi paljasti lisää tarkistettavia asioita:

| Check | Question |
|-------|----------|
| **Edeltävät taskit** | Ovatko kaikki riippuvuudet valmiina? |
| **Testiluku** | Vastaako briefing nykyistä testien määrää? |
| **Async testit** | Onko kaikissa async def -testeissä @pytest.mark.asyncio? |
| **Ei placeholdereita** | Ovatko KAIKKI testit ajettavia (ei "TODO" -testejä)? |
| **Syntaksi** | Ovatko koodiesimerkit syntaksiltaan valideja? |

---

### The Iron Law (Obra Superpowers)

```
┌─────────────────────────────────────────────────────────────┐
│     NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST         │
└─────────────────────────────────────────────────────────────┘
```

Write code before test? **Delete it. Start over.**

This is non-negotiable. The Iron Law exists because:
- Tests written after code tend to test implementation, not behavior
- "I'll add tests later" becomes "tests never added"
- Code written without tests often has hidden assumptions

**Red Flags - STOP and Start Over:**

| Thought | Reality |
|---------|---------|
| "I already manually tested it" | Manual testing ≠ automated test |
| "Tests after achieve the same purpose" | They don't - they test what you built, not what you should have built |
| "It's about spirit not ritual" | The ritual IS the spirit |
| "This is different because..." | It's not different |

**All of these mean: Delete code. Start over with TDD.**

See Obra `test-driven-development` skill for full details.

### 4.1 The TDD Cycle (RGRC)

```
┌─────────────────────────────────────────────────────────────────┐
│                    TDD CYCLE: RGRC                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. RED     - Write a failing test                              │
│              (from TS-XX.Y in TECH_SPEC)                        │
│                                                                 │
│  2. GREEN   - Make it pass                                      │
│              (simplest possible implementation)                 │
│                                                                 │
│  3. REFACTOR - Clean up the code                                │
│              (improve design, no new functionality)             │
│                                                                 │
│  4. COMMIT  - Save your progress                                │
│              (meaningful commit message)                        │
│                                                                 │
│  → Repeat for next test scenario                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 TDD Workflow Example

**TECH_SPEC Scenario:** TS-01.1 - Valid MemoryItem → returns item with UUID

**Step 1: RED (Write failing test)**

```python
# tests/unit/test_memory_service.py

def test_store_returns_item_with_uuid(memory_service):
    """
    REQ: REQ-01 (Memory Storage)
    AC: AC-01 (Store returns unique ID)
    TS: TS-01.1 (Valid MemoryItem → returns item with UUID)
    Type: Happy Path
    """
    item = MemoryItem(content="test content", category="CONTEXT")
    
    result = memory_service.store(item)
    
    assert result.id is not None
    assert len(result.id) == 36  # UUID format
    assert result.content == "test content"
```

Run: `pytest tests/unit/test_memory_service.py::test_store_returns_item_with_uuid -v`
Result: FAILED ❌

**Step 2: GREEN (Make it pass)**

```python
# src/memory/service.py

class MemoryService:
    def store(self, item: MemoryItem) -> MemoryItem:
        item.id = str(uuid.uuid4())
        return item
```

Run: `pytest tests/unit/test_memory_service.py::test_store_returns_item_with_uuid -v`
Result: PASSED ✅

**Step 3: REFACTOR (Clean up)**

```python
# src/memory/service.py

class MemoryService:
    def __init__(self, repository: MemoryRepository):
        self._repository = repository
    
    def store(self, item: MemoryItem) -> MemoryItem:
        """Store a memory item and return it with assigned ID."""
        item.id = self._generate_id()
        self._repository.save(item)
        return item
    
    def _generate_id(self) -> str:
        return str(uuid.uuid4())
```

Run: All tests still pass ✅

**Step 4: COMMIT**

```bash
git add .
git commit -m "feat(memory): implement basic memory storage (TS-01.1)"
```

### 4.3 Adding Scenarios During TDD

Claude Code may discover additional edge cases during implementation:

```python
def test_store_handles_unicode_content(memory_service):
    """
    REQ: REQ-01 (Memory Storage)
    AC: AC-01 (Store returns unique ID)
    TS: TS-01.10 (ADDED - Unicode content handled correctly)
    Type: Edge Case
    Note: Added during implementation - not in original TECH_SPEC
    """
    item = MemoryItem(content="こんにちは 🎉", category="CONTEXT")
    
    result = memory_service.store(item)
    
    assert result.content == "こんにちは 🎉"
```

**Important:** Added scenarios should be reported and Traceability Matrix updated.

### 4.3.1 E11: Arrange-Act-Assert Test Structure

**Tarkoitus:** Varmistaa että Claude Code kirjoittaa testit selkeässä, ylläpidettävässä rakenteessa.

**Periaate:** Jokainen testi noudattaa kolmivaiheista rakennetta:

| Vaihe | Alias | Tarkoitus |
|-------|-------|-----------|
| **Arrange** | Given | Valmistele testidata ja tilanne |
| **Act** | When | Suorita testattava toiminto |
| **Assert** | Then | Tarkista lopputulos |

**Täydellinen esimerkki:**

```python
def test_search_empty_database(memory_service):
    """
    REQ: REQ-01 (Memory Storage)
    AC: AC-03 (Search returns empty list when no matches)
    TS: TS-02.1 (Empty database → empty results)
    Type: Happy Path
    """
    # Arrange (Given): Empty database
    # - handled by fixture, database is fresh
    
    # Act (When): Search for non-existent term
    results = memory_service.search('nonexistent')
    
    # Assert (Then): Returns empty list, not None or error
    assert results == []
    assert isinstance(results, list)
```

**Miksi eksplisiittiset kommentit?**

| Hyöty | Selitys |
|-------|---------|
| **Selkeys** | Testin rakenne näkyy yhdellä silmäyksellä |
| **Ylläpidettävyys** | Helpompi päivittää kun tietää mikä osa tekee mitä |
| **Code Review** | Tarkastaja näkee heti onko kaikki vaiheet |
| **Claude Code -ohjaus** | AI ymmärtää rakenteen ja tuottaa konsistenttia koodia |

**Erityistapaukset:**

```python
# Kun Arrange on tyhjä (fixture hoitaa):
def test_get_by_id_returns_stored_item(memory_service, sample_item):
    """..."""
    # Arrange: handled by fixtures (memory_service, sample_item)
    
    # Act
    result = memory_service.get_by_id(sample_item.id)
    
    # Assert
    assert result == sample_item

# Kun Assert on monivaiheinen:
def test_store_creates_complete_record(memory_service):
    """..."""
    # Arrange
    item = MemoryItem(content="test", category="CONTEXT")
    
    # Act
    stored = memory_service.store(item)
    
    # Assert - verify all aspects
    assert stored.id is not None          # ID assigned
    assert stored.content == "test"       # Content preserved
    assert stored.created_at is not None  # Timestamp set
    assert stored.category == "CONTEXT"   # Category preserved
```

**⚠️ Vältä:**
- `# Arrange` -kommentteja ilman sisältöä (kirjoita mitä fixture tekee)
- Monta `# Act` -vaihetta samassa testissä (jaa erillisiksi testeiksi)
- `# Assert` ilman selitystä (kerro MITÄ tarkistetaan)

### 4.3.2 Pre-written Tests from Orchestrator

Sometimes claude.ai (orchestrator) provides ready test code in briefings. This is still TDD-compliant IF Claude Code follows this protocol:

```
Briefing contains test code?
│
├─ YES → 1. Copy test to file
│        2. Run test FIRST (before any production code)
│        3. Verify RED - test must FAIL
│        4. Only then write production code (GREEN)
│
└─ NO → Write test yourself (normal TDD)
```

**Why this matters:**
- The Iron Law requires seeing the test fail first
- Pre-written tests don't violate TDD if you verify RED
- Skipping RED verification defeats the purpose of TDD

**Anti-pattern:** Receiving test + implementation code together breaks TDD. If briefing contains both, ignore the implementation and start fresh.

| Briefing content | TDD-compliant? | CC action |
|------------------|----------------|-----------|
| TS descriptions only | ✅ Yes | Write test, verify RED |
| Ready test code | ✅ Yes | Run test, verify RED |
| Test + implementation | ❌ No | Use test only, ignore code |

### 4.4 Test File Structure

```
tests/
├── conftest.py              # Shared fixtures
├── pytest.ini               # Pytest configuration
├── unit/
│   ├── __init__.py
│   ├── test_memory_item.py
│   ├── test_text_splitter.py
│   └── test_embedding_client.py
├── integration/
│   ├── __init__.py
│   ├── test_memory_service.py
│   ├── test_sqlite_repository.py
│   └── test_context_manager.py
└── e2e/
    ├── __init__.py
    ├── test_memory_workflow.py
    └── test_chat_workflow.py
```

### 4.5 Pytest Configuration

**pytest.ini:**

```ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = -v --tb=short
markers =
    unit: Unit tests
    integration: Integration tests
    e2e: End-to-end tests
    slow: Tests that take > 1 second
```

**conftest.py (shared fixtures):**

```python
import pytest
from pathlib import Path
import tempfile

@pytest.fixture(scope="function")
def tmp_db_path(tmp_path):
    """Provides a temporary database path."""
    return tmp_path / "test.db"

@pytest.fixture(scope="function")
def memory_service(tmp_db_path):
    """Provides a fresh MemoryService for each test."""
    from memory_service import MemoryService
    service = MemoryService(db_path=str(tmp_db_path))
    yield service
    # Cleanup handled by tmp_path

@pytest.fixture(scope="module")
def sample_memory_items():
    """Shared test data for memory items."""
    from models import MemoryItem
    return [
        MemoryItem(content="Test content 1", category="CONTEXT"),
        MemoryItem(content="Test content 2", category="DECISION"),
        MemoryItem(content="Test content 3", category="SUMMARY"),
    ]
```

---

## 5. Running Tests

### 5.1 Common Commands

```bash
# Run all tests
pytest

# Run with verbose output
pytest -v

# Run specific test file
pytest tests/unit/test_memory_service.py

# Run specific test
pytest tests/unit/test_memory_service.py::test_store_returns_item_with_uuid

# Run by marker
pytest -m unit
pytest -m integration
pytest -m "not slow"

# Run with coverage
pytest --cov=src --cov-report=html

# Run and stop on first failure
pytest -x

# Run failed tests from last run
pytest --lf
```

### 5.2 Test Markers

Use markers to categorize tests:

```python
import pytest

@pytest.mark.unit
def test_simple_function():
    pass

@pytest.mark.integration
def test_database_connection():
    pass

@pytest.mark.e2e
def test_full_workflow():
    pass

@pytest.mark.slow
def test_large_dataset():
    pass
```

---

## 6. Anti-Patterns to Avoid

### 6.1 Testing Anti-Patterns

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| **Ice Cream Cone** | Too many E2E, too few unit | Follow 40/45/10/5 distribution |
| **Testing Implementation** | Tests break on refactor | Test behavior, not internals |
| **Over-Mocking** | Tests don't reflect reality | Prefer integration over mocks |
| **Flaky Tests** | Random pass/fail | Remove timing dependencies |
| **Test After** | Tests as afterthought | TDD - tests first, always |
| **No Assertion** | Tests that can't fail | Every test needs clear assertion |
| **Giant Tests** | 100+ line test methods | Split into focused tests |
| **No Traceability** | Can't trace test to requirement | Always include REQ/AC/TS in docstring |

### 6.2 Good vs. Bad Tests

**BAD: Testing implementation details**

```python
# ❌ This test breaks if implementation changes
def test_store_calls_repository_save():
    service = MemoryService(mock_repo)
    service.store(item)
    mock_repo.save.assert_called_once()  # Tests HOW, not WHAT
```

**GOOD: Testing behavior with traceability**

```python
# ✅ This test survives refactoring and is traceable
def test_store_persists_item(memory_service):
    """
    REQ: REQ-01, AC: AC-02, TS: TS-02.1
    """
    service.store(item)
    result = service.get_by_id(item.id)
    assert result.content == item.content  # Tests WHAT
```

---

## 7. Definition of Done (Testing)

### 7.1 Per Feature

- [ ] All TECH_SPEC test scenarios have corresponding tests
- [ ] Tests follow TDD cycle (written before implementation)
- [ ] Tests include REQ/AC/TS traceability in docstring
- [ ] No flaky tests

### 7.2 Per Module

- [ ] Unit test coverage > 80%
- [ ] Integration tests for all public APIs
- [ ] All tests pass locally
- [ ] Static analysis passes (mypy, ruff)
- [ ] Traceability Matrix updated in TECH_SPEC

### 7.3 Per Release

- [ ] All module tests pass
- [ ] E2E tests pass
- [ ] No regressions
- [ ] Performance thresholds met

---

## 8. Integration with Development Process

### 8.1 When Testing Happens

| Phase | Testing Activity |
|-------|------------------|
| **SPEC** | Define AC (WHAT to verify) |
| **User Review** | Review AC completeness |
| **Gemini Review** | Validate AC coverage |
| **TECH_SPEC** | Define TS (scenarios, lightweight format) |
| **CODE** | TDD execution (HOW to test) |
| **Commit** | All tests must pass |
| **PR** | CI runs all tests |

### 8.2 Traceability Matrix Update

After CODE phase, update Traceability Matrix in TECH_SPEC:

```markdown
| REQ | AC | Task | Test Scenario | Test | Status |
|-----|-----|------|---------------|------|--------|
| REQ-01 | AC-01 | Task-01 | TS-01.1 | test_store_returns_uuid | ✅ |
| REQ-01 | AC-01 | Task-01 | TS-01.2 | test_store_preserves_metadata | ✅ |
| REQ-01 | AC-02 | Task-01 | TS-02.1 | test_store_creates_file | ✅ |
```

---

## 9. Quick Reference

### 9.1 TDD Cycle Checklist

For each Test Scenario in TECH_SPEC:

1. [ ] Write failing test (RED) with REQ/AC/TS docstring
2. [ ] Verify test fails for right reason
3. [ ] Write minimal code to pass (GREEN)
4. [ ] Verify test passes
5. [ ] Refactor if needed (REFACTOR)
6. [ ] Verify tests still pass
7. [ ] Commit (COMMIT)
8. [ ] Update Traceability Matrix

### 9.2 Test Naming Convention

```
test_[method]_[scenario]_[expected_result]

Examples:
- test_store_valid_item_returns_uuid
- test_search_empty_query_returns_empty_list
- test_get_by_id_invalid_id_raises_value_error
```

### 9.3 Fixture Naming Convention

```
[resource]_[variation]

Examples:
- memory_service
- memory_service_with_data
- sample_memory_items
- tmp_db_path
```

### 9.4 Docstring Template

```python
def test_xxx():
    """
    REQ: REQ-XX (Requirement name)
    AC: AC-XX (Acceptance criterion)
    TS: TS-XX.Y (Scenario description)
    Type: Happy Path / Edge Case / Error Case
    """
```

---

## Change History

| Version | Date | Changes |
|---------|------|---------|
| 1.5 | 2025-12-16 | **Pre-flight Checklist:** CC:n tarkistuslista briefingille (async, testiluku, placeholderit) |
| 1.4 | 2025-12-16 | **Briefing Validation (4.0):** Gate function ennen testien kirjoittamista, Task-07 esimerkki |
| 1.3 | 2025-12-16 | **Obra Integration:** Iron Law -osio, Pre-written Tests from Orchestrator (4.3.2), Red Flags |
| 1.2 | 2025-12-15 | **E11: Arrange-Act-Assert** osio lisätty (4.3.1), täydelliset esimerkit Claude Code -ohjaukseen |
| 1.1 | 2025-12-04 | **Audit Trail Hierarchy**, TS-XX.Y naming, TECH_SPEC phase, lightweight format, traceability in docstrings. All v1.0 content preserved. |
| 1.0 | 2025-12-01 | Initial version: TDD process, pytest configuration, anti-patterns, Definition of Done |

---

*This document is part of the Claude API Planning Tool project documentation.*

