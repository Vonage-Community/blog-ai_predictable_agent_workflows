# Build Predictable Agentic Workflows

Supporting materials for the Vonage Developer article:

> **Your AI Agent Is Lying to You: How Workflows Create Reliable Results**

This repository accompanies the research and experiments described in the blog post, exploring how workflow-driven execution can improve consistency and reproducibility in agentic AI systems.

You can find the full article on the Vonage Developer Blog:

➡️ https://developer.vonage.com/

## About This Repository

AI agents are inherently probabilistic. When the same analytical query is executed multiple times, agents may select different metrics, apply different thresholds, or generate entirely different methodologies—even when the underlying data has not changed.

This repository contains the supporting materials used to investigate whether **validated workflows ("golden runs") can reduce that variability** and make agentic systems more predictable in production environments.

The experiments focused on four common analytical scenarios:

- Ranking consistency
- Query phrasing sensitivity
- Aggregation and trend analysis
- Edge cases involving ambiguous criteria

## Repository Contents

```
.
├── README.md
├── appendix.md
├── golden-run-workflow.md
├── references.md
├── LICENSE.md
└── workflow-diagrams/
```

### `appendix.md`

Additional experiment details, configuration information, metric definitions, and supporting notes referenced throughout the article.

### `golden-run-workflow.md`

The complete seven-step workflow used during Experiment 1, including the example code, validation checks, and edge-case handling used to stabilize repeated executions.

### `workflow-diagrams/`

The diagrams and visualizations created to illustrate how methodology drift occurs and how workflows can improve reproducibility.

### `references.md`

Additional reading and related resources covering agentic systems, ReAct loops, and reproducible analytical workflows.

## Important Notes

The examples in this repository are intended to demonstrate **workflow design patterns and experimental methodology**.

They are **not production-ready implementations**, and they have been simplified to focus on the concepts discussed in the article.

The datasets used in the experiments are not included.

## Key Takeaway

Dynamic reasoning is one of the greatest strengths of agentic AI systems—but it also introduces variability.

Workflows provide a practical mechanism for constraining critical analytical decisions so that the same methodology can be reused across repeated executions when consistency matters.

## Questions or Feedback?

If you have questions, ideas, or experiences building reproducible agentic systems, we'd love to hear from you.

You can reach the Vonage Developer Relations team at:

➡️ https://developer.vonage.com/