---
name: genome-wide-dh-prediction-evaluation
description: Use when you have BIRD-predicted DH matrices (log2(x+1)) plus true DNase-seq
  profiles for multiple cell types and need to quantify prediction accuracy overall,
  for differential DH, or versus ChromImpute.
when_to_use_negative:
- You only have a single cell type and no matched true DNase-seq track — there is
  no ground truth to compute prediction–truth correlations against
- You want BIRD to discover novel DHS locations not present in the training loci —
  BIRD will not discover new DHS locations when applied to a new gene expression sample
- Input expression data is exon array raw CEL output that has not yet been processed
  by GeneBASE into a gene expression file
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: BIRD_predict
  role: Generates the log2(x+1)-transformed, 0–14-bounded predicted DH matrices from
    FPKM expression input that are evaluated against true DNase-seq
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD
  role: Prediction engine and source of prebuilt models and training DNase-seq data
    used as ground truth
  repo: https://github.com/WeiqiangZhou/BIRD
- name: R
  role: Used to load DNase-seq .rds truth matrices, align BIRD and ChromImpute profiles
    on shared loci, compute Pearson correlations, and stratify results by similarity
    quartile
  repo: https://www.r-project.org
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:e83caf533ec390e147c3ca2e775055ab17edb5dda847097f02abd5d898318199
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WeiqiangZhou/BIRD
  tools:
  - BIRD
  - BIRD_predict
  - R
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# genome-wide-dh-prediction-evaluation

## Summary

Evaluates BIRD-predicted DNase I hypersensitivity (DH) profiles against true DNase-seq data by computing per-pair and per-locus Pearson correlations, stratifying cell-type pairs by DH-profile similarity quartiles, and benchmarking against ChromImpute imputation. This quantifies how well gene-expression-based prediction recovers genome-wide regulatory element activity, especially for differential DH between similar cell types.

## When to use

Use when you have BIRD-predicted DH matrices (log2(x+1)) plus true DNase-seq profiles for multiple cell types and need to quantify prediction accuracy overall, for differential DH, or versus ChromImpute.

## When NOT to use

- You only have a single cell type and no matched true DNase-seq track — there is no ground truth to compute prediction–truth correlations against
- You want BIRD to discover novel DHS locations not present in the training loci — BIRD will not discover new DHS locations when applied to a new gene expression sample
- Input expression data is exon array raw CEL output that has not yet been processed by GeneBASE into a gene expression file

## Inputs

- BIRD-predicted DH matrix (log2(x+1), values bounded 0–14) from BIRD_predict with human_hg38_model.bin
- True DNase-seq DH profiles for the same cell types (200bp genomic windows, e.g. DNase_data_167_cells.rds from BIRD-data)
- FPKM gene expression matrix with gene ensembl ids in the first column
- Precomputed cell-type similarity measure for quartile stratification
- ChromImpute-imputed DH tracks for shared test cell types (optional, for method comparison)

## Outputs

- Per-cell-type-pair Pearson correlations of differential DH across all loci and across differential loci
- Mean correlation per similarity-quartile stratum (expected ~0.42 all loci / ~0.55 differential loci for the highest-similarity quartile)
- Cross-locus and cross-cell-type agreement table for BIRD vs ChromImpute over 10 shared test cell types
- Differential-DH agreement per cell-type pair (e.g. written to bird_vs_chromimpute_comparison.csv)

## How to apply

Load BIRD-predicted DH matrices produced by BIRD_predict with human_hg38_model.bin (log2(x+1) transformed, bounded 0–14) and the matching true DNase-seq profiles (e.g., the 200bp-window DNase-seq matrices in BIRD-data). Form all unique cell-type pairs (e.g., 136 pairs from 17 test cell types), compute differential DH per locus (difference between the two cell types' DH values) for both predicted and true data, and calculate per-pair Pearson correlations across all loci and across the subset of differential loci. Assign each pair to one of four similarity quartiles based on a precomputed global DH-profile similarity measure and aggregate mean correlations per stratum: the paper reports ~0.42 (all loci) and ~0.55 (differential loci) for the highest-similarity quartile versus ~0.60–0.66 and ~0.69–0.75 for the remaining quartiles. For the method comparison, align BIRD and ChromImpute tracks on shared hg19/hg38 loci for the 10 shared test cell types and summarize cross-locus and cross-cell-type agreement, plus differential-DH agreement for highly similar pairs, where BIRD outperforms ChromImpute.

## Related tools

- **BIRD_predict** (Generates the log2(x+1)-transformed, 0–14-bounded predicted DH matrices from FPKM expression input that are evaluated against true DNase-seq) — https://github.com/WeiqiangZhou/BIRD
- **BIRD** (Prediction engine and source of prebuilt models and training DNase-seq data used as ground truth) — https://github.com/WeiqiangZhou/BIRD
- **R** (Used to load DNase-seq .rds truth matrices, align BIRD and ChromImpute profiles on shared loci, compute Pearson correlations, and stratify results by similarity quartile) — https://www.r-project.org

## Examples

```
path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt
```

## Evaluation signals

- Predicted values are bounded 0–14 in log2(x+1) scale — check output matrix range before computing correlations
- Highest-similarity quartile should show the LOWEST mean correlations (~0.42 all loci, ~0.55 differential loci), with remaining quartiles at ~0.60–0.66 and ~0.69–0.75
- Number of unique cell-type pairs should equal C(n,2), e.g. 136 pairs for 17 test cell types
- BIRD vs ChromImpute table should show comparable cross-locus and cross-cell-type performance across all 10 shared test cell types, with BIRD winning for differential DH between highly similar pairs
- Gene id matching step should produce an input_file.match.txt file confirming ensembl ids were matched to the model (version suffix ignored by default)

## Limitations

- BIRD will not discover new locations of DHSs when applied to analyze a new gene expression sample
- Genomic information in the current version of BIRD is based on human genome hg19 (newer hg38 models exist); coordinate alignment with external tracks like ChromImpute must respect this
- Whether gene expression can similarly predict other functional genomic data types remains an open question

## Evidence

- [methods] BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt: "path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt"
- [readme] Standard output will save a matrix contained all predited value in log scale (log2(x+1) transformed).: "Standard output will save a matrix contained all predited value in log scale (log2(x+1) transformed)."
- [readme] The predicted values are now bounded from 0 to 14.: "The predicted values are now bounded from 0 to 14."
- [readme] The first three columns in the data contains the location information of a genomic locus (200bp window).: [evidence span withheld — anchor status: external_doc]
- [methods] Genomic information in the current version of BIRD is based on human genome hg19.: "Genomic information in the current version of BIRD is based on **human genome hg19**."
- [discussion] one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample: "one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample"
