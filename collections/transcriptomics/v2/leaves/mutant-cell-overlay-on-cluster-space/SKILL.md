---
name: mutant-cell-overlay-on-cluster-space
description: Use when a set of cells has been genotyped for a somatic SNV from scRNA-seq
  (via cb_sniffer/VarTrix on a 10x Chromium BAM) and you need to project that genotype
  onto the whole-sample expression embedding/clustering to ask which expression programs
  associate with the mutant subclone.
when_to_use_negative:
- 'You need per-cell genotype certainty: dropout makes it impossible to call a cell
  truly wild-type, so treat unmutated labels as ''not detected'' rather than confirmed
  wild-type'
- Mutations lie in transcript regions with poor coverage on end-biased Chromium platforms,
  where detection sensitivity is limited and the overlay label will be sparse
- The sample's tumor type is a low-mutation-burden cancer like AML where few expressed
  SNVs are available to define a subclone
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: python3
  role: Loads the _counts_UB.tsv count matrix and t-SNE/cluster coordinates, overlays
    genotype labels, and computes local mutant-cell density per cell
  repo: https://github.com/genome/scrna_mutations
- name: pysam
  role: Used upstream to parse the 10x cellranger BAM and call mutant/ref barcodes
    feeding the genotype label list
  repo: https://github.com/sridnona/cb_sniffer
- name: Docker (sridnona/python3:180925.v1)
  role: Container image providing the python3/pysam environment for the genotyping
    and overlay analysis
- name: cb_sniffer
  role: Mutation barcode caller producing the mutant and ref barcode lists that are
    overlaid onto the cluster space
  repo: https://github.com/sridnona/cb_sniffer
- name: VarTrix
  role: Alternative tool to assign single-cell variant labels from the 10x BAM for
    overlaying on expression clustering
  repo: https://github.com/10XGenomics/vartrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:e324355d5b4d02e6939121f3acfc90ed13d49115c31db5025ec8bed7d192a15d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genome/scrna_mutations
  tools:
  - Docker
  - Docker (sridnona/python3:180925.v1)
  - VarTrix
  - cb_sniffer
  - pysam
  - python3
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mutant-cell-overlay-on-cluster-space

## Summary

Overlays cells genotyped as expressing a somatic mutation (e.g. GATA2R361C) as a binary/graded label onto the t-SNE/cluster space of the full sample and computes a local mutant-cell density per cell, enabling genotype-to-phenotype association for non-genotyped cells.

## When to use

Use when a set of cells has been genotyped for a somatic SNV from scRNA-seq (via cb_sniffer/VarTrix on a 10x Chromium BAM) and you need to project that genotype onto the whole-sample expression embedding/clustering to ask which expression programs associate with the mutant subclone.

## When NOT to use

- You need per-cell genotype certainty: dropout makes it impossible to call a cell truly wild-type, so treat unmutated labels as 'not detected' rather than confirmed wild-type
- Mutations lie in transcript regions with poor coverage on end-biased Chromium platforms, where detection sensitivity is limited and the overlay label will be sparse
- The sample's tumor type is a low-mutation-burden cancer like AML where few expressed SNVs are available to define a subclone

## Inputs

- UMI count matrix for the sample derived from Chromium UB_tag read counts (_counts_UB.tsv-style count table)
- Precomputed t-SNE or cluster coordinates for all cells of the sample
- List of cell barcodes identified as expressing the target mutation (mutant-cell genotype labels, e.g. from cb_sniffer or VarTrix)

## Outputs

- Cluster/embedding space plot or table with mutant cells overlaid as binary/graded labels
- Per-cell local mutant-cell density values across the full sample (including non-genotyped cells)

## How to apply

Load the UMI count matrix for the sample (the _counts_UB.tsv-style table of Chromium molecular barcode/UB_tag read counts) together with the precomputed t-SNE/cluster coordinates using python3. Load the list of barcodes identified as expressing the mutation of interest (e.g. GATA2R361C in sample 721214) and overlay these mutant cells as a binary or graded label onto the t-SNE/cluster space of the full sample, deliberately including the non-genotyped cells. For every cell in the embedded/clustered space, compute a local mutant-cell density value (e.g. the fraction or count of mutant-labeled cells in its local neighborhood/cluster). This density variable becomes the regressor for downstream per-gene regression of expression, controlling for covariates such as expression of the mutated gene (GATA2). The rationale, per the article, is that genotyped cells analyzed only in isolation discard information: 'we analyzed them in the context of the entire sample, thereby leveraging the expression information provided by the additional, non-genotyped cells.'

## Related tools

- **python3** (Loads the _counts_UB.tsv count matrix and t-SNE/cluster coordinates, overlays genotype labels, and computes local mutant-cell density per cell) — https://github.com/genome/scrna_mutations
- **pysam** (Used upstream to parse the 10x cellranger BAM and call mutant/ref barcodes feeding the genotype label list) — https://github.com/sridnona/cb_sniffer
- **Docker (sridnona/python3:180925.v1)** (Container image providing the python3/pysam environment for the genotyping and overlay analysis)
- **cb_sniffer** (Mutation barcode caller producing the mutant and ref barcode lists that are overlaid onto the cluster space) — https://github.com/sridnona/cb_sniffer
- **VarTrix** (Alternative tool to assign single-cell variant labels from the 10x BAM for overlaying on expression clustering) — https://github.com/10XGenomics/vartrix

## Examples

```
python3 cb_sniffer.py bam_file variant_file barcodes.tsv 721214  # then in python3: overlay the resulting mutant barcodes onto t-SNE coords and compute per-cell local mutant-cell density
```

## Evaluation signals

- The overlaid mutant barcodes correspond one-to-one with barcodes in the genotype output (cb_sniffer/vartrix label file) and with barcodes present in the _counts_UB.tsv matrix
- Every cell in the sample receives a defined local mutant-cell density value, including non-genotyped cells
- Overlay on the t-SNE shows mutant cells spatially concentrated (subclone-like localization) rather than uniformly scattered, consistent with a distinct subclone
- Downstream regression on the density variable should recover VIM as the top-ranked gene at FDR-adjusted p < 0.05 for the GATA2R361C case, with immune response, apoptosis, and leukocyte adhesion genes enriched
- Density computation is invariant to the ordering of cells in the input table

## Limitations

- Dropout (transcript and allelic dropout) occurs with most scRNA-seq platforms and reduces mutation-detection sensitivity about twofold, so mutant labels are incomplete and it is impossible to determine whether a cell is truly wild-type
- Partial transcript coverage on end-biased platforms such as Chromium limits variant-detection sensitivity, and coverage drops non-linearly across transcript length, so some variants yield much sparser overlays than others
- The approach depends on the specific mutational composition of the sample and likely performs better for tumor types with higher mutation burdens than AML

## Evidence

- [discussion] Analyze genotyped cells in the context of the entire sample to leverage expression information from non-genotyped cells: "we analyzed them in the context of the entire sample, thereby leveraging the expression information provided by the additional, non-genotyped cells"
- [methods] UB_tag count table as input: "read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv"
- [readme] VarTrix use case of overlaying variant labels on expression clustering: [evidence span withheld — anchor status: external_doc]
- [discussion] Dropout limits genotype certainty: "As a result, it is impossible to determine whether a cell is truly wild-type for a given mutation."
- [abstract] Expression signatures associated with subclonal mutations: "identify expression signatures associated with subclonal mutations, and find cell surface markers that could be used to purify subclones for further study"
