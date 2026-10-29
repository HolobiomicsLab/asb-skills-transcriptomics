---
name: sensitivity-fdr-curve-evaluation
description: Use when evaluating TFBS or regulatory-element predictions against a
  ChIP-sep gold standard by ranking candidate motif sites and quantifying sensitivity
  at controlled FDR levels, especially to compare predicted vs. true DH or baseline
  rankings.
when_to_use_negative:
- No ChIP-seq gold standard is available to define positive motif sites, so FDR cannot
  be estimated
- The goal is predicting new DNase I hypersensitive site locations rather than benchmarking
  ranked TFBS predictions — BIRD will not discover new DHS locations
- Sites lack motif occurrences, so there is no set of candidate motif sites to rank
  by predicted DH
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0203
tools:
- name: BIRD_predict
  role: Produces the predicted DH matrix used to rank motif sites for the sensitivity-FDR
    evaluation
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD
  role: Framework generating BIRD-predicted DH (predicted DNase-seq signal genome-wide)
    that is benchmarked against the ChIP-seq gold standard
  repo: https://github.com/WeiqiangZhou/BIRD
- name: R
  role: Required runtime for BIRD and used to compute sensitivity-FDR curves and aggregate
    the per-TF metric table
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:c7dd7b869c61ada850f5ee95262730bdd2efc910eeb0ff52da6bdf7cfecbb2e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WeiqiangZhou/BIRD
  tools:
  - BIRD
  - BIRD_predict
  - R
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# Sensitivity-FDR curve evaluation

## Summary

Benchmark transcription factor-binding site (TFBS) predictions by ranking motif sites by a predicted accessibility score (e.g., BIRD-predicted DNase I hypersensitivity, DH) and computing sensitivity at a series of false discovery rate thresholds, integrating the sensitivity-FDR curve (AUSFC) alongside AUROC.

## When to use

Use when evaluating TFBS or regulatory-element predictions against a ChIP-sep gold standard by ranking candidate motif sites and quantifying sensitivity at controlled FDR levels, especially to compare predicted vs. true DH or baseline rankings.

## When NOT to use

- No ChIP-seq gold standard is available to define positive motif sites, so FDR cannot be estimated
- The goal is predicting new DNase I hypersensitive site locations rather than benchmarking ranked TFBS predictions — BIRD will not discover new DHS locations
- Sites lack motif occurrences, so there is no set of candidate motif sites to rank by predicted DH

## Inputs

- BIRD-predicted DH matrix (log2(x+1) scale) from BIRD_predict run on a GM12878 FPKM data matrix
- Motif occurrences for the nine transcription factors (e.g., ELF1) in GM12878
- ENCODE ChIP-seq peak gold standard defining positive TFBS motif sites
- True DNase-seq data as reference ceiling
- Baseline rankings: motif-only and mean-DH scores

## Outputs

- Sensitivity-FDR curves per transcription factor for BIRD, motif-only, mean-DH, and true DNase-seq
- Per-TF AUSFC and AUROC values
- Aggregated comparison table tfbs_benchmark_metrics.csv

## How to apply

Intersect predicted DH values (e.g., BIRD-predicted DH in log2(x+1) scale from BIRD_predict with the human_hg38_model.bin and default upper bound -u 14) with motif occurrences of the transcription factors of interest to assign each motif site a predicted accessibility score. Define positives for each TF as motif sites overlapping the ENCODE ChIP-seq peak gold standard, and rank all motif sites by predicted DH. Sweep FDR thresholds, compute sensitivity at each threshold to construct the sensitivity-FDR curve, and integrate it to obtain AUSFC; also compute AUROC. Repeat the identical ranking and metric computation for the motif-only and mean-DH baselines, and for true DNase-seq data as the reference ceiling, then aggregate per-TF AUSFC and AUROC into a comparison table (e.g., tfbs_benchmark_metrics.csv). Validation reference: for ELF1 in GM12878, BIRD achieves AUSFC = 0.61 and AUROC = 0.93, exceeding both baselines and approaching true DNase-seq performance.

## Related tools

- **BIRD_predict** (Produces the predicted DH matrix used to rank motif sites for the sensitivity-FDR evaluation) — https://github.com/WeiqiangZhou/BIRD
- **BIRD** (Framework generating BIRD-predicted DH (predicted DNase-seq signal genome-wide) that is benchmarked against the ChIP-seq gold standard) — https://github.com/WeiqiangZhou/BIRD
- **R** (Required runtime for BIRD and used to compute sensitivity-FDR curves and aggregate the per-TF metric table)

## Examples

```
path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt
```

## Evaluation signals

- Per-TF AUROC and AUSFC values are finite and within [0, 1], with BIRD's values approaching the true DNase-seq ceiling (e.g., AUROC = 0.93 and AUSFC = 0.61 for ELF1 in GM12878)
- BIRD's AUSFC and AUROC exceed both the motif-only and mean-DH baselines for the nine TFs
- The sensitivity-FDR curve is monotone, constructed over a series of FDR thresholds, and integrates to the reported AUSFC
- tfbs_benchmark_metrics.csv contains rows for all nine TFs across BIRD, both baselines, and true DNase-seq
- Positives (motif sites overlapping ENCODE ChIP-seq peaks) are a strict subset of the ranked motif site universe per TF

## Limitations

- BIRD will not discover new locations of DHSs when applied to analyze a new gene expression sample, so rankings are confined to known loci/motif sites
- The true DNase-seq performance is a ceiling reference; BIRD-predicted DH approximates but does not reach it
- Whether gene expression can similarly predict other functional genomic data types remains an open question

## Evidence

- [abstract] TFBS prediction validated via sensitivity-FDR curves against ENCODE ChIP-seq gold standard: "We show applications of BIRD-predicted DH in predicting transcription factor-binding sites (TFBSs)"
- [results] BIRD-predicted DH close to true DNase-seq for TFBS prediction: [evidence span withheld — anchor status: not_found]
- [methods] BIRD_predict invocation producing the DH matrix used for ranking: "path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt"
- [readme] Output scale and default upper bound used for the predicted values: "Standard output will save a matrix contained all predited value in log scale (log2(x+1) transformed)."
- [discussion] BIRD cannot discover new DHS locations, limiting what ranking can capture: "one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample"
