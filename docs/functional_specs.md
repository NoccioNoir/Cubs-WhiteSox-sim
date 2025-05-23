# White Sox vs. Cubs Monte Carlo Simulation – Functional Specifications

This document details the core functions for the White Sox vs. Cubs Monte Carlo Simulation, organized into MVP iterations. Each section includes purpose, inputs, outputs, and implementation notes.

---

## MVP 1 – Core Monte Carlo Engine

### 1. `load_data(data_path)`

* **Purpose:** Ingest and validate batting and pitching CSVs for both teams.
* **Inputs:**

  * `data_path` (string): directory containing:

    * `cubs_standard_batting_clean.csv`
    * `whitesox_standard_batting_clean.csv`
    * `cubs_standard_pitching_clean.csv`
    * `whitesox_standard_pitching_clean.csv`
* **Outputs:**

  * Four Pandas DataFrames: `cubs_bat_df`, `ws_bat_df`, `cubs_pitch_df`, `ws_pitch_df`.
* **Notes:**

  * Raise descriptive errors if files are missing or columns are invalid.

### 2. `compute_batter_probs(df)`

* **Purpose:** Compute per‐plate‐appearance event probabilities for each batter.
* **Inputs:**

  * `df` (DataFrame): raw batting stats with columns `Player`, `PA`, `H`, `2B`, `3B`, `HR`, `BB`, `HBP`, `SO`.
* **Outputs:**

  * DataFrame indexed by `Player`, with probability columns `p_HR`, `p_3B`, `p_2B`, `p_1B`, `p_BB`, `p_HBP`, `p_SO`, `p_Out` (sums to 1).
* **Notes:**

  * Derive singles (`1B`) and “Out” as catch-all; verify normalization.

### 3. `compute_pitcher_probs(df)`

* **Purpose:** Compute per‐batters‐faced event probabilities for each pitcher.
* **Inputs:**

  * `df` (DataFrame): raw pitching stats with columns `Player`, `BF`, `HR`, `BB`, `HBP`, `SO`.
* **Outputs:**

  * DataFrame indexed by `Player`, with probability columns `p_HR`, `p_BB`, `p_HBP`, `p_SO`, `p_Out` (sums to 1).
* **Notes:**

  * Compute “Out” as `BF - (HR+BB+HBP+SO)`; verify normalization.

### 4. `simulate_pa(b_probs, p_probs=None, weight)`

* **Purpose:** Simulate a single plate appearance by blending batter and pitcher distributions.
* **Inputs:**

  * `b_probs` (Series): batter probabilities for events.
  * `p_probs` (Series, optional): pitcher probabilities for events.
  * `weight` (float 0–1): pitcher influence weight.
* **Outputs:**

  * String event: one of `{'HR','3B','2B','1B','BB','HBP','SO','Out'}`.
* **Notes:**

  * Default `p_probs=None` uses batter only.
  * Handle missing event keys by treating missing pitcher events as zero.

### 5. `simulate_half(lineup, pitcher_probs, batter_probs)`

* **Purpose:** Simulate one half-inning; manage outs, base-runners, and runs.
* **Inputs:**

  * `lineup` (list of Player IDs): batting order of nine players.
  * `pitcher_probs` (Series): probabilities for the current pitcher.
  * `batter_probs` (DataFrame): probabilities for all batters.
* **Outputs:**

  * Integer `runs` scored in the half-inning.
* **Notes:**

  * Maintain `outs` (0–3), `bases` (\[1B,2B,3B]), and update runs according to outcome logic.

### 6. `simulate_game(ws_lineup, cubs_lineup, ws_pitcher, cubs_pitcher, ws_pitch_probs, cubs_pitch_probs, cubs_bat_probs, ws_bat_probs)`

* **Purpose:** Simulate a full nine-inning game between the two teams.
* **Inputs:**

  * Batting orders and pitcher IDs as above.
  * Probability tables from prior functions.
* **Outputs:**

  * Tuple `(ws_runs, cubs_runs)` final score.
* **Notes:**

  * Alternate half-innings for nine frames; extras out of scope.

### 7. `monte_carlo(n_sims, **kwargs)`

* **Purpose:** Run many games to estimate White Sox win probability.
* **Inputs:**

  * `n_sims` (int): number of simulated games.
  * Keyword args matching `simulate_game` signature.
* **Outputs:**

  * Float: fraction of games where `ws_runs > cubs_runs`.
* **Notes:**

  * Ensure performance tuning (vectorize if needed) so 10,000 sims <= 2 minutes.

---

## MVP 2 – Analysis & Visualization

* **Plot Run-Differential Histogram**

  * **Function:** `plot_run_diff(simulation_results)`
  * **Purpose:** Visualize distribution of score margins.
  * **Inputs:** array of `(ws_runs - cubs_runs)` from many sims.
  * **Outputs:** Matplotlib histogram.
* **Export Summary CSV**

  * **Function:** `export_summary(filename, win_prob, stats)`
  * **Purpose:** Save `win_prob` and other summary statistics.
  * **Inputs:** output path, computed metrics.
  * **Outputs:** `.csv` file.

---

## MVP 3 – Roster & Schedule Integration

* **Enforce Defensive Constraints**

  * Use `*_roster_clean.csv` to ensure one player per position in lineup.
* **Filter Head-to-Head Dates**

  * Parse `2025_Cubs_schedule.csv` and `2025_CWS_schedule.csv` to select matchups.

---

## MVP 4 – Extended Features & Automation

* **Error Modeling**

  * Integrate defensive error probabilities derived from fielding metrics.
* **Auto-Fetch Updated Stats**

  * Function: `fetch_latest_stats(team)` to pull from online APIs.
* **Scenario Management**

  * Save/load simulation configurations (`save_simulation`, `load_simulation`).

---

*End of Functional Specifications*
