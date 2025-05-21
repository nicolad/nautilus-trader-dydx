# Order Submission Refactor TODO

This document collects notes for future refactoring of the order submission flow. The goal is to split logic into dedicated methods based on order duration and conditional triggers.

## Overview
- **Short-term Orders**: Designed for high frequency or intraday trading. These methods should focus on low latency submission and may include options for immediate cancellation if not filled quickly.
- **Long-term Orders**: Suitable for swing or position trades. They may require additional validation around funding rates or rollover mechanisms.
- **Conditional Orders**: Triggered when specific market conditions are met, such as price thresholds, indicators, or external signals. These methods should handle the evaluation of conditions and ensure correct order placement when triggered.

## Proposed Steps
1. Review current `ExecutionClient` API to identify shared logic between order types.
2. Design new method signatures:
   - `submit_short_term_order(...)`
   - `submit_long_term_order(...)`
   - `submit_conditional_order(...)`
3. Extract common components (serialization, validation, error handling) into private helper functions.
4. Update unit tests to cover each method independently.
5. Deprecate the existing monolithic `submit_order` method after migration.

The timeline for this work is flexible. Feedback is welcome through PR review since direct coding time is limited.
