---
name: encode-dnase-seq-truth-standard-selection
description: Use when a cell type has two ENCODE DNase-seq replicates and the goal
  is to quantify whether BIRD-predicted DH adds signal to the observed replicate in
  low signal-to-noise or few-replicate settings.
when_to_use_negative:
- The cell type has only one DNase-seq replicate, so no independent held-out truth
  track exists for the correlation benchmark
- The goal is to discover novel DHS locations in a new expression sample — BIRD will
  not discover new locations of DHSs
- Input expression data is raw CEL or unaligned reads rather than an FPKM matrix with
  ensembl gene ids in the first column
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: BIRD_predict
  role: Generates the genome-wide predicted DH track (log2(x+1), upper bound 14 via
    -u) used as the pseudo-replicate combined with the observed replicate
  repo: https://github.com/WeiqiangZhou/BIRD
- name: R
  role: Constructs the combined observed+BIRD profile on the model's genomic bins
    and computes genome-wide Pearson correlations against the held-out truth replicate
  repo: https://www.r-project.org
- name: BIRD-data
  role: Source of preprocessed hg19 DNase-seq tracks on 200bp windows (57 ENCODE exon-array,
    70 Roadmap, and 167 ENCODE RNA-seq samples) used as observed and truth replicates
  repo: https://github.com/WeiqiangZhou/BIRD-data
- name: BIRD-model
  role: Provides the prebuilt prediction models (human_hg38_model.bin for RNA-seq
    FPKM) required by BIRD_predict
  repo: https://github.com/WeiqiangZhou/BIRD-model
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:576e37249538d9aaec7d3973779c36b816795482f7a0a603e2b9614abe104b7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WeiqiangZhou/BIRD
  tools:
  - BIRD
  - BIRD-data
  - BIRD-model
  - BIRD_predict
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# encode-dnase-seq-truth-standard-selection

## Summary

Benchmarking scheme that designates one ENCODE DNase-seq replicate as a held-out truth track and tests whether combining the other replicate with a BIRD-predicted DNase I hypersensitivity (DH) profile as a pseudo-replicate improves genome-wide Pearson correlation with the truth.

## When to use

Use when a cell type has two ENCODE DNase-seq replicates and the goal is to quantify whether BIRD-predicted DH adds signal to the observed replicate in low signal-to-noise or few-replicate settings.

## When NOT to use

- The cell type has only one DNase-seq replicate, so no independent held-out truth track exists for the correlation benchmark
- The goal is to discover novel DHS locations in a new expression sample — BIRD will not discover new locations of DHSs
- Input expression data is raw CEL or unaligned reads rather than an FPKM matrix with ensembl gene ids in the first column

## Inputs

- GM12878 (and other test cell type) RNA-seq FPKM matrix with gene ensembl ids in the first column (FPKM_data_matrix.txt)
- Two ENCODE DNase-seq replicates for each test cell type (one held out as truth, one as observed-only)
- human_hg38_model.bin (or human_hg19_model.bin) prebuilt BIRD prediction model
- Genomic bin coordinates (200bp windows) used by the BIRD model

## Outputs

- BIRD-predicted DH data matrix in log2(x+1) scale bounded at 14 (output_file.txt)
- Pearson correlation summary per candidate profile (observed-only, BIRD-only, combined) vs the held-out truth replicate
- dh_prediction_correlation_summary.csv listing each profile's correlation, the combined-minus-observed delta, and improvement counts (12/16 test cell types, 9/12 ChIP-seq cases)

## How to apply

Set aside one ENCODE DNase-seq replicate as the held-out truth track and keep the second replicate as the observed-only component. Run BIRD_predict with the human_hg38_model.bin model on the cell type's RNA-seq FPKM matrix to obtain genome-wide predicted DH values in log2(x+1) scale, using the default upper bound (-u 14). In R, construct the BIRD-augmented profile by combining the BIRD-predicted DH track with the observed replicate track, aligning all tracks on the hg19/hg38 200bp genomic bin coordinates used by the model. Compute genome-wide Pearson correlations between the truth replicate and each candidate profile: observed replicate alone, BIRD prediction alone, and the combined observed+BIRD profile. Report the combined-minus-observed delta per cell type and aggregate the improvement counts; the acceptance criterion is that the combined profile's correlation with the held-out replicate exceeds the observed-only baseline (paper margins: ~0.82 vs ~0.76).

## Related tools

- **BIRD_predict** (Generates the genome-wide predicted DH track (log2(x+1), upper bound 14 via -u) used as the pseudo-replicate combined with the observed replicate) — https://github.com/WeiqiangZhou/BIRD
- **R** (Constructs the combined observed+BIRD profile on the model's genomic bins and computes genome-wide Pearson correlations against the held-out truth replicate) — https://www.r-project.org
- **BIRD-data** (Source of preprocessed hg19 DNase-seq tracks on 200bp windows (57 ENCODE exon-array, 70 Roadmap, and 167 ENCODE RNA-seq samples) used as observed and truth replicates) — https://github.com/WeiqiangZhou/BIRD-data
- **BIRD-model** (Provides the prebuilt prediction models (human_hg38_model.bin for RNA-seq FPKM) required by BIRD_predict) — https://github.com/WeiqiangZhou/BIRD-model

## Examples

```
path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt -u 14
```

## Evaluation signals

- Combined observed+BIRD profile's genome-wide Pearson correlation with the held-out replicate (~0.82) must exceed the observed-only baseline (~0.76)
- Pseudo-replicate improvement must hold in the reported fraction of cases: 12 of 16 test cell types for DNase-seq and 9/12 benchmark ChIP-seq cases
- BIRD output values must be in log2(x+1) scale and bounded between 0 and 14 (default upper bound)
- All candidate tracks (observed replicate, BIRD prediction, combined) must align on the same 200bp genomic bin coordinates before correlation
- The matched input matrix (input_file.match.txt) confirms ensembl gene id matching succeeded between the model and the FPKM matrix

## Limitations

- BIRD will not discover new locations of DHSs when applied to analyze a new gene expression sample — the pseudo-replicate can only sharpen known loci
- Predicted values are bounded (default upper bound 14 in log2(x+1) scale), capping extreme predicted DH values
- Genomic information in the current version of BIRD is based on human genome hg19; the hg38 model requires hg38-aligned tracks and mixing builds will corrupt the bin alignment
- The older quantile normalization was replaced by a more robust version, which significantly boosts performance when there are large numbers of tied values — using a stale BIRD version can degrade the pseudo-replicate

## Evidence

- [abstract] BIRD-predicted DH applied as pseudo-replicates: "We show applications of BIRD-predicted DH in predicting transcription factor-binding sites (TFBSs), turning publicly available gene expression samples in Gene Expression Omnibus (GEO) into a regulome"
- [methods] BIRD_predict invocation generating the pseudo-replicate track: "path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt"
- [readme] Output scale and default upper bound for predicted values: "The predicted values are now bounded from 0 to 14. Users can use the -u option to change the upper bound when using their own prediction model."
- [readme] 200bp genomic bin structure of DNase-seq training/truth tracks: [evidence span withheld — anchor status: external_doc]
- [discussion] BIRD cannot discover new DHS locations: "one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample"
