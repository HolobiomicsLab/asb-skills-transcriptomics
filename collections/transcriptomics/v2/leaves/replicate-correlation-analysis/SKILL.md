---
name: replicate-correlation-analysis
description: Use when two biological replicates each yield a ranked list of perturbation
  scores (e.g. 180 active TFs from systematic KO simulations) and you must show the
  ranking is reproducible before nominating candidate regulators.
when_to_use_negative:
- Only one replicate is available, so no cross-replicate agreement can be estimated
- Scores are not perturbation simulation outputs (e.g. raw gene expression matrices),
  where aggregation of negative perturbation scores across Monocle states is not meaningful
- The comparison goal is differential expression between conditions rather than reproducibility
  of a ranked TF perturbation list
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CellOracle
  role: Runs the systematic in silico TF knockout simulations on state-specific GRN
    (Links) models whose perturbation scores are aggregated and correlated between
    replicates
  repo: https://github.com/morris-lab/CellOracle
- name: Python
  role: Implementation environment for aggregating per-state perturbation scores,
    computing the Pearson correlation, and rendering the replicate scatter plot
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:d2013bc8752a00f5734be47327f8a7f48aa31ab7b9ef08b4ed321c5f48d25958
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/morris-lab/CellOracle
  repo_url: https://github.com/morris-lab/CellOracle
  tools:
  - CellOracle
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# replicate-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify reproducibility of CellOracle in silico TF knockout perturbation rankings across independent biological replicates of MEF-to-iEP reprogramming by computing Pearson correlation between replicate-level summed KO scores, plotted as a replicate-agreement scatter (Figure 3D).

## When to use

Use when two biological replicates each yield a ranked list of perturbation scores (e.g. 180 active TFs from systematic KO simulations) and you must show the ranking is reproducible before nominating candidate regulators.

## When NOT to use

- Only one replicate is available, so no cross-replicate agreement can be estimated
- Scores are not perturbation simulation outputs (e.g. raw gene expression matrices), where aggregation of negative perturbation scores across Monocle states is not meaningful
- The comparison goal is differential expression between conditions rather than reproducibility of a ranked TF perturbation list

## Inputs

- Per-replicate TF knockout perturbation scores from CellOracle in silico KO simulations across seven Monocle states (two iEP reprogramming replicates)
- Ranked TF tables (e.g. tf_ko_perturbation_ranking.csv) per replicate
- Processed scRNA-seq datasets annotated with Monocle states (48,515 cells total)

## Outputs

- Pearson correlation coefficient between replicate TF KO scores/rankings (expected r = 0.72)
- Scatter plot of replicate-by-replicate score agreement
- Ranked table of 180 TFs by summed negative perturbation score (tf_ko_perturbation_ranking.csv)

## How to apply

Load processed scRNA-seq Oracle objects for each iEP reprogramming replicate (48,515 cells total, annotated with seven Monocle states) and, per replicate, aggregate each TF's simulated perturbation scores by summing negative perturbation scores across the seven Monocle states into one total KO score per TF. Rank the 180 active TFs by this summed negative perturbation score (e.g. write tf_ko_perturbation_ranking.csv). Then compute the Pearson correlation between the two replicates' TF scores/rankings and render a scatter plot of the agreement. Validate the result biologically: the Hnf4α-Foxa1 transgene components should occupy the top rank since these factors drive reprogramming, and the replicate correlation should reach the reported agreement of r = 0.72. A high replicate agreement (r ~ 0.72) supports downstream nomination of novel regulators such as Id1, Fosb, Fos, Eno1, and Klf4.

## Related tools

- **CellOracle** (Runs the systematic in silico TF knockout simulations on state-specific GRN (Links) models whose perturbation scores are aggregated and correlated between replicates) — https://github.com/morris-lab/CellOracle
- **Python** (Implementation environment for aggregating per-state perturbation scores, computing the Pearson correlation, and rendering the replicate scatter plot)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(rep1_scores, rep2_scores)
```

## Evaluation signals

- Pearson correlation between the two replicates' TF KO scores/rankings reaches the reported r = 0.72
- Hnf4α-Foxa1 transgene components occupy the top of the TF ranking (rank 1), as expected for the reprogramming drivers
- Ranking table contains all 180 active TFs with one summed score per TF per replicate (sum of negative perturbation scores across seven Monocle states)
- Output file tf_ko_perturbation_ranking.csv is written with a consistent ranked ordering
- Scatter plot renders both replicates' scores on comparable axes showing rank agreement (Figure 3D)

## Limitations

- The replicate agreement r = 0.72 indicates rank agreement but not perfect reproducibility; top-ranked shared TFs between independent replicates are the robust candidates
- Scores depend on state-specific GRN inference, so variation in GRN inference quality per Monocle state propagates into KO scores
- Aggregation by summing negative perturbation scores across the seven Monocle states assumes a total effect on reprogramming efficiency, which is a modeling choice rather than a directly measured phenotype

## Evidence
