---
name: local-structure-preservation-rnn-evaluation
description: Reference for judging integration quality when a method must both mix
  batches (iLISI) and retain local neighborhood structure (50-RNN), e.g. benchmarking
  BATMAN, Seurat V3.0, MNNcorrect, or Scanorama.
when_to_use_negative:
- When the integration pipeline only has more than two datasets to correct — BATMAN's
  current implementation supports only two datasets, so the metrics still apply but
  the BATMAN method being evaluated does not.
- When the goal is only mixing without regard to local structure (iLISI alone) — this
  skill exists specifically to catch over-correction that destroys cell-to-cell relationships,
  as seen with Scanorama's 50-RNN of 0.14.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BATMAN
  role: Integrator whose output is evaluated with mean iLISI and mean 50-RNN; achieves
    highest iLISI (1.84) while best preserving cell-to-cell relationships (50-RNN
    0.92) in the LB scenario
  repo: https://github.com/mandricigor/batman
- name: Seurat
  role: Provides the V3.0 pre-processing that produces the log-counts fed to all integrators
    before metric computation; also a comparison integrator
- name: Scanorama
  role: Comparison integrator evaluated with the same iLISI/50-RNN metrics (iLISI
    1.01, 50-RNN 0.14)
- name: MNNcorrect
  role: Comparison integrator evaluated with the same iLISI/50-RNN metrics (iLISI
    1.00, 50-RNN 0.51)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:3a121bff2ab478752dcd9c67690270fe655ec00756f9ac37f0d3ce0df122969b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mandricigor/batman
  repo_url: https://github.com/mandricigor/batman
  tools:
  - BATMAN
  - MNNcorrect
  - Scanorama
  - Seurat
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Local structure preservation RNN evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates whether a single-cell RNA-seq batch-integration method preserves cell-to-cell (local) relationships while correcting batch effects, using the mean 50-RNN (retain-nearest-neighbor) metric reported alongside mean iLISI in the BATMAN benchmark.

## When to use

Reference for judging integration quality when a method must both mix batches (iLISI) and retain local neighborhood structure (50-RNN), e.g. benchmarking BATMAN, Seurat V3.0, MNNcorrect, or Scanorama.

## When NOT to use

- When the integration pipeline only has more than two datasets to correct — BATMAN's current implementation supports only two datasets, so the metrics still apply but the BATMAN method being evaluated does not.
- When the goal is only mixing without regard to local structure (iLISI alone) — this skill exists specifically to catch over-correction that destroys cell-to-cell relationships, as seen with Scanorama's 50-RNN of 0.14.

## Inputs

- Integrated (batch-corrected) cell-by-gene log-count matrices from each integrator
- Original (uncorrected) concatenated log-count matrices as the structure-preservation baseline
- Batch/dataset labels per cell (e.g. PBMC 8k vs Pan T cells) for iLISI computation

## Outputs

- Per-method mean 50-RNN and mean iLISI values (plus wall-clock runtime) in a CSV with one row per method (Original, BATMAN, Seurat V3.0, MNNcorrect, Scanorama)

## How to apply

After running each integrator (e.g. BATMAN, Seurat V3.0, MNNcorrect, Scanorama) on Seurat V3.0 pre-processed log-count matrices, compute the mean 50-RNN on the integrated output to quantify retention of cell-to-cell relationships, and compute mean iLISI to quantify batch mixing. Interpret the two metrics together: a method that maximizes iLISI but collapses 50-RNN is over-correcting local structure. In the LB scenario of the PBMC 8k (8,381 cells) vs Pan T cells (3,555 cells) benchmark, BATMAN reached mean iLISI 1.84 with mean 50-RNN 0.92, versus 50-RNN of 0.58 (Seurat V3.0), 0.51 (MNNcorrect), and 0.14 (Scanorama), with original data as the preservation reference (iLISI 1.82, 50-RNN 1.00). Record these metrics plus wall-clock runtime per method in a CSV with one row per method to support the comparison.

## Related tools

- **BATMAN** (Integrator whose output is evaluated with mean iLISI and mean 50-RNN; achieves highest iLISI (1.84) while best preserving cell-to-cell relationships (50-RNN 0.92) in the LB scenario) — https://github.com/mandricigor/batman
- **Seurat** (Provides the V3.0 pre-processing that produces the log-counts fed to all integrators before metric computation; also a comparison integrator)
- **Scanorama** (Comparison integrator evaluated with the same iLISI/50-RNN metrics (iLISI 1.01, 50-RNN 0.14))
- **MNNcorrect** (Comparison integrator evaluated with the same iLISI/50-RNN metrics (iLISI 1.00, 50-RNN 0.51))

## Examples

```
python3.7 batman.py -data pbmc8k.csv,panT.csv -output batman_integrated.csv -anchors 100; # then compute mean iLISI and mean 50-RNN on the output vs original log-counts
```

## Evaluation signals

- The benchmark CSV contains exactly 5 rows (Original, BATMAN, Seurat V3.0, MNNcorrect, Scanorama) including a BATMAN row.
- LB-scenario values match the reported reference: mean iLISI 1.84 and mean 50-RNN 0.92 for BATMAN, with iLISI ≤ 1.01 for the other methods.
- Original (uncorrected) data serves as the structure baseline with 50-RNN = 1.00; any integrator's 50-RNN is interpreted as retention relative to this baseline.
- Reported runtime ordering is reproduced: BATMAN 20.1 s < Scanorama 31.0 s < Seurat V3.0 211.2 s < MNNcorrect 1,099.5 s.
- A method with high iLISI but low 50-RNN (e.g. Scanorama: iLISI 1.01, 50-RNN 0.14) is flagged as failing local structure preservation.

## Limitations

- iLISI and 50-RNN must be read jointly: a method can maximize batch mixing while largely destroying local structure, as Scanorama did (iLISI 1.01 but 50-RNN 0.14 in the LB scenario).
- The metrics were reported for two-dataset integration scenarios (e.g. LB); BATMAN itself currently supports only two datasets, so multi-dataset evaluation is out of scope.
- 50-RNN depends on the pre-processing choice; in all experiments Seurat V3.0 pre-processing was used to obtain log-counts, so changing normalization changes metric comparability.

## Evidence
