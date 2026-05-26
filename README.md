# Network Topology and Systemic Risk: An Agent-Based Modeling Approach

This project studies how network topology affects the propagation of systemic risk in a stylized financial system.

The model is based on an agent-based simulation. Each agent is represented as a node in a network. Agents can receive an initial shock and then transmit financial stress to their neighbors over time. The main research question is whether different network structures lead to different cascade sizes, default levels, systemic-event probabilities, and crisis dynamics.

The project compares several artificial network topologies under the same approximate system size and target average degree. This makes the comparison focus on network structure rather than only on network density.

## Project Goal

The goal of the project is to analyze how financial contagion depends on the structure of the network.

More specifically, the project studies:

- whether some network topologies are more vulnerable to systemic cascades;
- how random shocks differ from targeted shocks to central agents;
- whether amplification mechanisms such as liquidity stress, leverage feedback, and herding change the results;
- whether the conclusions are robust to changes in system size, connectivity, and simulation horizon.

## Network Topologies

The project compares four network types.

### Erdős-Rényi Random Graph

The Erdős-Rényi graph is used as a random baseline. Edges are formed randomly between nodes with a probability calibrated to match the target average degree.

### Watts-Strogatz Small-World Graph

The Watts-Strogatz graph represents a small-world structure. It combines local clustering with relatively short paths between nodes.

### Barabási-Albert Scale-Free Graph

The Barabási-Albert graph represents a scale-free topology. It contains highly connected hubs that emerge through preferential attachment.

### Core-Middle-Periphery Graph

The core-middle-periphery graph is a stylized structure inspired by financial networks. It contains:

- a dense core;
- an intermediate middle layer;
- a sparse periphery.

This topology is generated as a stochastic block model with different connection probabilities between the layers. The block probabilities are calibrated so that the expected average degree is close to the target value.

## Model Description

Each node represents a financial agent.

Each agent has a stress level between 0 and 1. During the simulation, stress can increase because of shocks and stress received from connected neighbors. Stress can also decrease because of recovery.

Agents can be in one of three states:

1. `OK`
2. `Distressed`
3. `Defaulted`

Distress is reversible. This means that an agent can return from distress to the OK state if its stress level decreases. Default is absorbing. Once an agent defaults, it remains defaulted.

The simulation continues until either:

- the maximum time horizon is reached;
- or the system becomes stable according to the early-stopping rule.

## Initial Shocks

The model supports two types of initial shocks.

### Random Shock

A fixed fraction of nodes is selected randomly and receives the initial shock.

### Targeted Shock

The initial shock is applied to structurally central agents. In the code, these are selected as the highest-degree nodes.

This comparison helps test whether the same shock becomes more dangerous when it affects central agents.

## Scenarios

The project includes several scenarios.

### Baseline

In the baseline scenario, stress spreads through the network without additional amplification mechanisms.

### Liquidity Spiral

In this scenario, stress transmission becomes stronger and recovery becomes weaker during a specific crisis window.

### Leverage Feedback

In this scenario, highly stressed agents transmit more stress to their neighbors.

### Herding

In this scenario, an agent receives additional stress when a large enough share of its neighbors is already affected.

## Main Output Metrics

The simulation produces several main metrics.

### Cascade Metrics

- `cascade_size`: fraction of nodes that ever become distressed or defaulted.
- `final_affected`: final fraction of nodes that are distressed or defaulted.
- `peak_affected`: maximum affected fraction during the simulation.

### Default Metrics

- `final_default`: final fraction of defaulted nodes.
- `ever_defaulted`: fraction of nodes that ever defaulted.
- `peak_default`: maximum default fraction during the simulation.

### Systemic-Risk Indicators

- `systemic_by_default`: equals 1 if the final default fraction exceeds the systemic default threshold.
- `systemic_by_cascade`: equals 1 if the cascade size exceeds the systemic cascade threshold.

### Timing Metrics

- `time_to_peak_default`: time step at which the default fraction reaches its maximum.
- `time_to_peak_affected`: time step at which the affected fraction reaches its maximum.
- `time_to_50pct_peak_affected`: first time when affected fraction reaches 50% of its peak.
- `time_to_90pct_peak_affected`: first time when affected fraction reaches 90% of its peak.
- `active_cascade_duration_affected`: difference between the 90% and 50% peak times.
- `stabilization_time_affected`: first time after which the affected fraction remains stable.

