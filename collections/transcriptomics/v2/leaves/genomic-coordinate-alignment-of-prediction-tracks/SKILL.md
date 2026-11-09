---
name: genomic-coordinate-alignment-of-prediction-tracks
description: Use when two genome-wide DH or regulome track sets (e.g., BIRD predictions
  and ChromImpute imputations) must be compared across shared loci and cell types,
  requiring coordinate/locus matching before correlation.
when_to_use_negative:
- When only one prediction track exists and no reference (e.g., ChromImpute) track
  is available for comparison — there is nothing to align against.
- When the two tracks are on different genome builds (e.g., hg19 vs hg38) and no liftover/reconciliation
  is possible — coordinate matching would silently fail or mismatch loci.
- When the goal is de novo DHS discovery rather than comparing existing tracks — BIRD
  will not discover new DHS locations, so locus-level alignment to new coordinates
  is out of scope.
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Used to align BIRD and ChromImpute profiles on shared genomic loci (hg19/hg38
    coordinates) and compute per-cell-type cross-locus and cross-cell-type agreement
    metrics
- name: BIRD_predict
  role: Produces the BIRD-predicted DH matrices (log2(x+1), bounded to 14) that form
    one side of the aligned track comparison
  repo: github.com/WeiqiangZhou/BIRD
- name: BIRD
  role: Source of the gene-expression-based DH predictions and the locus definitions
    (hg19-based genomic information, 200bp windows in BIRD-data files)
  repo: github.com/WeiqiangZhou/BIRD
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:c615fb37244e059a0ac929e75104e4b9ccb9709ba80223efbd3656db4e81a93d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BIRD
  - BIRD_predict
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# Genomic coordinate alignment of prediction tracks

## Summary

Aligns genome-wide prediction tracks (e.g., BIRD-predicted DNase I hypersensitivity) against reference imputation tracks (e.g., ChromImpute DH) on shared genomic loci in hg19/hg38 coordinates so that per-cell-type cross-locus and per-locus cross-cell-type agreement can be computed in R.

## When to use

Use when two genome-wide DH or regulome track sets (e.g., BIRD predictions and ChromImpute imputations) must be compared across shared loci and cell types, requiring coordinate/locus matching before correlation.

## When NOT to use

- When only one prediction track exists and no reference (e.g., ChromImpute) track is available for comparison — there is nothing to align against.
- When the two tracks are on different genome builds (e.g., hg19 vs hg38) and no liftover/reconciliation is possible — coordinate matching would silently fail or mismatch loci.
- When the goal is de novo DHS discovery rather than comparing existing tracks — BIRD will not discover new DHS locations, so locus-level alignment to new coordinates is out of scope.

## Inputs

- BIRD-predicted DH data matrix per cell type (log2(x+1) values, upper bound 14, hg19/hg38 locus coordinates)
- ChromImpute-imputed DH tracks for the same 10 test cell types
- Locus coordinate table (chromosome, start, end; hg19 and/or hg38) from the BIRD model or data files

## Outputs

- Aligned BIRD vs. ChromImpute track matrices on shared genomic loci per cell type
- Cross-locus and cross-cell-type agreement (correlation) metrics per cell type
- Differential-DH agreement metrics for highly similar cell type pairs
- bird_vs_chromimpute_comparison.csv summarizing all metrics

## How to apply

Obtain the BIRD-predicted DH matrices for each test cell type (produced by BIRD_predict on the FPKM expression matrix, values in log2(x+1) scale bounded at 14) and the ChromImpute-imputed DH tracks for the same cell types. Because BIRD's genomic information is based on hg19 while models such as human_hg38_model.bin target hg38, first reconcile the genome build of both track sets, then align the profiles on shared genomic loci using R, matching by coordinate (200bp window locus) rather than by sequence. With the aligned matrices, compute per-cell-type cross-locus agreement (correlation of predicted vs. imputed DH across all shared loci) and per-locus cross-cell-type agreement (correlation across cell types at each locus), and summarize both in a results table such as bird_vs_chromimpute_comparison.csv. For highly similar cell type pairs, compute differential DH between pair members under both methods on the aligned loci to compare differential predictions.

## Related tools

- **R** (Used to align BIRD and ChromImpute profiles on shared genomic loci (hg19/hg38 coordinates) and compute per-cell-type cross-locus and cross-cell-type agreement metrics)
- **BIRD_predict** (Produces the BIRD-predicted DH matrices (log2(x+1), bounded to 14) that form one side of the aligned track comparison) — github.com/WeiqiangZhou/BIRD
- **BIRD** (Source of the gene-expression-based DH predictions and the locus definitions (hg19-based genomic information, 200bp windows in BIRD-data files)) — github.com/WeiqiangZhou/BIRD

## Evaluation signals

- Aligned matrices contain exactly the shared genomic loci between BIRD and ChromImpute tracks, with coordinates matching after build reconciliation.
- Cross-locus and cross-cell-type correlation/agreement metrics are reported for all 10 shared test cell types in the results table.
- The comparison table (bird_vs_chromimpute_comparison.csv) shows performance comparable between BIRD and ChromImpute, with neither method consistently outperforming the other, matching the paper's finding.
- Differential-DH comparison for highly similar cell type pairs shows BIRD substantially outperforming ChromImpute, consistent with the reported result.
- BIRD-predicted values fed into the alignment are in log2(x+1) scale and bounded by the default upper value of 14.

## Limitations

- BIRD will not discover new locations of DHSs when applied to analyze a new gene expression sample, so alignment is restricted to the loci covered by the model/track.
- Genomic information in the current version of BIRD is based on human genome hg19, while newer models target hg38; tracks on mixed builds must be reconciled before locus matching.
- It remains an open question whether gene expression can be similarly used to predict other functional genomic data types, limiting the comparison's extension beyond DH tracks.

## Evidence

- [other] Align the BIRD and ChromImpute profiles on shared genomic loci (hg19/hg38 coordinates) using R for per-cell-type cross-locus comparison and per-locus cross-cell-type comparison.: [evidence span withheld — anchor status: not_found]
- [methods] Genomic information in the current version of BIRD is based on human genome hg19.: "Genomic information in the current version of BIRD is based on **human genome hg19**."
- [readme] The reference genome for these datasets is huamn genome hg19.: [evidence span withheld — anchor status: external_doc]
- [readme] The first three columns in the data contains the location information of a genomic locus (200bp window).: [evidence span withheld — anchor status: external_doc]
