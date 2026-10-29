---
name: synthetic-ground-truth-benchmark-scoring
description: Use when evaluating a GRN-inference or multi-omics integration method
  on MOSim-simulated bulk or single-cell data where the simulated active regulatory
  relationships serve as ground truth for benchmarking.
when_to_use_negative:
- When the data are real experimental measurements with no simulated ground truth
  — true positive/negative regulatory sets do not exist and this scoring scheme is
  undefined.
- When the omics layers are outside MOSim's scope, e.g. proteomics, metabolomics,
  or spatially resolved data, since MOSim cannot simulate GRNs with spatial resolution.
- When the goal is single-cell evaluation — use Kendall's Tb correlation rather than
  Pearson, per the article's methods.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3170
tools:
- name: MOSim
  role: Generates the simulated multi-omics data and the active regulatory relationships
    used as benchmarking ground truth
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Runtime for MOSim simulation, NOISeq preprocessing, voom/limma DEG calling,
    MORE inference, and metric computation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:1bb62ea169cc3465891770957f4cc46e6143f38116964f15680291db1ead23f4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# synthetic-ground-truth-benchmark-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores GRN-inference output against ground truth from MOSim-simulated multi-omics data by comparing inferred gene–regulator regulations to simulated active regulations and to random feature pairs, computing sensitivity and F1-score. This is the article's prescribed way to quantitatively validate multi-omics regulatory inference methods on synthetic data.

## When to use

Use when evaluating a GRN-inference or multi-omics integration method on MOSim-simulated bulk or single-cell data where the simulated active regulatory relationships serve as ground truth for benchmarking.

## When NOT to use

- When the data are real experimental measurements with no simulated ground truth — true positive/negative regulatory sets do not exist and this scoring scheme is undefined.
- When the omics layers are outside MOSim's scope, e.g. proteomics, metabolomics, or spatially resolved data, since MOSim cannot simulate GRNs with spatial resolution.
- When the goal is single-cell evaluation — use Kendall's Tb correlation rather than Pearson, per the article's methods.

## Inputs

- MOSim-simulated bulk RNA-seq, miRNA-seq and TF count matrices with simulated active regulatory ground truth
- Inferred gene–regulator interaction list from the GRN-inference tool under test (e.g. MORE PLS1 Jack-Knife output: 370,566 potential interactions)
- Preprocessed/normalized expression matrices (NOISeq filtering, voom transformation)

## Outputs

- Sensitivity and F1-score of the inferred regulations vs simulated ground truth
- Comparison of gene–regulator correlations vs 20,000 random feature pairs with no simulated regulatory effect
- Metrics CSV (e.g. more_evaluation_metrics.csv) with DEG count, fitted model count, potential/significant interaction counts, sensitivity, F1

## How to apply

After running the inference tool on MOSim-simulated layers (e.g. RNA-seq with miRNA-seq and TF regulators), extract the inferred gene–regulator interactions and compare them against the active regulatory relationships MOSim reports as ground truth. For bulk data compute Pearson's correlation (and Kendall's Tb for single-cell) for gene–regulator pairs, and compare these correlations against 20,000 random feature pairs with no simulated regulatory effects to define the null. From the true/false positive assignments compute sensitivity and F1-score — in the MORE benchmark on MOSim data this yielded 85.5% sensitivity and 62.9% F1. Write the reproduced metrics (DEG count, fitted models, potential/significant interactions, sensitivity, F1) to a CSV such as more_evaluation_metrics.csv.

## Related tools

- **MOSim** (Generates the simulated multi-omics data and the active regulatory relationships used as benchmarking ground truth) — github.com/ConesaLab/MOSim
- **R** (Runtime for MOSim simulation, NOISeq preprocessing, voom/limma DEG calling, MORE inference, and metric computation)

## Examples

```
Rscript more_benchmark.R --sim MOSim --seed fixed --reads 30M --deg 0.5 --sigreg 0.6 --out more_evaluation_metrics.csv
```

## Evaluation signals

- Sensitivity and F1-score fall in a plausible range; reference run of MORE on MOSim bulk RNA-seq/miRNA-seq/TF data achieved 85.5% sensitivity and 62.9% F1.
- Random-pair null contains exactly 20,000 feature pairs with no simulated regulatory effect, matching the article's protocol.
- Downstream counts match the reference workflow: 10,593 DEGs, 5,573 fitted MORE models, 370,566 potential interactions (174,051 and 174,067 significant in groups 1 and 2).
- Metrics file (more_evaluation_metrics.csv) is produced with DEG count, fitted models, potential/significant interactions, sensitivity, and F1 fields populated.
- Correlation validation uses Pearson for bulk data and Kendall's Tb for single-cell data, consistent with the article's methods.

## Limitations

- Synthetic ground truth only covers simulated regulatory layers; only three methods supported comprehensive bulk multi-omics regulation simulation before MOSim, and none reported feature interactions or full customization.
- MOSim cannot simulate GRNs with spatial resolution and does not cover proteomics or metabolomics layers.
- Single-cell simulation is restricted to scRNA-seq and scATAC-seq, limiting which single-cell GRN tools can be benchmarked this way.
- Benchmark metrics are conditional on simulation settings (fixed seed, 30M reads, two groups, 20 time points, one replicate, 50% DEGs, 60% significant regulations); other designs may shift sensitivity/F1.

## Evidence
