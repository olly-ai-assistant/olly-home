---
name: smoke-test-generator
description: This skill should be used when generating smoke tests for code changes, APIs, or features. Triggers on "smoke test", "generate tests", "write a test", "test this code", or when asked to create minimal test coverage.
---

# Smoke Test Generator

This skill generates minimal, fast smoke tests that verify core functionality without exhaustive coverage.

## When to Use This Skill

- After implementing a new feature
- After making changes to existing code
- When asked to "test" or "write tests for" something
- Before deployments to verify basic functionality
- During code review to validate changes

## What Makes a Good Smoke Test

1. **Fast**: Runs in milliseconds, not seconds
2. **Focused**: Tests ONE thing per test
3. **Deterministic**: Same result every time
4. **Independent**: No dependency on other tests
5. **Clear failure**: Exact error message when it fails

## Test Generation Process

### Step 1: Identify the Core Function

What is the primary purpose of this code?
- Authentication? → Test login success + failure
- Data retrieval? → Test fetch with valid + invalid input
- Data mutation? → Test create/update with valid + invalid data
- API endpoint? → Test HTTP status + response shape

### Step 2: Determine Test Cases

For each function/endpoint, generate:

```
Happy Path     → Does it work with valid input?
Invalid Input  → Does it fail gracefully?
Edge Cases     → Empty, null, boundary values
```

### Step 3: Write the Test

```javascript
// Template for function smoke test
describe('functionName', () => {
  it('should return expected result with valid input', () => {
    const result = functionName(validInput);
    expect(result).toMatchObject({ /* expected shape */ });
  });

  it('should throw/return error with invalid input', () => {
    expect(() => functionName(invalidInput)).toThrow(/expected error/);
  });
});
```

## Language/Framework Templates

### JavaScript/TypeScript (Jest/Vitest)

```javascript
describe('moduleName', () => {
  it('should handle valid input', () => {
    expect(module.function(input)).toBe(expectedOutput);
  });

  it('should reject invalid input', () => {
    expect(() => module.function(badInput)).toThrow();
  });
});
```

### Python (pytest)

```python
def test_function_valid_input():
    result = function(valid_input)
    assert result == expected_output

def test_function_invalid_input():
    with pytest.raises(ExpectedError):
        function(invalid_input)
```

### Bash/Shell

```bash
@test "command works with valid args" {
  run command --valid-arg
  assert_success
  assert_output --partial "expected text"
}
```

## API Endpoint Smoke Test Template

```javascript
describe('GET /api/resource/:id', () => {
  it('returns 200 with valid id', async () => {
    const res = await request(app).get('/api/resource/123');
    expect(res.status).toBe(200);
    expect(res.body).toHaveProperty('id');
  });

  it('returns 404 for non-existent id', async () => {
    const res = await request(app).get('/api/resource/999999');
    expect(res.status).toBe(404);
  });
});
```

## Smoke Test Checklist

Before finalizing, verify:

- [ ] Test runs in <1 second total
- [ ] Each test has exactly one assertion focus
- [ ] Tests are idempotent (can run multiple times)
- [ ] Tests don't share mutable state
- [ ] Failure message clearly indicates what broke
- [ ] Tests cover the primary use case

## Minimal vs Exhaustive

**Smoke tests are NOT full test suites.** They verify:
- Does it run at all?
- Does the happy path work?
- Do obvious edge cases not crash it?

**Skip for smoke tests:**
- Detailed validation coverage
- Performance/load testing
- Security penetration testing
- Full edge case enumeration

## Anti-Patterns

| Bad | Good |
|-----|------|
| One test with 20 assertions | Multiple focused tests |
| Tests depend on execution order | Tests are independent |
| Slow database queries in every test | Use mocks/fixtures |
| Tests read from real external APIs | Mock external calls |
| 500 lines for one feature test | <50 lines per test |

## Output Location

Save smoke tests in:
- `tests/smoke/` (general)
- `src/**/*.test.js` or `src/**/*.spec.js` (co-located)
- `tests/api/smoke.test.js` (API tests)

Use naming convention: `*.smoke.test.js` or `*.smoke.spec.js` to distinguish from unit tests.
