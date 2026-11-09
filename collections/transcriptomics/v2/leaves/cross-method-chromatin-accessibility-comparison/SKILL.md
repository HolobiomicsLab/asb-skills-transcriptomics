---
name: cross-method-chromatin-accessibility-comparison
description: Use when evaluating whether expression-based DH prediction (BIRD, log2(x+1)
  scale) performs comparably to multi-data-type imputation (ChromImpute) on shared
  test cell types, or when comparing differential DH between similar cell type pairs.
when_to_use_negative:
- The goal is discovering DNase I hypersensitive sites at new genomic locations —
  BIRD only predicts DH at loci in its model and will not discover new DHS locations
- The genome is not human hg19/hg38, since BIRD's current genomic information is based
  on human genome hg19 with hg38 models released separately
- No matched gene expression (FPKM RNA-seq or exon array) data exist for the cell
  types, since BIRD prediction requires an expression matrix as its only input
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BIRD_predict
  role: Generates the BIRD-predicted DH matrices for the 10 test cell types from each
    FPKM expression matrix using the human_hg38_model.bin model
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD
  role: Underlying software being benchmarked; its gene-expression-based DH predictions
    are compared to ChromImpute imputations
  repo: https://github.com/WeiqiangZhou/BIRD
- name: R
  role: Aligns BIRD and ChromImpute profiles on shared genomic loci and computes genome-wide
    correlation/agreement metrics, including differential-DH comparisons
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:574e71789054bd204b99b7c286323c2720b1d9fc55fc929bf0611d959a2c48fd
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cross-method-chromatin-accessibility-comparison

## Summary

Benchmarks BIRD-predicted DNase I hypersensitivity (DH), which uses only gene expression, against ChromImpute-imputed DH, which integrates multiple functional genomic data types, across the 10 shared test cell types, including cross-locus and cross-cell-type agreement plus differential-DH comparisons between highly similar cell type pairs.

## When to use

Use when evaluating whether expression-based DH prediction (BIRD, log2(x+1) scale) performs comparably to multi-data-type imputation (ChromImpute) on shared test cell types, or when comparing differential DH between similar cell type pairs.

## When NOT to use

- The goal is discovering DNase I hypersensitive sites at new genomic locations — BIRD only predicts DH at loci in its model and will not discover new DHS locations
- The genome is not human hg19/hg38, since BIRD's current genomic information is based on human genome hg19 with hg38 models released separately
- No matched gene expression (FPKM RNA-seq or exon array) data exist for the cell types, since BIRD prediction requires an expression matrix as its only input

## Inputs

- FPKM gene expression matrix with gene ensembl ids as the first column (FPKM_data_matrix.txt) for each of the 10 test cell types
- Prebuilt prediction model human_hg38_model.bin from the BIRD-model repository
- ChromImpute-imputed DH tracks for the same 10 cell types (multi-data-type imputation)
- Genomic locus coordinate information (hg19/hg38) for aligning BIRD and ChromImpute profiles

## Outputs

- BIRD-predicted DH data matrix per cell type (log2(x+1), values bounded 0-14)
- Matched input matrix (e.g. FPKM_data_matrix.txt.match.txt) confirming gene id alignment
- Per-cell-type cross-locus and cross-cell-type correlation/agreement table for BIRD vs ChromImpute
- Differential-DH agreement metrics per highly similar cell type pair
- Aggregated comparison CSV (e.g. bird_vs_chromimpute_comparison.csv)

## How to apply

Run BIRD_predict with the human_hg38_model.bin on each cell type's FPKM expression matrix (BIRD auto-matches ensembl gene ids and outputs predictions in log2(x+1) scale, bounded from 0 to 14 by default). Retrieve the ChromImpute-imputed DH tracks for the same 10 cell types. Align both methods' profiles on shared genomic loci using R for per-cell-type cross-locus comparison and per-locus cross-cell-type comparison. Compute genome-wide correlation/agreement metrics per cell type and summarize them in a results table. For highly similar cell type pairs, compute differential DH under both methods and compare; the paper's finding is that performance is comparable between methods at bulk level, but BIRD substantially outperforms ChromImpute for differential DH between high-similarity pairs. Write the aggregated metrics to a CSV such as bird_vs_chromimpute_comparison.csv.

## Related tools

- **BIRD_predict** (Generates the BIRD-predicted DH matrices for the 10 test cell types from each FPKM expression matrix using the human_hg38_model.bin model) — https://github.com/WeiqiangZhou/BIRD
- **BIRD** (Underlying software being benchmarked; its gene-expression-based DH predictions are compared to ChromImpute imputations) — https://github.com/WeiqiangZhou/BIRD
- **R** (Aligns BIRD and ChromImpute profiles on shared genomic loci and computes genome-wide correlation/agreement metrics, including differential-DH comparisons)

## Examples

```
path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt
```

## Evaluation signals

- The results table reports cross-locus and cross-cell-type agreement for all 10 shared test cell types, with neither BIRD nor ChromImpute consistently outperforming the other
- Each input expression matrix produces a matched gene-id file (input_file.match.txt), confirming ensembl id matching against the model
- Predicted values fall in log2(x+1) scale and are bounded between 0 and 14 (default upper bound; adjustable via -u)
- The differential-DH comparison shows BIRD outperforming ChromImpute for highly similar cell type pairs, consistent with the paper's finding
- All aggregated metrics (including differential-DH agreement per pair) are present in bird_vs_chromimpute_comparison.csv

## Limitations

- One limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample
- Predicted values are bounded from 0 to 14 by default; the -u option changes the upper bound, and the -l option switches to the locus-level model
- The comparison covers only the 10 shared test cell types where both BIRD predictions and ChromImpute imputations are available

## Evidence

- [results] comparable performance finding: [evidence span withheld — anchor status: not_found]
- [methods] BIRD_predict command: "path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt"
- [readme] gene id matching: "BIRD will first match the ensembl id of the genes used for building the prediction model with the ids in the input data matrix."
- [readme] prediction bound: "The predicted values are now bounded from 0 to 14. Users can use the -u option to change the upper bound when using their own prediction model."
- [discussion] new DHS limitation: "one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample"
