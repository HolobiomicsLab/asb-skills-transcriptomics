---
name: single-cell-rna-seq-simulation
description: Use when a new or existing rare-cell detection method needs quantitative
  benchmarking against competitors, and no real dataset with validated rare-cell ground-truth
  labels of controlled size and differential-expression level is available.
when_to_use_negative:
- When real experimental data with externally validated rare-cell identities (e.g.
  tuft cells in the intestine dataset marked by Dclk1/Trpm5) is available — biological
  realism matters more than controlled ground truth
- When the goal is to characterize detection limits only in one regime — a single
  Splatter setting is insufficient; the sweep over population size and de.prob is
  what makes conclusions general
- When the tool under test requires major cell types to be pre-specified and simulation
  scenarios cannot encode that annotation
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Splatter
  role: Generates simulated scRNA-seq expression profiles with planted rare populations
    and ground-truth labels, sweeping rare population size (2–100 cells) and de.prob
    (0.1–0.8)
- name: GapClust
  role: Rare-cell detector under test; called on each simulated dataset via its second-order-derivative-type
    distance score
  repo: github.com/fabotao/GapClust
- name: RaceID
  role: Competitor rare-cell detector run on the same simulated count matrices with
    default settings
- name: GiniClust
  role: Competitor rare-cell detector run on the same simulated count matrices with
    default settings
- name: CellSIUS
  role: Competitor rare-cell detector run on the same simulated count matrices with
    default settings
- name: FiRE
  role: Competitor rare-cell detector run on the same simulated count matrices with
    default settings
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:f8834564c8bdbf375d156ac01a2c4d4a2aa71daddd48878ef4fbd9b7101ce692
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellSIUS
  - FiRE
  - GapClust
  - GiniClust
  - RaceID
  - Splatter
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# single-cell-rna-seq-simulation

## Summary

Generate ground-truth single-cell RNA-seq benchmark data with Splatter by planting rare cell populations across controlled simulation parameters (rare population size 2–100 cells, de.prob 0.1–0.8), enabling rigorous F1-score comparison of rare-cell detection methods such as GapClust, RaceID, GiniClust, CellSIUS and FiRE.

## When to use

Use when a new or existing rare-cell detection method needs quantitative benchmarking against competitors, and no real dataset with validated rare-cell ground-truth labels of controlled size and differential-expression level is available.

## When NOT to use

- When real experimental data with externally validated rare-cell identities (e.g. tuft cells in the intestine dataset marked by Dclk1/Trpm5) is available — biological realism matters more than controlled ground truth
- When the goal is to characterize detection limits only in one regime — a single Splatter setting is insufficient; the sweep over population size and de.prob is what makes conclusions general
- When the tool under test requires major cell types to be pre-specified and simulation scenarios cannot encode that annotation

## Inputs

- Splatter simulation parameter grid: rare population size 2–100 cells, de.prob 0.1–0.8
- Simulated scRNA-seq count matrices with Splatter ground-truth rare-cell labels
- Rare-cell detection tool calls (GapClust, RaceID, GiniClust, CellSIUS, FiRE) with default settings

## Outputs

- 99 simulated expression profiles per DE-probability scenario with ground-truth rare-cell labels
- Per-dataset, per-method F1 scores for rare-cell detection
- Aggregated F1 tables/panels (Supplementary Fig. 2a-e style) ranking methods per simulation setting

## How to apply

Use Splatter in R to generate simulated expression profiles per differential-expression scenario: sweep the rare population size from 2 to 100 cells and the de.prob parameter from 0.1 to 0.8, producing 99 simulated datasets per DE-probability scenario while retaining Splatter's ground-truth rare-cell labels. Run the method under test (e.g. GapClust, which exploits a second-order-derivative-type distance score capturing the inter-cluster gap) on every simulated count matrix, then run each competitor (RaceID, GiniClust, CellSIUS, FiRE) on the same matrices with default settings. Compare each method's called rare cells against the ground-truth labels and compute per-dataset, per-method F1 scores (precision/recall over planted rare cells). Aggregate F1 scores into scenario tables/panels (matching Supplementary Fig. 2a-e) and check ranking consistency across settings — the benchmark's validity signal was that GapClust ranked best in every simulated setting. Rationale: simulation gives exact ground truth over a controlled parameter grid, isolating method sensitivity to rare-population size and DE level.

## Related tools

- **Splatter** (Generates simulated scRNA-seq expression profiles with planted rare populations and ground-truth labels, sweeping rare population size (2–100 cells) and de.prob (0.1–0.8))
- **GapClust** (Rare-cell detector under test; called on each simulated dataset via its second-order-derivative-type distance score) — github.com/fabotao/GapClust
- **RaceID** (Competitor rare-cell detector run on the same simulated count matrices with default settings)
- **GiniClust** (Competitor rare-cell detector run on the same simulated count matrices with default settings)
- **CellSIUS** (Competitor rare-cell detector run on the same simulated count matrices with default settings)
- **FiRE** (Competitor rare-cell detector run on the same simulated count matrices with default settings)

## Examples

```
library(Splatter); sim <- splatSimulate(params, group.prob = c(0.98, rare_prob), de.prob = 0.1); counts <- counts(sim); GapClust::GapClust(t(counts))
```

## Evaluation signals

- Each scenario yields 99 simulated datasets and every simulated dataset carries Splatter ground-truth rare-cell labels
- F1 score computed per dataset per method against ground truth; precision and recall derivable from called-vs-planted rare cell sets
- Aggregated F1 tables reproduce the structure of Supplementary Fig. 2a-e panels across all five simulation scenarios
- Ranking check: the method expected to win should rank best in every simulation setting (as GapClust did), otherwise investigate parameter misconfiguration
- Tool calls must use default settings for competitors to keep the comparison reproducible and fair

## Limitations

- Simulated profiles may not capture real-data characteristics (dropout, ambient noise, batch effects); results on real datasets were deferred to supplementary analyses and reported separately
- Missing version numbers for the compared tools (GiniClust, RaceID, CellSIUS, FiRE, EDGE) in the article's benchmark discussion limit exact reproducibility
- GapClust parameter settings (e.g. K, skewness threshold where 'Candidate k can be identified if the skewness<k> > 2') were not reported per benchmark run, complicating replication

## Evidence

- [results] Across the five Splatter simulation scenarios, GapClust provides the best rare cell detection performance among all compared methods in every simulation setting, as evaluated by F1 score.: [evidence span withheld — anchor status: not_found]
- [results] Splatter sweep workflow: [evidence span withheld — anchor status: not_found]
- [intro] GapClust method description: "A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information."
- [readme] In terms of rare cell detection, GapClust offers best performance compared with other methods, quantified by F1 score.: "In terms of rare cell detection, GapClust offers best performance compared with other methods, quantified by F<sub>1</sub> score."
- [intro] Existing tools limitation: "scRNA-seq datasets containing thousands of cells preclude the use of RaceID due to unrealistic amount of running time"
