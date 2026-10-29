---
name: gene-expression-filtering-of-network-priors
description: Use when you have a motif- and TSS-derived candidate TF–target mask built
  from ATAC-seq peaks and need to restrict it to genes detectably expressed in your
  scRNA-seq dataset prior to EM-based GRN inference.
when_to_use_negative:
- You have no accompanying single-cell transcriptomic dataset to define the expressed-gene
  universe — the filter step is undefined
- Your prior mask is already gene-level and restricted to expressed genes; re-filtering
  adds nothing
- You want data-driven GRN edges without an epigenetic/motif prior — scKINETICS supports
  co-expression and manifold constraints instead
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MOODS
  role: scans ATAC-seq peak sequences against CisBP PWMs to identify TF motif binding
    sites at P < 1e-10, generating the TF–target pairs that are subsequently expression-filtered
- name: ChIPseeker
  role: annotates motif-containing peaks to target genes by mapping each peak to its
    target gene's TSS within 500 bp upstream / 3000 bp downstream
- name: scKINETICS
  role: consumes the expression-filtered candidate regulatory mask as the epigenetic
    prior during EM-based velocity and GRN inference
  repo: http://github.com/dpeerlab/scKINETICS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:319a2b5b0aed1637ed48555e11b42c13eddb6a35d01d8cbd2dc42b6e612a51b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChIPseeker
  - MOODS
  - scKINETICS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# gene-expression-filtering-of-network-priors

## Summary

Filters a candidate TF–target regulatory prior mask so it only spans genes expressed in the accompanying single-cell transcriptomic dataset, before feeding it to scKINETICS as an epigenetic prior. This keeps the learned GRN constrained to biologically plausible, observable regulator–target interactions.

## When to use

Use when you have a motif- and TSS-derived candidate TF–target mask built from ATAC-seq peaks and need to restrict it to genes detectably expressed in your scRNA-seq dataset prior to EM-based GRN inference.

## When NOT to use

- You have no accompanying single-cell transcriptomic dataset to define the expressed-gene universe — the filter step is undefined
- Your prior mask is already gene-level and restricted to expressed genes; re-filtering adds nothing
- You want data-driven GRN edges without an epigenetic/motif prior — scKINETICS supports co-expression and manifold constraints instead

## Inputs

- Binary TF-by-gene candidate regulatory mask from MOODS motif scanning (P < 1e-10) plus ChIPseeker TSS mapping (500 bp upstream, 3000 bp downstream)
- List of genes expressed in the single-cell transcriptomic (scRNA-seq) dataset
- Bulk ATAC-seq peak set (e.g. 6575 healthy-pancreas peaks from Alonso-Curbelo et al. 2021)
- CisBP transcription factor motif PWM database

## Outputs

- candidate_regulatory_mask.csv: expression-filtered binary TF–target prior mask
- Mask summary statistics: 1876 genes, 1291 exclusive targets, 585 TFs, 21.6% nonzero entries

## How to apply

First build the candidate mask: scan bulk ATAC-seq peak sequences (e.g. the 6575 healthy-pancreas peaks from Alonso-Curbelo et al. 2021) with MOODS against CisBP position weight matrices at a strict threshold of P < 1e-10, then map motif-containing peaks to genes with ChIPseeker using a TSS window of 500 bp upstream and 3000 bp downstream. Next intersect the resulting target-gene list with the genes expressed in the single-cell transcriptomic dataset, retaining only expressed targets; TFs themselves may remain as rows regardless. Assemble the filtered assignments into a binary TF-by-gene matrix (candidate_regulatory_mask.csv) and compute summary statistics as a validity check. The rationale is that scKINETICS learns regulator impacts only over its prior-supported interactions, so an unfiltered mask would admit targets whose expression cannot be modeled from the single-cell data.

## Related tools

- **MOODS** (scans ATAC-seq peak sequences against CisBP PWMs to identify TF motif binding sites at P < 1e-10, generating the TF–target pairs that are subsequently expression-filtered)
- **ChIPseeker** (annotates motif-containing peaks to target genes by mapping each peak to its target gene's TSS within 500 bp upstream / 3000 bp downstream)
- **scKINETICS** (consumes the expression-filtered candidate regulatory mask as the epigenetic prior during EM-based velocity and GRN inference) — http://github.com/dpeerlab/scKINETICS

## Examples

```
annotatePeak(peak_set, tssRegion = c(-500, 3000), TxDb = txdb)  # ChIPseeker TSS mapping before expression filtering
```

## Evaluation signals

- The rebuilt filtered mask must reproduce the reported statistics: 1876 total genes, 1291 exclusive targets, 585 TFs
- Nonzero (positive) entries in the binary mask should equal ~21.6% of the TF-by-gene matrix
- Every gene retained as a target must be present in the expressed-gene list from the single-cell dataset
- No TF–target pair in the mask should correspond to a peak whose nearest-TSS assignment falls outside the -500 bp / +3000 bp window
- All motif calls underlying the mask must come from MOODS scans at the P < 1e-10 threshold

## Limitations

- TSS-proximal mapping (500 bp upstream / 3000 bp downstream) may miss distal enhancer–gene relationships, limiting the prior's reach
- The filter restricts the mask to genes captured in the single-cell dataset; targets absent from the scRNA-seq are dropped from the candidate network
- No changelog found for the scKINETICS repository, so tool version drift in mask-reproduction scripts is hard to track

## Evidence

- [methods] Filter the peak-to-gene assignments to retain only target genes that are expressed in the single-cell transcriptomic dataset.: [evidence span withheld — anchor status: not_found]
- [methods] confirm the rebuilt mask reproduces the reported statistics: [evidence span withheld — anchor status: not_found]
- [methods] MOODS threshold: [evidence span withheld — anchor status: not_found]
- [abstract] epigenetic priors in scKINETICS: [evidence span withheld — anchor status: not_found]
- [readme] GRN construction in demo: [evidence span withheld — anchor status: external_doc]
