# Lens: uncommitted code

The code is fresh and unreviewed, so hunt for bugs before design.

Focus:
- **Bug-prone spots:** off-by-ones, null or undefined paths, type coercion surprises.
- **Error handling:** try/catch coverage, unhandled async rejections, fallbacks that hide failures.
- **Edge cases in new logic:** empty input, a single item, the maximum, malformed data.
- **Untested branches:** new code paths with no test.
- **Input boundaries:** sanitization, injection, auth checks.

Don't report:
- missing docstrings, comments or type hints;
- unused imports;
- a narrower exception type, unless the broad handler hides a real failure.

These are noise at this stage, and lint tools catch them.
