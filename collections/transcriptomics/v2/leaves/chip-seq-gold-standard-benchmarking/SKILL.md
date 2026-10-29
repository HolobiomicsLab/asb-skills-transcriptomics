---
name: chip-seq-gold-standard-benchmarking
description: Use when evaluating whether predicted chromatin accessibility (e.g.,
  BIRD-predicted DH) can rank TF motif sites correctly, by comparing against ENCODE
  ChIP-seq peaks for known TFs such as ELF1 in GM12878.
when_to_use_negative:
- When no ChIP-seq gold standard peaks exist for the TF of interest — positives cannot
  be defined and the benchmark is undefined
- When the goal is discovery of novel DHS locations rather than benchmarking ranked
  TFBS predictions — BIRD will not discover new DHS positions outside its trained
  locus set
- When input is raw RNA-seq reads rather than a gene expression matrix — BIRD requires
  an FPKM data matrix with Ensembl gene IDs as the first column
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
tools:
- name: BIRD
  role: Predicts DNase I hypersensitivity from gene expression, providing the predicted-DH
    scores ranked for TFBS benchmarking
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD_predict
  role: Command-line prediction step producing the log2(x+1)-scale DH matrix intersected
    with motif sites
  repo: https://github.com/WeiqiangZhou/BIRD
- name: R
  role: Required runtime for BIRD (Rscript executable) and used for metric computation
    and aggregation into the comparison CSV
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:92c53ce58e160cc501a49ae60d7f03fdfef4be28b565d7db7a50ebfcf9bc05fa
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# chip-seq-gold-standard-benchmarking

## Summary

Benchmarks predicted transcription factor-binding sites (TFBSs) against ENCODE ChIP-seq peaks treated as a gold standard, using sensitivity-FDR curves (AUSFC) and AUROC, with motif-only and mean-DH baselines and true DNase-seq as the reference ceiling.

## When to use

Use when evaluating whether predicted chromatin accessibility (e.g., BIRD-predicted DH) can rank TF motif sites correctly, by comparing against ENCODE ChIP-seq peaks for known TFs such as ELF1 in GM12878.

## When NOT to use

- When no ChIP-seq gold standard peaks exist for the TF of interest — positives cannot be defined and the benchmark is undefined
- When the goal is discovery of novel DHS locations rather than benchmarking ranked TFBS predictions — BIRD will not discover new DHS positions outside its trained locus set
- When input is raw RNA-seq reads rather than a gene expression matrix — BIRD requires an FPKM data matrix with Ensembl gene IDs as the first column

## Inputs

- Matrix of BIRD-predicted DH values in log2(x+1) scale for ~1M genomic loci (200bp windows, hg19/hg38)
- Motif occurrence sets for the transcription factors under evaluation (e.g., nine TFs including ELF1 in GM12878)
- ENCODE ChIP-seq peak calls (gold standard positives) for each TF
- True DNase-seq signal for the same sample as reference ceiling
- Mean-DH and motif-only baseline scores per motif site

## Outputs

- Sensitivity-FDR curves per TF and method
- Per-TF AUSFC and AUROC values for BIRD, motif-only, mean-DH, and true DNase-seq
- Aggregated comparison table (tfbs_benchmark_metrics.csv)

## How to apply

Intersect predicted DH values (in log2(x+1) scale, upper bound 14 by default via -u) with motif occurrences of the target TFs (e.g., nine TFs in GM12878) to assign each motif site an accessibility score. Define positives as motif sites overlapping the ENCODE ChIP-seq peak gold standard, then rank all motif sites by the predicted score. Compute sensitivity across a series of false discovery rate thresholds to construct sensitivity-FDR curves, integrate to obtain AUSFC, and also compute AUROC. Repeat the identical ranking and metric computation for the motif-only and mean-DH baselines and for true DNase-seq as the reference ceiling, then aggregate per-TF AUSFC/AUROC into a comparison table (e.g., tfbs_benchmark_metrics.csv).

## Related tools

- **BIRD** (Predicts DNase I hypersensitivity from gene expression, providing the predicted-DH scores ranked for TFBS benchmarking) — https://github.com/WeiqiangZhou/BIRD
- **BIRD_predict** (Command-line prediction step producing the log2(x+1)-scale DH matrix intersected with motif sites) — https://github.com/WeiqiangZhou/BIRD
- **R** (Required runtime for BIRD (Rscript executable) and used for metric computation and aggregation into the comparison CSV)

## Examples

```
path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt
```

## Evaluation signals

- BIRD AUSFC for ELF1 in GM12878 should reach ~0.61 and AUROC ~0.93
- BIRD's AUSFC/AUROC must exceed both the motif-only and mean-DH baselines for each TF
- True DNase-seq metrics serve as the reference ceiling; BIRD should approach, not surpass, it
- Positives are exactly motif sites overlapping ENCODE ChIP-seq peaks; site counts per TF must match across all compared methods
- Aggregated per-TF metrics table (tfbs_benchmark_metrics.csv) should contain rows for BIRD, motif-only, mean-DH, and true DNase-seq for all nine TFs

## Limitations

- BIRD will not discover new locations of DHSs when applied to analyze a new gene expression sample, so benchmarking is restricted to its trained locus set
- Whether gene expression can predict other functional genomic data types remains an open question
- BIRD was developed for human hg19 (with hg38 models added later); model and genome build must match the motif/ChIP-seq coordinate system

## Evidence

- [other] BIRD's TFBS prediction performance: [evidence span withheld — anchor status: not_found]
- [readme] predicted values bounded: "The predicted values are now bounded from 0 to 14. Users can use the -u option to change the upper bound when using their own prediction model."
- [methods] BIRD_predict invocation: "path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt"
- [readme] output log scale: "Standard output will save a matrix contained all predited value in log scale (log2(x+1) transformed)."
- [discussion] limitation on new DHS locations: "one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample"
