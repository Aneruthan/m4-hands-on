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