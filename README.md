# RHOAIENG Observability Deployment View Dashboard

Observability dashboard providing a high-level overview of VLLM model deployment health, including request volume, token throughput, and latency statistics.

## Table of Contents

1. [Overview](#1-overview)
2. [Audience](#2-audience)
3. [Key Features & Panels](#3-key-features--panels)
4. [Data Source Requirements](#4-data-source-requirements)
5. [Dashboard Variables (Filters)](#5-dashboard-variables-filters)
6. [Known Limitations & Troubleshooting](#6-known-limitations--troubleshooting)
7. [How to Import](#7-how-to-import)

## 1. Overview

This dashboard provides a high-level overview of VLLM model deployment behavior and key performance indicators. It is designed to help users quickly assess deployment health and identify trends in request activity, token usage, and latency.

This is the "Deployment Overview Dashboard - Level 0" Grafana dashboard, specifically designed for monitoring VLLM (Very Large Language Model) deployments.

## 2. Audience

**Primary Audience:** Data Scientists, Product Managers

## 3. Key Features & Panels

The dashboard is organized into logical rows and includes the following key visualizations:

### High-Level Filters & Selections

- **Deployment ID:** Allows filtering data by specific VLLM model deployments (e.g., `granite-33-2b-instruct`)
- **Time Frame:** Standard Grafana time picker for selecting historical periods (1D, 1M, 1Y, All). Default set to `now/d+10h` to `now/d+16h` (10 AM - 4 PM local time)
- **Rush Hours Type:** A custom toggle for filtering data by specific time windows:
  - **Static:** 10 AM - 4 PM weekdays
  - **Dynamic:** Placeholder for future custom logic

### Query Statistics

#### Requests Over Time (Time Series Graph)
- **Description:** Visualizes the rate of successful requests per second over time, along with an average rate summary
- **Chart Type:** Time series
- **X-Axis:** Time
- **Y-Axis:** Requests/sec (Volume)
- **Extra Stats:**
  - **Avg:** Displays the average requests per second over the selected time frame
  - **Note:** Uses a workaround query due to Prometheus parser limitations
  - **p50, p90, p99:** Not implementable directly for requests/sec rate due to Prometheus parser limitations and metric type
- **Filtering:** Reacts to Deployment ID. Dynamic "Rush Hours Type" filtering is not fully operational for this panel

#### E2E Latency Statistics (Stat Panels)
- **Description:** Provides key percentile statistics for End-to-End Request Latency
- **Units:** milliseconds (ms)
- **Extra Stats:**
  - **p50 Latency:** Shows the median (50th percentile) latency
  - **p90 Latency:** Shows the 90th percentile latency
  - **p99 Latency:** Shows the 99th percentile latency

#### Input Token Size Distribution (Histogram Graph)
- **Description:** Visualizes the distribution of input (prompt) token lengths per request, along with average and percentile summaries of token size
- **Chart Type:** Histogram
- **X-Axis:** Token Count (Buckets)
- **Y-Axis:** counts/sec (cps)
- **Extra Stats:**
  - **Avg:** Displays the average number of input tokens per successful request
  - **p50, p90, p99:** Shows the 50th, 90th, and 99th percentiles of Input Token Size
- **Filtering:** Reacts to Deployment ID. Dynamic "Rush Hours Type" filtering is not fully operational for this panel

#### Input Tokens Over Time (Time Series Graph)
- **Description:** Visualizes the rate of input tokens processed per second over time, with an average rate summary
- **Chart Type:** Time series
- **X-Axis:** Time
- **Y-Axis:** counts/sec (Volume)
- **Extra Stats:**
  - **Avg:** Displays the average rate of input tokens per second over the selected time frame
  - **Note:** Uses a workaround query due to Prometheus parser limitations
  - **p50, p90, p99:** Not implementable directly for tokens/sec rate due to Prometheus parser limitations and metric type
- **Note:** Does not include Rush Hours Type filtering in PromQL

#### Output Tokens Over Time (Time Series Graph)
- **Description:** Visualizes the rate of output tokens generated per second over time, with an average rate summary
- **Chart Type:** Time series
- **X-Axis:** Time
- **Y-Axis:** counts/sec (Volume)
- **Extra Stats:**
  - **Avg:** Displays the average rate of output tokens per second over the selected time frame
  - **Note:** Uses a workaround query due to Prometheus parser limitations
  - **p50, p90, p99:** Not implementable directly for tokens/sec rate due to Prometheus parser limitations and metric type
- **Note:** Does not include Rush Hours Type filtering in PromQL

### Performance Statistics

**High-level Selection:** This section will feature a high-level selection for Metric (e.g., TTFT, ITL, E2E latency, TPS) and Aggregation (average, p90, p99, max), allowing users to dynamically choose which performance aspect and aggregation method to display.

**Note on Implementation:** These graphs are currently planned. Due to ongoing challenges with Prometheus PromQL parsing for complex time-based filtering and aggregations, dynamic "Rush Hours Type" filtering is not implemented directly within their queries. Users may need to adjust the dashboard's global time range manually to focus on specific periods.

#### E2E Latency Over Time (Time Series Graph)
- **Description:** Plots the End-to-End Request Latency over time
- **Chart Type:** Time series
- **X-Axis:** Time
- **Y-Axis:** Latency (seconds/milliseconds)
- **Extra Stats:** Avg, p50, p90, p99 (These will represent percentiles of latency, not rate. The Avg will be the average latency)
- **Note:** Trend line not available

#### TTFT Over Time (Time Series Graph)
- **Description:** Plots the Time To First Token (TTFT) over time
- **Chart Type:** Time series
- **X-Axis:** Time
- **Y-Axis:** Time (seconds/milliseconds)
- **Extra Stats:** Avg, p50, p90, p99 (These will represent percentiles of TTFT, not rate. The Avg will be the average TTFT)
- **Note:** Trend line not available

#### ITL Over Time (Time Series Graph)
- **Description:** Plots the Inter-Token Latency (ITL, or time per output token) over time
- **Chart Type:** Time series
- **X-Axis:** Time
- **Y-Axis:** Time (seconds/milliseconds)
- **Extra Stats:** Avg, p50, p90, p99 (These will represent percentiles of ITL, not rate. The Avg will be the average ITL)
- **Note:** Trend line not available

#### TPS (Tokens Per Second) (Time Series Graph)
- **Description:** Visualizes the overall throughput of tokens (prompt + generation) processed per second
- **Chart Type:** Time series
- **X-Axis:** Time
- **Y-Axis:** Tokens/sec (Volume)
- **Extra Stats:** Avg, p50, p90, p99 (These percentiles of rate are not implementable directly due to Prometheus parser limitations and metric type. The Avg will use a workaround)
- **Note:** Trend line not available

## 4. Data Source Requirements

This dashboard is designed to query metrics from a Prometheus data source. The Prometheus instance is expected to be collecting metrics from your VLLM deployments, specifically:

- `vllm:request_success_total` (Counter)
- `vllm:e2e_request_latency_seconds_bucket` (Histogram)
- `vllm:request_prompt_tokens_bucket` (Histogram)
- `vllm:prompt_tokens_total` (Counter)
- `vllm:generation_tokens_total` (Counter)
- `vllm:time_to_first_token_seconds_bucket` (Histogram)
- `vllm:time_per_output_token_seconds_bucket` (Histogram)
- `vllm:tokens_total` (Counter for TPS)

These metrics are expected to have a `model_name` label (e.g., `model_name="granite-33-2b-instruct"`) for filtering.

## 5. Dashboard Variables (Filters)

### Deployment_id
- **Type:** Custom (workaround due to Prometheus `label_values()` function not being supported)
- **Values:** Manually listed model_names (e.g., `granite-33-2b-instruct`)
- **Note:** Requires manual update if new models are deployed

### rush_hours
- **Status:** Not actively used for filtering due to `rush_hours_type` variable taking precedence, but present
- **Note:** Due to limitations, `rush_hours` and 'Rush Hours Type' is not yet implemented

### rush_hours_type
- **Type:** Custom
- **Values:**
  - `^All__.*$` : All
  - `^Static__.*$` : Static
  - `^Dynamic__.*$` : Dynamic
- **Functionality:** Controls filtering for "All hours", "Static hours" (10 AM - 4 PM UTC-adjusted weekdays), or "Dynamic" (placeholder) via Grafana transformations

## 6. Known Troubleshooting

This dashboard has been built against a Prometheus environment with specific limitations that may cause "No data" or unexpected behavior if not understood:

### Missing Metrics
The most common reason for "No data" is that the required `vllm:` metrics (e.g., `vllm:request_success_total`, `vllm:prompt_tokens_total`) are not currently available/collected in your Prometheus instance. This must be resolved by the VLLM deployment team or Prometheus administrator.


### Troubleshooting "No data"

1. **Check Raw Metric:** First, verify the raw metric exists in OpenShift Console (Observe > Metrics) over a wide time range (e.g., `vllm:request_success_total`)
2. **Check Variable:** Ensure Deployment_id dropdown shows actual model names
3. **Check Time Range:** Set dashboard to "Last 7 days" or "Last 24 hours"
4. **Confirm "Code" Mode:** All queries in panels MUST be in "Code" mode

## 7. How to Import

1. In Grafana, go to **Dashboards** (left sidebar)
2. Click **"Import"**
3. Paste the entire JSON content of this dashboard into the **"Import via panel json"** text area
4. Click **"Load"**
5. Select your Prometheus data source
6. Click **"Import"**