### Cumulative Burden Metrics

- `area_affected`: area under the affected-fraction curve.
- `area_default`: area under the default-fraction curve.

These metrics capture not only the final size of the crisis, but also its speed, persistence, and cumulative severity.

## Code Structure

The code is organized into several main blocks.

### 1. Environment and Imports

The first cells install and import the required packages:

- `networkx`;
- `pandas`;
- `numpy`;
- `matplotlib`;
- `tqdm`;
- `scipy`;
- `statsmodels`.

The helper function `set_global_seed()` sets random seeds for reproducibility.

### 2. Graph Generation

The code defines several functions for generating network topologies.

#### `make_er()`

Generates an Erdős-Rényi random graph. The edge probability is chosen so that the expected average degree is close to the target value.

#### `make_ws()`

Generates a Watts-Strogatz small-world graph. The function adjusts the neighborhood size so that it is valid for the selected number of nodes.

#### `make_ba()`

Generates a Barabási-Albert scale-free graph. The parameter `m` is chosen based on the target average degree.

#### `expected_avg_degree_block_model()`

Calculates the expected average degree for an undirected stochastic block model.

#### `calibrate_block_probabilities()`

Scales the base block-affinity matrix so that the expected average degree of the block model is close to the target average degree.

#### `ensure_connected_by_bridges()`

Connects disconnected graph components by adding bridge edges. This keeps all nodes in the graph and makes the network connected.

#### `make_core_middle_periphery()`

Generates the stylized core-middle-periphery topology. Nodes are divided into core, middle, and periphery layers. The function also stores node-layer attributes and block probabilities.

#### `graph_summary()`

Calculates structural network statistics, including:

- number of nodes;
- number of edges;
- average degree;
- density;
- clustering coefficient;
- average shortest path length;
- minimum degree;
- maximum degree;
- 90th percentile of degree.

#### `build_topologies()`

Builds all four network topologies using the same number of nodes, target average degree, and graph seed.

### 3. Visualization

#### `plot_graph()`

Creates a quick visual representation of a graph using a spring layout. If the graph is large, the function can visualize only a sampled subset of nodes.

#### `cmp_layer_summary()`

Summarizes the degree distribution across the core, middle, and periphery layers of the CMP topology.

### 4. Simulation Configuration

#### `SimConfig`

`SimConfig` is a dataclass that stores all simulation parameters.

It includes:

- simulation horizon;
- early-stopping parameters;
- shock size;
- shock mode;
- shock strength;
- recovery rate;
- transmission strength;
- distress and default thresholds;
- systemic-event thresholds;
- liquidity spiral parameters;
- leverage feedback parameters;
- herding parameters;
- edge-weight settings.

This makes the experiments easier to reproduce and modify.

### 5. Shock Selection

#### `pick_shocked_nodes()`

Selects initially shocked nodes.

It supports two modes:

- `random`: selects shocked nodes randomly;
- `targeted`: selects nodes with the highest degree.

### 6. Core Simulation

#### `simulate_once()`

This is the main simulation function.

It runs one agent-based simulation on one fixed graph. The function:

1. draws individual distress and default thresholds;
2. initializes node stress levels;
3. applies the initial shock;
4. updates stress levels over time;
5. updates agent states;
6. applies optional liquidity, leverage, and herding mechanisms;
7. tracks affected and defaulted nodes;
8. stops early if the system becomes stable;
9. returns all main outcome metrics.

The function returns both final outcomes and time-series information.

### 7. Experiment Runners

#### `run_experiment()`

Runs multiple simulations on one fixed graph and stores both simulation outcomes and graph-level structural metrics.

#### `run_all_topologies()`

Runs the experiment for all topology types.

#### `run_all_topologies_over_graphs()`

Repeats the experiment across multiple independently generated graph realizations.

This is important because the results depend on both:

- graph-generation randomness;
- simulation randomness.

### 8. Scenario Configuration

#### `make_scenario_config()`

Creates scenario-specific versions of the baseline configuration.

It supports:

- `baseline`;
- `liquidity`;
- `leverage`;
- `herding`.

