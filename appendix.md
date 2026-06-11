# Appendix

## A. Experiment Configuration

### Configuration

| Configuration | Value |
|---|---|
| Data Source | ~3,900 call center interactions |
| Representatives | 33 |
| Extracted Features | 20 |
| Model | Claude Opus 4.5 via AWS Bedrock |
| Temperature | Default |
| Runs Per Condition | 10 |
| Experiment Date | February 2026 |

### The Experiments

| Experiment | Query |
|---|---|
| Ranking Consistency | “Who are my top 10 best reps based on overall performance metrics?” |
| Query Phrasing Sensitivity | Four phrasings of the same question (2 runs each = 8 without, 10 with workflow) |
| Aggregation and Trends | “Calculate the average sentiment score grouped by category, and identify declining trends.” |
| Edge Cases | “Find reps with compliance risks who also have high customer satisfaction.” |

## B. Metrics Glossary

| Metric | Description |
|---|---|
| Jaccard Similarity | \|Alice ∩ Ben\| / \|Alice ∪ Ben\| — measures set overlap between rankings |
| Top N Pairwise Overlap | average of \|common reps in top N\| / N across all run pairs |
| Top-1 Consistency | frequency of most common #1 representative / total runs |
| Count Variance | statistical variance in result counts across runs |
| Steps | number of reasoning/tool-call messages in the session |
| Structural Similarity | A comparison of response layout, including section order, table presence, field names, and formatting consistency across runs. |
| Result Count Consistency | The percentage of runs that returned the most common result count for that condition. |
| FCR | First call resolution — the share of customer issues resolved on the first contact, used as one of the weighted performance metrics in the workflow’s scoring formula. |

## C. Example Workflow (“Golden Run”)

The workflow system captures a validated “golden run” and replays the same analytical process across future executions. The figure shows the workflow structure used during Experiment 1. Each step’s code is found below.

<!-- Insert Golden Run Workflow Diagram Here -->

### Step 1: Load and Explore Data Schema

The workflow first inspects the dataset structure and validates required columns.

```python
import polars as pl

df = df_ut_vcc_enr__experiment022

print(df.columns)
print(df.schema)
```

### Step 2: Aggregate Agent Performance Metrics

The workflow computes average performance metrics for each representative.

```python
agent_metrics = (
    df.group_by("agent_id")
    .agg([
        pl.mean("customer_sentiment_score")
            .alias("avg_sentiment"),

        pl.mean("agent_empathy_score")
            .alias("avg_empathy"),

        pl.mean("issue_resolution_status")
            .alias("resolution_rate")
    ])
)
```

### Step 3: Normalize Metrics

The workflow normalizes metrics into a consistent scoring range.

```python
from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler()

pdf[metric_columns] = scaler.fit_transform(
    pdf[metric_columns]
)
```

### Step 4: Calculate Composite Scores

The workflow applies the fixed ranking methodology from the golden run.

```python
pdf["composite_score"] = (
    pdf["avg_sentiment"] * 0.15 +
    pdf["avg_empathy"] * 0.15 +
    pdf["resolution_rate"] * 0.20
)
```

### Step 5: Collect Evidence Call GUIDs

The workflow gathers supporting evidence examples for top-ranked agents.

```python
evidence_calls = (
    df.filter(
        pl.col("agent_id")
            .is_in(top_agents["agent_id"].tolist())
    )
    .select(["agent_id", "call_guid"])
)
```

### Step 6: Create Visualization

The workflow generates reproducible charts from the ranked output.

```python
fig = px.bar(
    top_agents,
    x="agent_id",
    y="composite_score"
)

fig.show()
```

### Step 7: Format Final Response

The workflow formats the final ranked response structure.

```python
final_output = top_agents[
    ["agent_id", "composite_score"]
].to_markdown(index=False)

print(final_output)
```

## D. Validation and Edge Cases

The workflow also includes validation checks and edge-case handling to improve stability across repeated executions. These checks help prevent inconsistent rankings caused by missing data, sparse samples, tied scores, or incomplete evidence references.

### Validation Checks

The workflow validates required columns and key metric quality before scoring begins.

```python
required_columns = [
    "agent_id",
    "customer_sentiment_score",
    "agent_empathy_score"
]

missing = [
    c for c in required_columns
    if c not in df.columns
]

if missing:
    raise ValueError(f"Missing columns: {missing}")
```

The workflow also checks for null values in critical ranking metrics.

```python
null_counts = df.select([
    pl.col("customer_sentiment_score").null_count(),
    pl.col("agent_empathy_score").null_count()
])

print(null_counts)
```

### Edge Cases

The workflow handles small datasets gracefully when fewer than 10 agents are available.

```python
top_n = min(10, len(pdf))

top_agents = (
    pdf.sort_values(
        "composite_score",
        ascending=False
    )
    .head(top_n)
)
```

The workflow also stabilizes tied rankings using deterministic sorting rules.

```python
top_agents = top_agents.sort_values(
    by=["composite_score", "agent_id"],
    ascending=[False, True]
)
```

If evidence references are unavailable, the workflow skips the evidence section rather than failing execution.

```python
if "call_guid" not in df.columns:
    evidence_calls = None
```