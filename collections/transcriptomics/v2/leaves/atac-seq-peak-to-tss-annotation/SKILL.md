---
name: atac-seq-peak-to-tss-annotation
description: 'Use when building a candidate TF–target GRN mask from ATAC-seq peaks:
  motif sites in peaks must be mapped to nearby genes and intersected with an expressed-gene
  set to seed regulatory network inference (e.g., scKINETICS priors).'
when_to_use_negative:
- You need distal enhancer–gene links beyond the −500 bp/+3000 bp TSS window — the
  TSS-anchored annotation will miss long-range peak-to-gene assignments
- You already have a curated TF–target prior or ChIP-seq-derived regulator set and
  do not need to derive one from ATAC-seq motifs
- Your target genes lack expression information in the matched single-cell dataset,
  since the mask is filtered to expressed genes only
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: MOODS
  role: Scans ATAC-seq peak sequences with CisBP PWMs to identify TF motif binding
    sites at P < 1e-10
- name: ChIPseeker
  role: Annotates motif-containing peaks to target genes' transcription start sites
    within 500 bp upstream / 3000 bp downstream
- name: scKINETICS
  role: Consumes the resulting candidate regulatory mask as its epigenetic prior for
    GRN and velocity inference
  repo: http://github.com/dpeerlab/scKINETICS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:99e61b92527dbec1adff4a3e9fe829a4b9bd1f5e7606fe33c6d72a5936f5d5f0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChIPseeker
  - MOODS
  - scKINETICS
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# atac-seq-peak-to-tss-annotation

## Summary

Constructs a binary candidate TF–target regulatory mask by scanning ATAC-seq peak sequences for TF motifs (MOODS, P < 1e-10) and annotating motif-containing peaks to genes near their transcription start sites (ChIPseeker, −500/+3000 bp TSS window), as used to build scKINETICS' epigenetic prior for the pancreatitis dataset.

## When to use

Use when building a candidate TF–target GRN mask from ATAC-seq peaks: motif sites in peaks must be mapped to nearby genes and intersected with an expressed-gene set to seed regulatory network inference (e.g., scKINETICS priors).

## When NOT to use

- You need distal enhancer–gene links beyond the −500 bp/+3000 bp TSS window — the TSS-anchored annotation will miss long-range peak-to-gene assignments
- You already have a curated TF–target prior or ChIP-seq-derived regulator set and do not need to derive one from ATAC-seq motifs
- Your target genes lack expression information in the matched single-cell dataset, since the mask is filtered to expressed genes only

## Inputs

- Bulk ATAC-seq peak set (e.g., 6575 healthy-pancreas peaks from Alonso-Curbelo et al. 2021)
- CisBP transcription factor motif database (position weight matrices)
- Reference genome/annotation for peak-to-TSS mapping (ChIPseeker)
- Expressed-gene list from the single-cell transcriptomic dataset

## Outputs

- Binary TF-by-gene candidate regulatory mask (candidate_regulatory_mask.csv)
- Summary statistics of the mask: number of genes, exclusive targets, TF count, fraction of nonzero entries

## How to apply

Start from the bulk ATAC-seq peak set (here, 6575 healthy-pancreas peaks from Alonso-Curbelo et al. 2021) and CisBP position weight matrices. Scan peak sequences with MOODS to call TF motif binding sites at significance threshold P < 1e-10. Annotate each motif-containing peak to its target gene's TSS with ChIPseeker, keeping peaks within 500 bp upstream to 3000 bp downstream of the TSS. Filter the resulting TF–target pairs to retain only target genes expressed in the single-cell transcriptomic dataset, then assemble the filtered assignments into a binary TF-by-gene candidate mask (candidate_regulatory_mask.csv) and compute summary statistics (gene counts, exclusive targets, TF count, fraction of nonzero entries) as a validation check.

## Related tools

- **MOODS** (Scans ATAC-seq peak sequences with CisBP PWMs to identify TF motif binding sites at P < 1e-10)
- **ChIPseeker** (Annotates motif-containing peaks to target genes' transcription start sites within 500 bp upstream / 3000 bp downstream)
- **scKINETICS** (Consumes the resulting candidate regulatory mask as its epigenetic prior for GRN and velocity inference) — http://github.com/dpeerlab/scKINETICS

## Examples

```
R: library(ChIPseeker); annotatePeak('motif_peaks.bed', tssRegion=c(-500, 3000), TxDb=txdb)  # peak-to-TSS annotation for the candidate regulatory mask
```

## Evaluation signals

- Rebuilt mask reproduces the reported statistics: 1876 union target/TF genes, 1291 exclusive targets, 585 TFs, and 21.6% nonzero entries
- Every retained target gene is expressed in the single-cell transcriptomic dataset (post-filter invariant)
- All motif calls pass the MOODS significance threshold P < 1e-10
- Mask is strictly binary (0/1 TF-by-gene entries) with rows = TFs and columns = target genes
- Peak-to-gene assignments fall within the TSS window of −500 bp upstream to +3000 bp downstream

## Limitations

- TSS-proximal annotation (−500/+3000 bp) cannot capture distal enhancer-mediated TF–target relationships
- The mask is limited to genes expressed in the matched single-cell dataset, excluding unexpressed putative targets
- Motif-based assignment links any TF whose PWM matches within the peak window, so candidate pairs may include non-functional binding

## Evidence

- [methods] Scan the ATAC-seq peak sequences with MOODS to identify TF motif binding sites, using the significance threshold P < 1e-10: [evidence span withheld — anchor status: not_found]
- [methods] Annotate motif-containing peaks to genes with ChIPseeker, mapping each peak to its target gene's transcription start site using a window of 500 bp upstream and 3000 bp downstream of the TSS: [evidence span withheld — anchor status: not_found]
- [readme] Demo covers input file generation and GRN construction: [evidence span withheld — anchor status: external_doc]
- [results] Validation statistics of the rebuilt mask: [evidence span withheld — anchor status: not_found]
