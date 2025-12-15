# Testing the Amplifier Dev Collection

This directory contains test cases for validating the collection works correctly.

## Quick Start

### 1. Register the Collection Locally

Add to `.amplifier/settings.yaml`:

```yaml
collection_sources:
  amplifier-dev: ./amplifier-collection-amplifier-dev
```

### 2. Run Manual Tests

```bash
cd /path/to/amplifier-dev

# Test contract fetching
amplifier run --profile amplifier-dev:amplifier-dev \
  "What is the Tool contract? Show me the required interface."

# Test module scaffolding
amplifier run --profile amplifier-dev:amplifier-dev \
  "Help me create a new tool called 'weather' that fetches weather data."

# Test profile design
amplifier run --profile amplifier-dev:amplifier-dev \
  "Help me design a profile for code review that only has read access."
```

### 3. Verify Expected Behaviors

Check the test response against `expected_behavior` in `profile-tests.yaml`.

## Test Categories

| Tag | What It Tests |
|-----|---------------|
| `contracts` | Fetching and explaining module contracts |
| `webfetch` | That tool-web is used to get latest docs |
| `fallback` | Embedded summaries work when offline |
| `module-author` | Scaffolding new modules |
| `profile-designer` | Designing profiles and collections |
| `amplifier-explainer` | Explaining architecture concepts |
| `edge-case` | Handling ambiguous or unusual requests |

## Automated Testing (Future)

When the `profile-comparison` orchestrator is implemented:

```bash
# Compare against baseline (regression testing)
amplifier compare \
  --baseline amplifier-dev:amplifier-dev \
  --test ./profiles/amplifier-dev.md \
  --test-suite ./tests/profile-tests.yaml \
  --fail-on-regression

# Run full test suite
amplifier test --profile amplifier-dev:amplifier-dev \
  --suite ./tests/profile-tests.yaml \
  --output ./test-results.json
```

## Writing New Tests

Add to `profile-tests.yaml`:

```yaml
- id: unique-test-id
  prompt: "The prompt to send"
  tags: [category1, category2]
  expected_behavior: |
    - First expected behavior
    - Second expected behavior
    - What the response should contain/do
```

### Good Test Characteristics

1. **Specific**: Tests one capability
2. **Observable**: Expected behavior is verifiable
3. **Independent**: Doesn't depend on other tests
4. **Documented**: Tags explain what's being tested

## CI Integration (Future)

```yaml
# .github/workflows/test-collection.yml
name: Test Collection

on:
  push:
    paths:
      - 'amplifier-collection-amplifier-dev/**'

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Amplifier
        run: uv tool install amplifier

      - name: Register collection
        run: |
          mkdir -p ~/.amplifier
          echo "collection_sources:
            amplifier-dev: ./amplifier-collection-amplifier-dev" > ~/.amplifier/settings.yaml

      - name: Run tests
        run: |
          amplifier test \
            --profile amplifier-dev:amplifier-dev \
            --suite ./amplifier-collection-amplifier-dev/tests/profile-tests.yaml \
            --output ./results.json

      - name: Check results
        run: |
          # Parse results.json and fail if any tests failed
          python scripts/check-test-results.py ./results.json
```
