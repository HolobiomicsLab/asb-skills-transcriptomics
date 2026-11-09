---
name: ground-truth-grn-validation
description: Decision support for ranking gene regulatory network inference methods
  when synthetic single-cell data with an imposed causal ground-truth GRN is available,
  or when reproducing BEELINE-style benchmark comparisons.
when_to_use_negative:
- Evaluating GRN inference on real experimental scRNA-seq data without a reliable
  imposed ground truth, since curated databases are not ideal benchmarks due to context
  specificity of gene regulation
- Benchmarking on multi-cell-type datasets such as PBMC-All, where the article states
  results are not conclusive and follow-up studies are required
- Evaluating a cell-type-specific method like CeSpGRN in isolation, since no comprehensive
  evaluation of this method class from different perspectives has been performed
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: GRouNdGAN
  role: Generates the scRNA-seq simulation data with an imposed causal ground-truth
    GRN and provides the --benchmark_grn command that evaluates reconstructed GRNs
    against the ground truth
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers regulatory edges from the reference dataset to define the imposed ground-truth
    GRN (each gene regulated by its top 15 identified TFs) before benchmarking
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: BEELINE
  role: Reference benchmark framework whose curated experimental-benchmark method
    ordering is the external validation target for the GRouNdGAN-based ranking
  repo: https://github.com/Murali-group/Beeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:aebd58e81b1e899b8d4ab51eb7b5fca8cd91a339775ff93717937fc36c3ace47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - BEELINE
  - GRNBoost2
  - GRouNdGAN
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# ground-truth-grn-validation

## Summary

Validates GRN inference methods against an imposed ground-truth GRN using GRouNdGAN's benchmarking command, computing per-gene precision-at-k (k = 15, matching the top-15 GRNBoost2-derived regulators per gene) and a global precision-recall curve with AUPRC against a random-predictor baseline, then ranks methods (e.g., PIDC > GENIE3 > GRNBoost2) for comparison with BEELINE's curated-benchmark ordering.

## When to use

Decision support for ranking gene regulatory network inference methods when synthetic single-cell data with an imposed causal ground-truth GRN is available, or when reproducing BEELINE-style benchmark comparisons.

## When NOT to use

- Evaluating GRN inference on real experimental scRNA-seq data without a reliable imposed ground truth, since curated databases are not ideal benchmarks due to context specificity of gene regulation
- Benchmarking on multi-cell-type datasets such as PBMC-All, where the article states results are not conclusive and follow-up studies are required
- Evaluating a cell-type-specific method like CeSpGRN in isolation, since no comprehensive evaluation of this method class from different perspectives has been performed

## Inputs

- GRouNdGAN-simulated single-cell RNA-seq expression matrix (~100k cells x 1000 genes, library-size normalized)
- Inferred GRN files from each method as tab-separated CSV with columns TF, Gene, importance (headers optional)
- Exported ground-truth GRN CSV (set via [GRN Benchmarking]/ground truth save path)

## Outputs

- Per-method precision-at-k values (k = 15) per gene
- Global precision-recall curve with AUPRC and random-predictor baseline AUPRC per method per dataset
- Aggregated method ranking table (ranking_table.csv)

## How to apply

Generate or obtain GRouNdGAN-simulated scRNA-seq data (e.g., ~100k cells x 1000 genes from BoneMarrow/PBMC references) whose causal generating graph — each gene regulated by its top 15 GRNBoost2-identified TFs — serves as ground truth. Run each candidate inference method (GRNBoost2, GENIE3, PIDC, PPCOR, LEAP, SCODE, SINCERITIES, CeSpGRN) on the simulated expression matrices and format every inferred network as a tab-separated CSV with columns in the order TF, Gene, importance (headers optional). Export the ground-truth GRN to CSV via the [GRN Benchmarking]/ground truth save path setting, then run GRouNdGAN's --benchmark_grn command to evaluate each inferred network against it. Compute precision-at-k per gene with k = 15 and a global precision-recall curve with AUPRC including the random-predictor baseline; aggregate into a ranking table (ranking_table.csv). Correct application is confirmed when the ranking aligns with BEELINE's curated experimental benchmarks (PIDC first, only LEAP/SCODE order swapped).

## Related tools

- **GRouNdGAN** (Generates the scRNA-seq simulation data with an imposed causal ground-truth GRN and provides the --benchmark_grn command that evaluates reconstructed GRNs against the ground truth) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers regulatory edges from the reference dataset to define the imposed ground-truth GRN (each gene regulated by its top 15 identified TFs) before benchmarking) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **BEELINE** (Reference benchmark framework whose curated experimental-benchmark method ordering is the external validation target for the GRouNdGAN-based ranking) — https://github.com/Murali-group/Beeline

## Examples

```
docker pull yazdanz/groundgan:4b98686
```

## Evaluation signals

- PIDC ranks first among all eight benchmarked methods on both BoneMarrow and PBMC datasets (followed by GENIE3 and GRNBoost2)
- Method ordering on GRouNdGAN-simulated data matches BEELINE's curated-benchmark results with only LEAP and SCODE ranks swapped
- Each method's AUPRC exceeds the random-predictor baseline AUPRC reported in the PR curve
- All inferred networks parse as tab-separated CSVs with columns ordered TF, Gene, importance and are accepted by the --benchmark_grn evaluator
- Per-gene precision-at-k values use k = 15, consistent with the top-15 GRNBoost2-derived regulators per gene in the imposed GRN

## Limitations

- GRouNdGAN only supports bipartite GRNs; multi-layer GRNs and TF-gene-phenotype causal graphs cannot be imposed or validated
- The imposed causal graph of an observational dataset is not necessarily unique; verifying the imposed GRN is the most probable one is beyond the study's scope
- GRouNdGAN generates library-size normalized expression values, not unnormalized counts, which constrains the simulated data used for benchmarking
- Resemblance to the reference deteriorates if imposed TF-gene relationships differ significantly from reference data patterns, weakening benchmark fidelity
- Model is generative, not predictive, and must be retrained for each new dataset before benchmarking

## Evidence

- [methods] --benchmark_grn evaluates inferred GRN against exported ground truth: "--benchmark_grn  Evaluate the performance of a GRN inference method in inferring the ground truth GRN"
- [methods] Benchmark command compares reconstructed GRN to simulation ground truth: "This command evaluates the reconstructed GRN (from GRouNdGAN-simulate data) against the ground truth GRN (used for simulation)."
- [intro] Eight methods benchmarked, results aligned with BEELINE: "Using GRouNdGAN, we benchmark eight GRN inference methods and find the results to be aligned with BEELINE’s experimental results19."
- [methods] Ground truth GRN exported as CSV for benchmarking: "If you specify ``[GRN Benchmarking]/ground truth save path``, the ground truth GRN will be exported as a CSV:"
- [discussion] Recommended edge-finding from reference dataset before imposing GRN: "we recommend first finding a set of regulatory edges from the reference dataset (e.g., using GRNBoost2 or even other data modalities)"
