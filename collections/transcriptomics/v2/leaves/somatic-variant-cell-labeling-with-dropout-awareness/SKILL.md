---
name: somatic-variant-cell-labeling-with-dropout-awareness
description: Use when single-cell RNA-seq reads must be classified per cell as mutant
  or unknown for pre-called somatic SNVs, and one must reason about why wild-type
  calls are impossible.
when_to_use_negative:
- VarTrix is the better choice when one needs a maintained tool producing Cell Ranger-format
  genome matrices (sparse matrix of variant x cell) rather than cb_sniffer's barcode
  calls
- De novo variant discovery from the scRNA-seq reads themselves — reverse transcription
  noise gives a high false-positive rate; pre-called eWGS somatic SNVs are required
- Scenarios requiring definitive wild-type genotype calls per cell — dropout makes
  wild-type status indeterminable, so cells can only be labeled mutant or unknown
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: cb_sniffer
  role: Mutation barcode caller that parses CB barcodes from 10x cellranger BAMs and
    labels cells mutant/ref per somatic SNV with -f/-mq/-bq quality filters
  repo: https://github.com/sridnona/cb_sniffer
- name: pysam
  role: Python library used inside the sridnona/python3:180925.v1 Docker image to
    perform barcode- and UMI-aware pileups at variant positions
- name: Docker
  role: Containerization of the python3/pysam environment (image sridnona/python3:180925.v1)
    used for the pileup and labeling workflow
- name: 10XGenomics/vartrix
  role: Alternative/complementary tool that assigns single cells to pre-defined variants
    and outputs a genome matrix; noted in the repo as an approach for the same cell-genotyping
    task
  repo: https://github.com/10XGenomics/vartrix
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:4d3a860adfed5c3cbbde9c2c1f358f3e88b653ed20127972d3002a74f58af585
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sridnona/cb_sniffer
  tools:
  - 10XGenomics/vartrix
  - Docker
  - cb_sniffer
  - pysam
  - python3
  - sridnona/python3:180925.v1
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# somatic-variant-cell-labeling-with-dropout-awareness

## Summary

Labels individual cells in 10x scRNA-seq data as mutant or unknown (rather than wild-type) for eWGS-discovered somatic SNVs, using barcode/UMI-aware pileups, while explicitly treating transcript and allelic dropout as an unavoidable sensitivity limit.

## When to use

Use when single-cell RNA-seq reads must be classified per cell as mutant or unknown for pre-called somatic SNVs, and one must reason about why wild-type calls are impossible.

## When NOT to use

- VarTrix is the better choice when one needs a maintained tool producing Cell Ranger-format genome matrices (sparse matrix of variant x cell) rather than cb_sniffer's barcode calls
- De novo variant discovery from the scRNA-seq reads themselves — reverse transcription noise gives a high false-positive rate; pre-called eWGS somatic SNVs are required
- Scenarios requiring definitive wild-type genotype calls per cell — dropout makes wild-type status indeterminable, so cells can only be labeled mutant or unknown

## Inputs

- Cellranger-aligned 10x single cell BAM with CB cell barcode tags and UB_tag Chromium molecular barcode/UMI tags plus .bai index
- Somatic variant list from eWGS of paired tumor/normal samples (headered file: chrm, start, stop, ref, var, gene_name, trv_type)
- List of good cell barcodes file (barcodes.tsv from Cell Ranger)

## Outputs

- Per-cell genotype labels (mutant vs unknown) for each somatic SNV and sample
- Per-sample counts of mutant-labeled cells (e.g., _counts_UB.tsv of UB_tag read counts)
- Mutant and reference barcode calls per variant per sample (prefixed by UPN/sample name)

## How to apply

Discover somatic SNVs first with enhanced whole-genome sequencing (eWGS) of paired tumor/normal samples, then test each variant position in the 10x 5' scRNA-seq BAMs with cb_sniffer, which parses CB cell barcodes from cellranger-aligned reads and applies read-count (--filter, default 0), mapping-quality (--mapq, default 0), and base-quality (--baseq, default 1) thresholds. Classify each cell/variant pair as mutant when at least one variant-supporting read/UMI exists; otherwise label the cell unknown, because dropout (transcript and allelic) makes it impossible to determine whether a cell is truly wild-type. Recognize that dropout halves detection sensitivity and that end-biased Chromium coverage falls non-linearly across transcripts, so some variants are far more detectable than others. Aggregate results per sample (e.g., hundreds to thousands of mutant cells per AML case, ~22% of cells) and treat unmutated cells as uninformative rather than wild-type when interpreting subclonal structure.

## Related tools

- **cb_sniffer** (Mutation barcode caller that parses CB barcodes from 10x cellranger BAMs and labels cells mutant/ref per somatic SNV with -f/-mq/-bq quality filters) — https://github.com/sridnona/cb_sniffer
- **pysam** (Python library used inside the sridnona/python3:180925.v1 Docker image to perform barcode- and UMI-aware pileups at variant positions)
- **Docker** (Containerization of the python3/pysam environment (image sridnona/python3:180925.v1) used for the pileup and labeling workflow)
- **10XGenomics/vartrix** (Alternative/complementary tool that assigns single cells to pre-defined variants and outputs a genome matrix; noted in the repo as an approach for the same cell-genotyping task) — https://github.com/10XGenomics/vartrix

## Examples

```
python3 cb_sniffer.py sample.bam variants.tsv good_barcodes.tsv UPN1 -mq 0 -bq 1
```

## Evaluation signals

- Per-sample mutant-cell counts fall in the reported 'hundreds to thousands' range for the five AML cases (Table 1 values, e.g., 669, 604, 8200, 2003, 354; ~22% of total cells, ranging 396 to 8200)
- Average of ~49 mutant cells per variant across the variant set
- No cell is ever labeled wild-type; all non-mutant cells are unknown, consistent with the dropout rationale
- Output counts table (_counts_UB.tsv) contains per-cell UB_tag read counts consistent with 10x CB/UB_tag conventions
- cb_sniffer runs reproduce calls when the documented defaults (-f 0, -mq 0, -bq 1) or explicit filters are used on cellranger-aligned BAMs

## Limitations

- Dropout (transcript and allelic dropout) occurs on most scRNA-seq platforms and reduces the sensitivity of mutation detection by a factor of two
- It is impossible to determine whether a cell is truly wild-type for a given mutation, so unlabeled cells cannot be assigned genotype
- Partial transcript coverage on end-biased platforms such as Chromium limits sensitivity, and coverage drops non-linearly across transcript length so some variants are much more easily detectable than others
- Utility depends on the mutational composition of the sample; it will likely perform better for tumor types with higher mutation burdens than AML, and some observed expression heterogeneity without detectable genetic heterogeneity may stem from limited sensitivity

## Evidence

- [discussion] Wild-type calls are impossible due to dropout: "As a result, it is impossible to determine whether a cell is truly wild-type for a given mutation."
- [discussion] Dropout occurs broadly and halves sensitivity: "In addition, dropout reduces the sensitivity of mutation detection by a factor of two."
- [discussion] Dropout types on scRNA-seq platforms: "Dropout (including transcript dropout and allelic dropout) occurs with most scRNA-seq platforms."
- [readme] cb_sniffer purpose from README: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data"
- [abstract] Scale of mutant-cell labeling per AML sample: "we identify hundreds to thousands of cells containing tumor-specific mutations in each case"
- [intro] eWGS-first discovery strategy: "we first use “enhanced” whole-genome sequencing (eWGS) of paired tumor/normal samples to discover somatic mutations, and then focus on detecting those mutations in the scRNA-seq data"
