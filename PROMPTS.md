# Module 4 – Hands-on

## Session 4A – AI-Assisted Debugging

### Part A – Stack Trace Analysis

### Attempt 1 — First stack-trace line

The AI explained that the test failed because a customer with exactly
5 years of loyalty was charged $95 instead of the expected $90. The
most likely cause identified was that the loyalty discount boundary
for the 10% tier may be incorrect.

### Attempt 2

The test `quotesDiscountForLoyalCustomer` failed on its price check. Nothing crashed before that. JUnit compared two numbers and they did not match.

The test builds a $100 order for a customer with exactly 5 loyalty years and no tax. It expects $90 (10% off, tier 3). `PriceEngine.quote` returned $95.

The loyalty rules in `PriceEngine` say tier 3 starts at 5 years and is 10% off. The branch that applies 10% uses `years > 5`, so only 6 years and up get that rate. A customer with exactly 5 years misses it and falls into the next branch, `years >= 3`, which is the 5% tier.

$100 minus 5% is $95, which is the value in the assertion.

**Most likely root cause:** an off-by-one on the tier-3 boundary. Five years is documented as the start of the 10% tier, but the condition still treats 5 years as the 5% tier.

### My Assessment

The AI's diagnosis matches the expected and actual values and identifies the boundary condition causing the failure. The additional context confirms where the failure occurs, although the first error line already strongly suggested the loyalty-tier boundary issue.

## Part B – Propose and Verify the Patch

### AI Proposed Patch

The AI proposed the following minimal patch:

```diff
- if (years > 5) {
+ if (years >= 5) {

### Verification

The patch was applied and verified using `make test`.

Result: 10 tests successful, 0 tests failed.

The fix resolved the original `quotesDiscountForLoyalCustomer` failure
without causing any regression in the other tests.

## Session 4B – Reducing Cyclomatic Complexity

### Part A – Refactor Target Ranking

### Initial Complexity

Manual estimate: approximately 18–19, depending on whether the
compound `&&` condition is counted separately.

PMD was not available because the provided Makefile does not contain
a `pmd` target.

### AI Ranking

1. `quote`
   - Current cyclomatic complexity: approximately 18–19
   - First refactor: `extract-method`
   - Target: loyalty-tier calculation block
   - Expected reduction: approximately 3 decision points in `quote`

`PriceEngine` contains only one method, so there were no other methods
to rank.

## Part B – Extract Method Refactor

### Selected Refactor

The loyalty-tier discount calculation was selected for extraction.

### AI Proposed Refactor

The AI extracted the loyalty-tier discount calculation and application
into the private helper method `applyLoyaltyDiscount(...)`.

The public method signature of `quote(Order, Customer)` was unchanged.

### Refactor Review

The refactor preserves the existing loyalty-tier conditions and
discount arithmetic. The fixed `years >= 5` condition remains unchanged.
No public method signatures were changed, and no early returns were
introduced or moved.

### Verification

The refactored code was compiled and tested using `make test`.

Result: 10 tests successful, 0 tests failed.

The refactor did not introduce any regression.

## Part D – Reflection

### Complexity

The original `quote` method was estimated at approximately 18–19
cyclomatic complexity. After extracting the loyalty-tier discount
block, the main `quote` method has fewer decision points while the
behavior remains unchanged.

PMD was not available because the provided Makefile does not contain
a `pmd` target, so the complexity reduction was not measured using PMD.

### SpotBugs

SpotBugs was not run because the provided Makefile does not contain
a SpotBugs target.

### Rejected Refactor

The AI ranking identified `extract-method` as the first refactor move
for `quote`. No alternative refactor was proposed in the ranking, so
I did not reject an AI-proposed refactor during this task.