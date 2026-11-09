---
name: tfbs-prediction-from-predicted-dnase-hypersensitivity
description: Use when TFBSs must be predicted for a sample with gene expression (FPKM
  matrix) but no matched DNase-seq data, and predicted DH can substitute for experimental
  accessibility to rank motif sites.
when_to_use_negative:
- When experimental DNase-seq data are available for the sample — true DNase-seq serves
  as the reference ceiling and should be used directly
- When the goal is to discover novel DHS locations rather than score known motif sites
  — BIRD will not discover new locations of DHSs when applied to a new gene expression
  sample
- When only a ChIP-seq peak gold standard exists for a TF but no motif occurrences
  — the workflow scores motif sites and cannot rank sites lacking motif instances
edam_operation: http://edamontology.org/operation_0445
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0621
tools:
- name: BIRD
  role: Framework for predicting DNase I hypersensitivity from gene expression; predicted
    DH is used in place of experimental DNase-seq to score TFBSs
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD_predict
  role: Command-line prediction step that converts the FPKM input matrix (via -b model,
    -i input, -o output) into a predicted DH matrix used for motif-site ranking
  repo: https://github.com/WeiqiangZhou/BIRD
- name: R
  role: Required runtime for BIRD (Rscript must be executable) and used to compute
    sensitivity-FDR curves, AUSFC, AUROC, and aggregate the benchmark metrics table
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:6eb4f1d89930c3963845be9f49561718a3a5792bb7844f3370f7f6713954790c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WeiqiangZhou/BIRD
  tools:
  - BIRD
  - BIRD_predict
  - R
  edam_operation: http://edamontology.org/operation_0445
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# tfbs-prediction-from-predicted-dnase-hypersensitivity

## Summary

Benchmarks transcription factor-binding site (TFBS) prediction by intersecting motif occurrences with BIRD-predicted DNase I hypersensitivity (DH) derived from gene expression, comparing against motif-only and mean-DH baselines and true DNase-seq when experimental DNase data are unavailable.

## When to use

Use when TFBSs must be predicted for a sample with gene expression (FPKM matrix) but no matched DNase-seq data, and predicted DH can substitute for experimental accessibility to rank motif sites.

## When NOT to use

- When experimental DNase-seq data are available for the sample — true DNase-seq serves as the reference ceiling and should be used directly
- When the goal is to discover novel DHS locations rather than score known motif sites — BIRD will not discover new locations of DHSs when applied to a new gene expression sample
- When only a ChIP-seq peak gold standard exists for a TF but no motif occurrences — the workflow scores motif sites and cannot rank sites lacking motif instances

## Inputs

- Gene expression FPKM data matrix with gene Ensembl IDs in the first column (e.g., GM12878 FPKM_data_matrix.txt)
- Prebuilt BIRD prediction model binary (e.g., human_hg38_model.bin)
- Motif occurrences for the transcription factors of interest (nine TFs in the GM12878 benchmark)
- ENCODE ChIP-seq peak gold standard for each TF (to define positive motif sites)
- True DNase-seq data for the sample (reference ceiling for comparison)

## Outputs

- Matrix of BIRD-predicted DH values in log2(x+1) scale (output_file.txt)
- Motif sites ranked by predicted DH accessibility score
- Sensitivity-FDR curves per TF, integrated to AUSFC, plus AUROC per TF
- Aggregated comparison table of per-TF AUSFC/AUROC for BIRD, motif-only baseline, mean-DH baseline, and true DNase-seq (tfbs_benchmark_metrics.csv)

## How to apply

Run BIRD_predict with the prebuilt human model binary (e.g., human_hg38_model.bin via -b) on the GM12878 FPKM data matrix (via -i) to produce predicted DH values in log2(x+1) scale, using the default upper bound of 14 (-u) if needed. Intersect the predicted DH values with motif occurrences of the transcription factors of interest to assign each motif site a predicted accessibility score. Define positives for each TF as motif sites overlapping the ENCODE ChIP-seq peak gold standard, then rank all motif sites by predicted DH. Compute sensitivity across a series of false discovery rate thresholds to build sensitivity-FDR curves, integrating them for AUSFC, and also compute AUROC. Repeat the ranking and metric computation for motif-only and mean-DH baselines and for true DNase-seq as the reference ceiling, then aggregate per-TF AUSFC and AUROC into a comparison table (e.g., tfbs_benchmark_metrics.csv). Correct application is evidenced when BIRD's scores approach true DNase-seq performance and exceed the baselines (e.g., AUSFC = 0.61 and AUROC = 0.93 for ELF1 in GM12878).

## Related tools

- **BIRD** (Framework for predicting DNase I hypersensitivity from gene expression; predicted DH is used in place of experimental DNase-seq to score TFBSs) — https://github.com/WeiqiangZhou/BIRD
- **BIRD_predict** (Command-line prediction step that converts the FPKM input matrix (via -b model, -i input, -o output) into a predicted DH matrix used for motif-site ranking) — https://github.com/WeiqiangZhou/BIRD
- **R** (Required runtime for BIRD (Rscript must be executable) and used to compute sensitivity-FDR curves, AUSFC, AUROC, and aggregate the benchmark metrics table)

## Examples

```
path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt
```

## Evaluation signals

- BIRD-based TFBS prediction achieves AUSFC = 0.61 and AUROC = 0.93 for ELF1 in GM12878, approaching true DNase-seq performance
- Per-TF AUSFC and AUROC for BIRD exceed both the motif-only and mean-DH baselines across the nine TFs
- BIRD_predict output is a matrix of predicted values log2(x+1)-transformed, bounded 0 to 14 by default
- Gene ID matching produces input_file.match.txt (e.g., FPKM_data_matrix.txt.match.txt) with Ensembl IDs matched between model and input
- A comparison table (tfbs_benchmark_metrics.csv) is populated with AUSFC/AUROC rows for BIRD, both baselines, and true DNase-seq

## Limitations

- BIRD will not discover new locations of DHSs when applied to analyze a new gene expression sample — only known DHS loci are scored
- Whether gene expression can be similarly used to predict other functional genomic data types remains unanswered
- Extending BIRD to other gene expression platforms is future work

## Evidence

- [results] TFBS benchmark headline result: [evidence span withheld — anchor status: not_found]
- [methods] BIRD_predict CLI usage: "path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt"
- [readme] Output scale and R requirement: "Standard output will save a matrix contained all predited value in log scale (log2(x+1) transformed)."
- [readme] Upper bound default: "The predicted values are now bounded from 0 to 14. Users can use the -u option to change the upper bound"
- [readme] Gene id matching behavior: "BIRD will first match the ensembl id of the genes used for building the prediction model with the ids in the input data matrix."
- [abstract] Application scope of BIRD-predicted DH: "We show applications of BIRD-predicted DH in predicting transcription factor-binding sites (TFBSs)"
- [discussion] Limitation on novel DHS discovery: "one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample"
