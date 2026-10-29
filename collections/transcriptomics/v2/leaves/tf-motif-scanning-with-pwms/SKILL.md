---
name: tf-motif-scanning-with-pwms
description: Use when building a candidate TF–target regulatory mask from bulk ATAC-seq
  peaks and a PWM motif database (e.g. CisBP), to serve as an epigenetic prior for
  GRN inference methods such as scKINETICS.
when_to_use_negative:
- The input is already a curated TF–target interaction list or a pre-built binary
  GRN mask — no motif scanning or peak-to-gene mapping is needed.
- You lack peak sequences or genomic coordinates (e.g. only differential expression
  data), since motif scanning requires ATAC-seq peak DNA sequence.
- You need a validated or literature-supported GRN rather than a permissive candidate
  prior mask for subsequent EM-based inference.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3308
tools:
- name: MOODS
  role: Scans ATAC-seq peak sequences with CisBP PWMs to identify TF motif binding
    sites at significance threshold P < 1e-10
- name: ChIPseeker
  role: Annotates motif-containing peaks to target genes by mapping each peak to its
    target gene's TSS within a 500 bp upstream / 3000 bp downstream window
- name: scKINETICS
  role: Downstream consumer of the candidate regulatory mask as an epigenetic prior
    for EM-based GRN and velocity inference
  repo: http://github.com/dpeerlab/scKINETICS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:0de9db2da7a5de5be21bc28c8e3f326f76d185f62fb11322c843254adc9e7a8b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChIPseeker
  - MOODS
  - scKINETICS
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# TF motif scanning with PWMs

## Summary

Construct a candidate TF–target gene regulatory network mask by scanning ATAC-seq peak sequences with position weight matrices from CisBP, then mapping motif-containing peaks to target genes. This provides the epigenetic prior mask that scKINETICS uses to constrain its expectation-maximization inference of regulator-target impacts.

## When to use

Use when building a candidate TF–target regulatory mask from bulk ATAC-seq peaks and a PWM motif database (e.g. CisBP), to serve as an epigenetic prior for GRN inference methods such as scKINETICS.

## When NOT to use

- The input is already a curated TF–target interaction list or a pre-built binary GRN mask — no motif scanning or peak-to-gene mapping is needed.
- You lack peak sequences or genomic coordinates (e.g. only differential expression data), since motif scanning requires ATAC-seq peak DNA sequence.
- You need a validated or literature-supported GRN rather than a permissive candidate prior mask for subsequent EM-based inference.

## Inputs

- Bulk ATAC-seq peak set (e.g. n=6575 healthy-pancreas peaks from Alonso-Curbelo et al. 2021)
- CisBP transcription factor motif database (position weight matrices)
- Genome assembly reference for peak sequence extraction
- Gene expression table or cell-by-gene matrix from a single-cell transcriptomic dataset (for expressed-gene filtering)

## Outputs

- Binary TF-by-gene candidate regulatory mask (candidate_regulatory_mask.csv)
- Summary statistics of the mask (number of genes, exclusive targets, TFs, fraction of nonzero entries)

## How to apply

Obtain the bulk ATAC-seq peak set (in the pancreatitis study, 6575 healthy-pancreas peaks from Alonso-Curbelo et al. 2021) and the CisBP transcription factor motif database of position weight matrices. Scan the ATAC-seq peak sequences with MOODS to identify TF motif binding sites, using the significance threshold P < 1e-10. Annotate motif-containing peaks to genes with ChIPseeker, mapping each peak to its target gene's transcription start site using a window of 500 bp upstream and 3000 bp downstream of the TSS. Filter the peak-to-gene assignments to retain only target genes expressed in the accompanying single-cell transcriptomic dataset, then assemble the filtered TF–target assignments into a binary TF-by-gene candidate mask and compute summary statistics.

## Related tools

- **MOODS** (Scans ATAC-seq peak sequences with CisBP PWMs to identify TF motif binding sites at significance threshold P < 1e-10)
- **ChIPseeker** (Annotates motif-containing peaks to target genes by mapping each peak to its target gene's TSS within a 500 bp upstream / 3000 bp downstream window)
- **scKINETICS** (Downstream consumer of the candidate regulatory mask as an epigenetic prior for EM-based GRN and velocity inference) — http://github.com/dpeerlab/scKINETICS

## Evaluation signals

- The rebuilt mask reproduces the reported statistics: 1876 genes in the union of TFs and targets, 1291 exclusive targets, 585 TFs, and 21.6% nonzero entries.
- The mask is saved as candidate_regulatory_mask.csv in binary TF-by-gene format.
- All retained target genes are expressed in the single-cell transcriptomic dataset (no unexpressed genes pass the filter).
- Every motif hit arises from a MOODS scan with P < 1e-10 against CisBP PWMs — hits at looser thresholds indicate parameter drift.
- Peak-to-gene assignments fall within the TSS-anchored annotation window (500 bp upstream, 3000 bp downstream).

## Limitations

- The mask is a candidate prior, not a validated network — scKINETICS refines it via expectation-maximization; motif-based edges may include non-functional binding sites.
- The TSS-proximity mapping (500 bp upstream / 3000 bp downstream) ignores distal enhancer–gene relationships.
- The TSS-proximity window and P < 1e-10 threshold are dataset-specific choices tuned in the pancreatitis study and may need adjustment for other tissues or peak sets.

## Evidence

- [methods] Scan the ATAC-seq peak sequences with MOODS to identify TF motif binding sites, using the significance threshold P < 1e-10: [evidence span withheld — anchor status: not_found]
- [methods] ChIPseeker TSS-anchored annotation window: [evidence span withheld — anchor status: not_found]
- [results] Validated mask statistics: [evidence span withheld — anchor status: not_found]
- [abstract] Epigenetic prior motivation: [evidence span withheld — anchor status: not_found]
