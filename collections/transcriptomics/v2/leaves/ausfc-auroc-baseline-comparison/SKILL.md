---
name: ausfc-auroc-baseline-comparison
description: Use when evaluating whether accessibility-based TFBS predictions (e.g.,
  from BIRD-predicted DH in GM12878) recover known binding sites better than motif-only
  or mean-DH baselines, using AUSFC and AUROC metrics.
when_to_use_negative:
- The prediction task is not site-level TFBS recovery — e.g., genome-wide regression
  of DH values themselves, where AUROC/AUSFC against ChIP-seq positives is not the
  appropriate metric
- No gold-standard ChIP-seq peaks exist for the TF/cell type, since positives cannot
  be defined and sensitivity-FDR curves cannot be constructed
- Evaluating new DHS discovery rather than ranking known motif sites — BIRD explicitly
  will not discover new locations of DHSs, so ranking-based TFBS benchmarks do not
  apply
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: BIRD
  role: Source of the predicted DH values being benchmarked for TFBS prediction quality
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD_predict
  role: Produces the predicted DH matrix (log2(x+1), -u upper bound default 14) that
    is intersected with motif sites for ranking
  repo: https://github.com/WeiqiangZhou/BIRD
- name: R
  role: Required runtime for computing sensitivity-FDR curves, AUSFC, and AUROC benchmark
    metrics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:14b11d1a925784aa802326e664a809646f6596b234b5d52b903df69057fc16bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WeiqiangZhou/BIRD
  tools:
  - BIRD
  - BIRD_predict
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# ausfc-auroc-baseline-comparison

## Summary

Benchmark TFBS predictions built from predicted DNase I hypersensitivity (DH) by ranking motif sites, defining positives via ChIP-seq gold standards, and computing sensitivity-FDR curves (AUSFC) alongside AUROC, compared against motif-only and mean-DH baselines with true DNase-seq as the performance ceiling.

## When to use

Use when evaluating whether accessibility-based TFBS predictions (e.g., from BIRD-predicted DH in GM12878) recover known binding sites better than motif-only or mean-DH baselines, using AUSFC and AUROC metrics.

## When NOT to use

- The prediction task is not site-level TFBS recovery — e.g., genome-wide regression of DH values themselves, where AUROC/AUSFC against ChIP-seq positives is not the appropriate metric
- No gold-standard ChIP-seq peaks exist for the TF/cell type, since positives cannot be defined and sensitivity-FDR curves cannot be constructed
- Evaluating new DHS discovery rather than ranking known motif sites — BIRD explicitly will not discover new locations of DHSs, so ranking-based TFBS benchmarks do not apply

## Inputs

- BIRD-predicted DH matrix in log2(x+1) scale (e.g., from BIRD_predict -b human_hg38_model.bin on a GM12878 FPKM data matrix)
- Motif occurrence sites for the transcription factors of interest (e.g., nine TFs including ELF1 in GM12878)
- ENCODE ChIP-seq peak gold standard sets defining positive TFBS for each TF
- True DNase-seq signal for the same cell type (reference ceiling)
- Mean DH baseline values and motif scores for the same motif sites
- R for computing sensitivity-FDR curves, AUSFC, and AUROC

## Outputs

- Per-TF AUSFC and AUROC values for BIRD-predicted DH, motif-only baseline, mean-DH baseline, and true DNase-seq
- Aggregated comparison table (e.g., tfbs_benchmark_metrics.csv)
- Sensitivity-FDR curves per TF per method

## How to apply

For each transcription factor, intersect predicted DH values (in log2(x+1) scale, upper bound 14 by default from BIRD_predict) with motif occurrences to assign an accessibility score to each motif site. Define positives as motif sites overlapping the ENCODE ChIP-seq peak gold standard and rank all motif sites by the predicted score. Compute sensitivity across a series of false discovery rate thresholds to construct sensitivity-FDR curves and integrate them to obtain AUSFC, and additionally compute AUROC. Repeat the ranking and metric computation for the motif-only baseline (rank by motif score alone) and the mean-DH baseline, and use true DNase-seq data as the reference ceiling. Aggregate per-TF AUSFC and AUROC values into a comparison table (e.g., tfbs_benchmark_metrics.csv); correctness is judged by BIRD approaching true DNase-seq performance and exceeding both baselines, e.g., AUSFC = 0.61 and AUROC = 0.93 for ELF1 in GM12878.

## Related tools

- **BIRD** (Source of the predicted DH values being benchmarked for TFBS prediction quality) — https://github.com/WeiqiangZhou/BIRD
- **BIRD_predict** (Produces the predicted DH matrix (log2(x+1), -u upper bound default 14) that is intersected with motif sites for ranking) — https://github.com/WeiqiangZhou/BIRD
- **R** (Required runtime for computing sensitivity-FDR curves, AUSFC, and AUROC benchmark metrics)

## Examples

```
path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt
```

## Evaluation signals

- BIRD AUSFC and AUROC should exceed both the motif-only and mean-DH baselines for each TF (e.g., ELF1: AUSFC = 0.61, AUROC = 0.93)
- BIRD metrics should approach the true DNase-seq reference ceiling rather than fall far below it
- Sensitivity-FDR curves must be constructed from sensitivity computed at a series of FDR thresholds and integrated to yield AUSFC; verify the integration is monotone in ranking quality
- Positives are motif sites overlapping the ENCODE ChIP-seq peak gold standard — verify positive counts per TF are consistent across compared methods
- All methods must be ranked over the identical set of motif sites so the comparison table (tfbs_benchmark_metrics.csv) is directly comparable

## Limitations

- BIRD will not discover new locations of DHSs when applied to analyze a new gene expression sample, so it can only re-rank known motif/locus sites, not find novel binding sites
- Predicted values are bounded from 0 to 14 by default (-u), which caps the dynamic range of scores used for ranking
- Performance depends on the model matching the expression data type and genome build (e.g., human_hg38_model.bin for hg38 RNA-seq FPKM)

## Evidence

- [results] AUSFC definition and comparison setup: [evidence span withheld — anchor status: not_found]
- [methods] BIRD_predict invocation: "path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt"
- [methods] R requirement: "It is required that **R** is installed and make sure the **Rscript** command is executable"
- [readme] Output scale and upper bound: "Standard output will save a matrix contained all predited value in log scale (log2(x+1) transformed)."
- [readme] Upper bound option: "The predicted values are now bounded from 0 to 14. Users can use the -u option to change the upper bound when using their own prediction model."
