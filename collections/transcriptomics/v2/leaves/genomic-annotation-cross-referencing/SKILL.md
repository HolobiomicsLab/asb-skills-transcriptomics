---
name: genomic-annotation-cross-referencing
description: Use when a set of variants or genes identified by identifiers (GENCODE
  v19 gene symbols and/or Ensembl gene IDs) must be joined to per-gene quantitative
  scores (MRSD read depths) across biosamples to yield a per-variant classification
  and an aggregate fraction.
when_to_use_negative:
- The goal is assessment of differential gene expression or allele-specific expression,
  which the MRSD model was not designed to predict.
- The RNA-seq data are long-read, single-cell, or spatially resolved; MRSD was validated
  only for bulk short-read poly-A enrichment RNA-seq processed with the Cummings et
  al. pipeline.
- Gene identifiers are not GENCODE v19-compatible symbols or Ensembl gene IDs, so
  they cannot be matched to the .results.mrsd.txt id1/id2 columns.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: MRSDexplorer
  role: Generates the per-gene MRSD score tables (.results.mrsd.txt) per biosample
    that the gene list is cross-referenced against
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: MRSD
  role: Statistical framework producing the gene- and tissue-specific minimum required
    sequencing depth values being joined to the variant list
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: Python
  role: Used with pandas to join each variant's gene to its MRSD score in each of
    the four biosamples and compute the variant fraction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:3e709245852f13f71183ccab0d72fa2b133d3051631bcb058c642feb33e248a5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MRSD
  - MRSDexplorer
  - Python
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# genomic-annotation-cross-referencing

## Summary

Cross-references a variant-anchored gene list (e.g., SpliceAI-prioritized ClinVar VUS-harboring genes) against gene-level MRSD score tables (.results.mrsd.txt) computed for multiple GTEx-derived biosamples, to classify each gene as low- or high-MRSD and quantify the fraction of variants assayable by RNA-seq.

## When to use

Use when a set of variants or genes identified by identifiers (GENCODE v19 gene symbols and/or Ensembl gene IDs) must be joined to per-gene quantitative scores (MRSD read depths) across biosamples to yield a per-variant classification and an aggregate fraction.

## When NOT to use

- The goal is assessment of differential gene expression or allele-specific expression, which the MRSD model was not designed to predict.
- The RNA-seq data are long-read, single-cell, or spatially resolved; MRSD was validated only for bulk short-read poly-A enrichment RNA-seq processed with the Cummings et al. pipeline.
- Gene identifiers are not GENCODE v19-compatible symbols or Ensembl gene IDs, so they cannot be matched to the .results.mrsd.txt id1/id2 columns.

## Inputs

- MRSD score tables (.results.mrsd.txt) for whole blood, LCL, skeletal muscle, and cultured fibroblast GTEx-derived datasets (MRSD in millions of reads)
- One-per-line gene list of 7,507 SpliceAI-filtered ClinVar VUS-harboring genes (GENCODE v19 gene symbol and/or Ensembl gene ID)
- Junction count files (.junc-counts.txt) and sequencing depth files (.seq-depths.txt) per biosample for MRSDexplorer runs

## Outputs

- Per-variant classification table (mrsd_variant_fraction.csv) with per-biosample MRSD values and low-MRSD flags
- Final fraction of SpliceAI-prioritized ClinVar VUSs in low-MRSD genes (expected range 25.8%-67.8%)

## How to apply

First generate MRSD score tables per biosample by running MRSDexplorer.py on GTEx-derived whole blood, LCL, skeletal muscle, and cultured fibroblast junction-count and seq-depths files at --number_reads 8, --sj_prop 0.75, --mrsd_param 0.95, --read_type unique. Load the one-per-line gene list of the 7,507 SpliceAI-filtered ClinVar VUS-harboring genes, matching GENCODE v19 gene symbols and/or Ensembl gene IDs as accepted in the .genes.txt format. In Python (pandas), join each variant's gene to its MRSD score in each of the four biosample .results.mrsd.txt tables; treat '-' entries as genes whose median junction coverage was 0 reads (not assayable). Flag each gene as low-MRSD if its MRSD is <= 100 million reads in at least one biosample, then compute the fraction of variants in low-MRSD genes, stratifying across model stringency settings to yield the 25.8%-67.8% range, and write the per-variant table and final fraction to mrsd_variant_fraction.csv.

## Related tools

- **MRSDexplorer** (Generates the per-gene MRSD score tables (.results.mrsd.txt) per biosample that the gene list is cross-referenced against) — github.com/mcgm-mrsd/mrsd-explorer
- **MRSD** (Statistical framework producing the gene- and tissue-specific minimum required sequencing depth values being joined to the variant list) — github.com/mcgm-mrsd/mrsd-explorer
- **Python** (Used with pandas to join each variant's gene to its MRSD score in each of the four biosamples and compute the variant fraction)

## Examples

```
python MRSDexplorer.py --tissues all --number_reads 8 --sj_prop 0.75 --mrsd_param 0.95 --read_type unique example transcripts.introns.bed
```

## Evaluation signals

- The computed fraction of SpliceAI-prioritized ClinVar VUSs in low-MRSD genes reproduces the reported 25.8%-67.8% range, with the upper bound (67.8%, i.e., 5,086/7,507) corresponding to the least stringent model setting in at least one biosample.
- All '-' entries in the .results.mrsd.txt tables are treated as median junction coverage of 0 reads (not assayable), not dropped silently.
- Every gene in the variant table maps to an id1/id2 pair (GENCODE v19 symbol and Ensembl gene ID) in the MRSD tables; unmapped identifiers are reported, not silently excluded.
- Low-MRSD flag is TRUE iff MRSD <= 100 million reads in at least one of the four biosamples.
- mrsd_variant_fraction.csv contains a per-variant classification plus the final aggregate fraction.

## Limitations

- MRSD scores are most accurate for Illumina 75 bp paired-end poly-A enrichment RNA-seq workflows matching the GTEx sequencing protocol used to derive the scores.
- MRSD predictions may not reflect coverage in patients with widespread transcriptomic changes (e.g., interferonopathies, chromatin structure disorders, spliceosome disruption), and the model does not account for eQTLs or sQTLs.
- MRSD cannot be directly extended to predict suitability for detecting allele-specific expression biases or differential gene expression.

## Evidence

- [readme] Where '-' is given as a required read depth, median coverage of splice junctions for the given gene was 0 reads.: "Where '-' is given as a required read depth, median coverage of splice junctions for the given gene was 0 reads."
- [readme] Gene list format accepted by MRSDexplorer: "This file can have up to two columns, containing the GENCODE v19 gene symbol and/or the Ensembl gene ID."
- [abstract] Headline ClinVar VUS assayability finding: "up to 67.8% of the variants of uncertain significance in ClinVar that are predicted to impact splicing could be assayed by RNA sequencing in at least one of the biosamples"
- [methods] MRSDexplorer invocation parameters: "The `MRSDexplorer.py` script can then be used to generate MRSD scores"
