# Cost Engine and Cost-to-Quality

## Goals

- Produce transparent, reproducible estimates.
- Support effective-dated model prices.
- Preserve provider-specific billable usage dimensions.
- Separate request estimates from provider-reported billing.
- Compare model costs only on equivalent workloads and evaluation datasets.

## Basic text-token calculation

For a provider whose prices are stated per million tokens:

`cost = input_tokens * input_rate / 1_000_000`
`      + output_tokens * output_rate / 1_000_000`
`      + cached_input_tokens * cached_input_rate / 1_000_000`

This formula is illustrative. Actual billing semantics vary. Ensure cached input is not also counted as ordinary input, and add support for provider-specific reasoning tokens, image/audio units, batch rates, tool use, discounts, and other billable categories as needed.

## Pricing catalogue

Each price record should include:
- Provider and exact model/price identifier.
- Effective start and optional end.
- Billable unit and rate.
- Currency.
- Source/reference and last verification time.
- Version and change history.

Never assume a model alias maps permanently to the same model or price. Unknown models should be marked as unpriced instead of assigned a guessed rate.

## Monetary precision

Use PostgreSQL `NUMERIC` or integer minor units, not JavaScript floating-point as the final accounting representation. Use decimal arithmetic in the cost engine. Preserve the inputs and price version used to calculate each estimate.

## Provenance

Display labels:
- Estimated — calculated from request usage and a Model Meter price record.
- Provider reported — returned by provider usage/billing data.
- Reconciled — matched or adjusted according to an explicit rule.

Billing imports can include credits, taxes, discounts, minimums, adjustments, and currency differences. Do not expect request estimates to exactly equal an invoice.

## Cost-to-quality workflow

1. Define a task and representative evaluation dataset.
2. Run candidate models against the same cases and relevant settings.
3. Evaluate task-specific quality, cost, latency, and failure rates.
4. Calculate cost per successful task where success can be measured.
5. Require a minimum sample size and inspect uncertainty.
6. Recommend a candidate only when it meets explicit quality and latency constraints.
7. Keep the initial workflow offline and human-approved.

Do not reduce all tasks to one generic “AI quality score.” A model suitable for classification may not be suitable for code generation or support responses.

## Suggested initial metrics

- Estimated cost per request.
- Cost per successful task.
- Input/output/cached token totals.
- Median and p95 latency.
- Error and retry rates.
- Quality score by task and evaluation version.
- Estimated savings versus a documented baseline.

Savings must be reported as estimates with a stated baseline, not as guaranteed outcomes.
