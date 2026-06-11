# Golden Run Workflow

This document contains the full workflow generated during **Experiment 1** — the **"golden run"** that the agent subsequently replayed across 10 additional executions.

Each step captures:
- the **purpose** of the step,
- the **tool or action** used,
- the **exact code or query** executed,
- the **expected output**, and
- the **error-handling strategy** used if the step could not be completed as intended.

The purpose of preserving this workflow was to stabilize the analytical methodology across repeated executions, reducing variability introduced by dynamic agent reasoning.

---

## Step 1: Load and Explore Data Schema

**Purpose**

Understand the available columns and data types in the VCC dataset to identify relevant performance metrics.

**Tool / Action**

Use `python_interpreter` with Polars.

**Code / Query**

```python
import polars as pl

df = df_ut_vcc_encr__experiment022

print("Columns:", df.columns)
print("Schema:", df.schema)
```

**Expected Output**

A list of column names including:

- `agent_id`
- `agent_empathy_score`
- `agent_script_adherence_score`
- `customer_overall_sentiment_score`
- `first_call_resolution_indicator`
- `customer_frustration_level`
- `issue_resolution_status`
- `call_guid`

**Error Handling**

If the dataframe cannot be found, check the available dataframes in the execution environment and use the appropriate VCC dataset.

---

## Step 2: Aggregate Agent Performance Metrics

**Purpose**

Calculate average performance metrics for each agent across all of their calls.

**Tool / Action**

Use `python_interpreter` with Polars `group_by` aggregation.

**Code / Query**

```python
agent_metrics = (
    df.group_by("agent_id")
    .agg([
        pl.count("call_guid").alias("total_calls"),
        pl.mean("agent_empathy_score").alias("avg_empathy"),
        pl.mean("agent_script_adherence_score").alias("avg_script_adherence"),
        pl.mean("customer_overall_sentiment_score").alias("avg_customer_sentiment"),
        pl.mean("first_call_resolution_indicator").alias("fcr_rate"),
        pl.mean("issue_resolution_status").alias("resolution_rate"),
        pl.mean("customer_frustration_level").alias("avg_frustration"),
    ])
    .collect()
)
```

**Expected Output**

A dataframe containing one row per agent with aggregated performance metrics.

**Error Handling**

If any required columns do not exist, inspect the schema and adjust the column names accordingly.

---

## Step 3: Convert to Pandas for Normalization

**Purpose**

Convert the Polars dataframe to Pandas for easier min-max normalization calculations.

**Tool / Action**

Use `python_interpreter`.

**Code / Query**

```python
agent_metrics_pd = agent_metrics.to_pandas()
```

**Expected Output**

A Pandas dataframe containing the same aggregated data, ready for normalization.

**Error Handling**

If the conversion fails, ensure that the Polars dataframe has been collected prior to conversion.

---

## Step 4: Calculate Normalized Scores and Composite Score

**Purpose**

Normalize all metrics to a 0–1 scale and calculate a weighted composite score to enable fair comparison between agents.

**Tool / Action**

Use `python_interpreter` with Pandas.

**Code / Query**

```text
Apply min-max normalization to each metric, invert frustration (lower is better), and combine into a weighted composite score:

- Empathy: 20%
- Script Adherence: 15%
- Customer Sentiment: 20%
- First Call Resolution: 20%
- Resolution Rate: 15%
- Low Frustration: 10%

Sort by composite score and return the top 10 agents.
```

**Expected Output**

The top 10 agents ranked by composite score, including all supporting metrics.

**Error Handling**

If division by zero occurs during normalization (for example, if all agents have identical values for a metric), assign a normalized value of `0.5`.

---

## Step 5: Collect Evidence Call GUIDs

**Purpose**

Retrieve sample call GUIDs for the highest-ranked agents to provide evidence supporting the rankings.

**Tool / Action**

Use `python_interpreter`.

**Code / Query**

```text
For each of the top-10 agent_ids, pull the first 3 call_guid values from the source dataframe into an evidence_calls dictionary.
```

**Expected Output**

A dictionary mapping each top-ranked agent to three example call GUIDs.

**Error Handling**

If an agent has fewer than three calls, include all available call GUIDs.

---

## Step 6: Create Visualization

**Purpose**

Generate a bar chart visualizing the composite scores of the top 10 agents.

**Tool / Action**

Use `python_interpreter` with Plotly.

**Code / Query**

```text
Build a Plotly Figure with a single Bar trace:

- x = agent_id
- y = composite_score

Apply layout settings including:
- chart title
- axis labels
- plotly_white template

Write the resulting chart to plot_top_agents.html.
```

**Expected Output**

An interactive HTML bar chart saved to file.

**Error Handling**

If Plotly is unavailable, skip visualization generation and continue with text-based output.

---

## Step 7: Format Final Response with Evidence Links

**Purpose**

Create a comprehensive markdown response containing rankings, metrics, evidence links, and supporting insights.

**Tool / Action**

Use `python_interpreter` with `final_answer`.

**Code / Query**

```text
Build a markdown ranking table from top_10.

Build per-agent evidence link blocks from evidence_calls.

Pass the assembled response string to final_answer().
```

**Expected Output**

A formatted markdown response containing:

- dynamic rankings,
- supporting metrics,
- evidence links, and
- analytical insights.

**Error Handling**

If any agent data is missing, exclude that agent from the final response rather than failing execution entirely.

---

# Data Validation Checks

The workflow included the following validation steps before generating the final output:

- Verify that at least one agent exists in the dataset.
- Confirm that all required metric columns are present.
- Check for null values in key performance metrics.
- Validate that composite scores fall within the expected `0–1` range.

---

# Edge Cases

The workflow also accounted for several edge conditions:

- If fewer than 10 agents exist, return all available agents.
- If all agents have identical values for a metric, ensure normalization handles the situation gracefully.
- If the `call_guid` column is unavailable, skip the evidence links section rather than failing execution.