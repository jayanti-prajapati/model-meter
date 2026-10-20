# Use Cases

Real-world scenarios where Model Meter helps engineering teams understand and control their AI spend.

---

## 1. Attribute AI spend to features and projects

### The problem

Your company has five AI-powered features: a support bot, a code review assistant, a content generator, an email summarizer, and a search re-ranker. The OpenAI invoice arrives every month showing a single total. There is no breakdown by feature, team, or environment. When the CFO asks which product line drove last month's 40% cost increase, nobody has an answer.

### How Model Meter helps

Add `projectId` to each `init()` call (or use one API key per project):

```ts
// support-bot/instrumentation.ts
init({ apiKey: '...', endpoint: '...', projectId: 'support-bot', environment: 'production' });

// code-review/instrumentation.ts
init({ apiKey: '...', endpoint: '...', projectId: 'code-review', environment: 'production' });
```

Within one sprint, the dashboard shows a cost breakdown by project, provider, and model. The support bot — running on GPT-4o with long system prompts — turns out to be 68% of spend. The team migrates the classification step to GPT-4o-mini and saves 30% without changing user-visible quality.

### Outcome

- Spend is attributed, not guessed
- Product managers can see the cost of their feature
- Finance can report by cost center

---

## 2. Stop surprise bills before they arrive

### The problem

A junior developer ships a new product recommendation feature on a Friday afternoon using `gpt-4o` with a 4,000-token system prompt. By Monday morning the feature has made 500,000 calls. The bill is $4,000 higher than expected. The team only discovers this on the 1st of the month.

### How Model Meter helps

Set a project-level budget (Milestone 3):

```
Project: product-recommendations
Monthly budget: $500
Alert at: 80% ($400)
```

When spending crosses $400, an alert fires — on Friday, not the following Monday. The team can investigate and rate-limit or switch models before the bill escalates.

The spend time series in the dashboard would have shown the spike on Friday afternoon. Even before budget alerts ship, the dashboard gives engineers and engineering managers a near-real-time view of AI spend by project.

### Outcome

- Anomalous spend is visible within hours, not weeks
- Budget owners receive alerts before limits are exceeded
- Engineers develop a cost-awareness habit

---

## 3. Choose the right model for each task

### The problem

Your team uses GPT-4o for every AI task — because it's the most capable and nobody has time to evaluate alternatives. You suspect the search re-ranker and email classifier don't need GPT-4o, but you've never measured it.

### How Model Meter helps

The cost-to-quality workflow (Milestone 6) runs both models on the same evaluation dataset and compares:
- Estimated cost per 1,000 requests
- Task-specific quality score (precision, recall, or custom metric)
- Median and p95 latency
- Error rate

For the email classifier: GPT-4o-mini achieves 94% accuracy vs. GPT-4o's 96%, at 1/10 the cost per request. The 2-percentage-point quality difference is acceptable. The team migrates the classifier and saves $1,200/month.

Even before Milestone 6, the cost breakdown in the dashboard makes the relative cost of each model visible — often enough to spark the right conversation.

### Outcome

- Model selection is data-driven, not guess-based
- Lower-cost models are used where quality constraints allow
- Teams can experiment with model changes and immediately see cost impact

---

## 4. Reconcile AI costs with provider invoices

### The problem

Model Meter estimates $8,400 for the month based on request-level token counts. The Anthropic invoice says $9,100. The $700 difference is unexplained. Is it pricing changes? Context caching? Batch discounts? Management wants an explanation.

### How Model Meter helps

The billing reconciliation feature (Milestone 5) imports the provider's usage data and compares it to Model Meter's estimates on a per-request or per-day basis. The reconciliation report shows:
- Which models have price differences (and when prices changed)
- How much of the gap is from cached vs. non-cached tokens
- Any credits, adjustments, or line items in the provider invoice not reflected in estimates

The two figures — estimated and provider-reported — are always stored separately. Model Meter never pretends the estimate equals the invoice.

### Outcome

- Finance gets a clear, line-item explanation of the invoice
- Discrepancies between estimates and actuals are documented
- The team can verify that Model Meter's pricing catalogue is accurate

---

## 5. Monitor multi-provider AI spend in one place

### The problem

Your platform calls OpenAI for code generation, Anthropic for customer-facing content, Google Gemini for embedding and search, and Groq for latency-sensitive classification. Cost data lives in four separate provider dashboards with different billing periods, currencies (all USD, but different invoice dates), and data export formats. Nobody has a single view.

### How Model Meter helps

All providers flow through one SDK integration. The dashboard's provider breakdown shows spend side-by-side:

```
October 2026 spend by provider:
  OpenAI      $3,240  (47%)
  Anthropic   $2,100  (30%)
  Google      $1,050  (15%)
  Groq          $560   (8%)
```

The breakdown can be sliced by model, project, environment, and date range. Historical comparisons show which provider's spend changed — and which project drove that change.

### Outcome

- Single pane of glass for multi-provider AI spend
- No manual consolidation of four different provider dashboards
- Spend trends are visible across providers with consistent attribution

---

## More patterns

See [docs/FAQ.md](FAQ.md) for common questions, and the [Roadmap](ROADMAP.md) for the full list of planned features covering each of these use cases.