Each scenario changes only the relevant parameters while keeping the rest of the baseline setup consistent.

### 9. Summary Functions

#### `summarize_results()`

Aggregates simulation results by selected grouping columns, such as scenario, topology, or shock mode.

It calculates:

- number of runs;
- mean cascade size;
- median cascade size;
- standard deviation;
- mean final default;
- systemic-event probabilities;
- timing metrics;
- cumulative burden metrics;
- 95th percentile outcomes.

### 10. Statistical Tests

#### `holm_correction()`

Applies the Holm-Bonferroni correction to adjust p-values for multiple pairwise comparisons.

#### `run_tests_by_condition()`

Runs statistical tests for each condition.

For each scenario or shock mode, it performs:

1. Kruskal-Wallis test across topologies;
2. pairwise Mann-Whitney U tests;
3. Holm correction for multiple comparisons.

The code performs these tests on graph-level aggregated outcomes. This avoids treating repeated simulations on the same graph as fully independent observations.

### 11. Effect Sizes

#### `cliffs_delta()`

Calculates Cliff’s delta for pairwise comparisons between two groups.

#### `cliffs_delta_label()`

Gives a rough interpretation of Cliff’s delta:

- negligible;
- small;
- medium;
- large.

#### `pairwise_effect_sizes_by_condition()`

Calculates pairwise effect sizes for topology differences under each condition.

### 12. Bootstrap Confidence Intervals

#### `bootstrap_ci()`

Calculates bootstrap confidence intervals for a selected statistic, usually the mean.

#### `bootstrap_summary_table()`

Creates a summary table with:

- mean;
- median;
- standard deviation;
- 95% bootstrap confidence interval.

### 13. Chi-Square Tests

#### `chi_square_systemic_by_condition()`

Tests whether binary systemic-event probabilities differ across topologies.

It is used for binary indicators such as:

- `systemic_by_cascade`;
- `systemic_by_default`.

### 14. Robustness and Sensitivity Checks

The later parts of the code run additional experiments to check robustness.

These include:

- different system sizes;
- different average connectivity levels;
- different simulation time horizons;
- parameter grids for shock intensity and transmission strength.

## Experiments

### Baseline Experiment

The baseline experiment runs the model across all four topologies without additional amplification mechanisms.

Main output files:

- `results_baseline_multigraph.csv`;
- `summary_baseline_multigraph.csv`.

### Scenario Comparison

This experiment compares the following scenarios:

- baseline;
- liquidity;
- leverage;
- herding.

Main output files:

- `results_all_scenarios_multigraph.csv`;
- `summary_all_scenarios.csv`.

### Random vs Targeted Shock Comparison

This experiment compares random shocks with targeted shocks to high-degree nodes.

Main output files:

- `results_random_vs_targeted_multigraph.csv`;
- `summary_random_vs_targeted.csv`.

### Robustness Checks

This experiment checks whether the results are stable under changes in:

- system size;
- average connectivity.

Main output files:

- `results_robustness_size_connectivity.csv`;
- `summary_robustness_size_connectivity.csv`.

### Time-Horizon Sensitivity

This experiment checks whether the results are sensitive to the maximum simulation horizon.

Main output file:

- `summary_time_horizon_sensitivity.csv`.

### Parameter Grid

The parameter grid studies how different shock strengths and transmission strengths affect systemic outcomes.

## Statistical Analysis

The statistical analysis is based on graph-level aggregated outcomes.

This means that the code first averages simulation results within each graph realization and then applies statistical tests. This is done because repeated simulations on the same graph are not fully independent.

The project uses:

- Kruskal-Wallis tests;
- Mann-Whitney U tests;
- Holm correction;
- Cliff’s delta;
- bootstrap confidence intervals;
- chi-square tests for systemic-event indicators.

Generated statistical files include prefixes such as:

- `GRAPH_tests_kw_`;
- `GRAPH_tests_pairwise_`;
- `GRAPH_effect_sizes_`;
- `GRAPH_bootstrap_ci_`;
- `GRAPH_BINARY_chi_square_`;
- `ROBUST_kw_`;
- `ROBUST_pairwise_`.

## Requirements

The main packages used in the project are:

```bash
networkx==3.3
pandas
numpy
matplotlib
tqdm
scipy
statsmodels
