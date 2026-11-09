---
name: cisbp-motif-database-query
description: Use when you have a set of ATAC-seq/DNase peak sequences and need TF
  motif hits to construct a candidate TF–target regulatory mask as a prior for GRN
  inference.
when_to_use_negative:
- You already possess a TF–target prior mask or ChIP-seq–derived regulatory edges,
  so motif scanning is redundant.
- Your peaks lack assignable target genes (e.g., distal peaks with no TSS within the
  500 bp upstream / 3000 bp downstream annotation window), since filtering will discard
  them.
- You need expression dynamics rather than a static candidate regulator set — that
  is scKINETICS's EM estimation, not motif querying.
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3175
tools:
- name: MOODS
  role: Scans ATAC-seq peak sequences against CisBP position weight matrices to find
    TF motif sites at P < 1e-10
- name: ChIPseeker
  role: Annotates motif-containing peaks to target genes relative to the transcription
    start site (500 bp upstream, 3000 bp downstream window)
- name: scKINETICS
  role: Consumes the resulting binary TF–gene candidate mask as its epigenetic prior
    for GRN inference
  repo: http://github.com/dpeerlab/scKINETICS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:8f9483d03f98d9e518b8bea71c5430f465a99b7a414a1c9462ee354e82756bdc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChIPseeker
  - MOODS
  - scKINETICS
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cisbp-motif-database-query

## Summary

Retrieves CisBP transcription factor position weight matrices and scans them against open chromatin peak sequences (e.g., bulk ATAC-seq peaks from Alonso-Curbelo et al. 2021) with MOODS at a P < 1e-10 significance threshold, to identify candidate TF motif binding sites for building a prior GRN mask.

## When to use

Use when you have a set of ATAC-seq/DNase peak sequences and need TF motif hits to construct a candidate TF–target regulatory mask as a prior for GRN inference.

## When NOT to use

- You already possess a TF–target prior mask or ChIP-seq–derived regulatory edges, so motif scanning is redundant.
- Your peaks lack assignable target genes (e.g., distal peaks with no TSS within the 500 bp upstream / 3000 bp downstream annotation window), since filtering will discard them.
- You need expression dynamics rather than a static candidate regulator set — that is scKINETICS's EM estimation, not motif querying.

## Inputs

- Bulk ATAC-seq peak set / peak sequences (e.g., 6575 healthy-pancreas peaks from Alonso-Curbelo et al. 2021)
- CisBP transcription factor motif database (position weight matrices)

## Outputs

- Per-peak TF motif binding site calls at P < 1e-10 (MOODS scan results)
- Downstream binary TF-by-gene candidate regulatory mask (candidate_regulatory_mask.csv)

## How to apply

Obtain position weight matrices for transcription factors from the CisBP motif database, then scan the ATAC-seq peak sequences with MOODS using a strict significance threshold of P < 1e-10 to call TF motif binding sites. In the scKINETICS pancreatitis workflow this was applied to the 6575 healthy-pancreas bulk ATAC-seq peaks from Alonso-Curbelo et al. 2021; motif-containing peaks were subsequently annotated to genes with ChIPseeker (500 bp upstream to 3000 bp downstream of the TSS), filtered to expressed genes, and assembled into a binary TF-by-gene candidate mask (candidate_regulatory_mask.csv) whose reported statistics are 1876 genes, 1291 exclusive targets, 585 TFs, and 21.6% nonzero entries. The mask serves as the epigenetic prior that scKINETICS combines with gene-gene co-expression and phenotypic manifold constraints during expectation-maximization.

## Related tools

- **MOODS** (Scans ATAC-seq peak sequences against CisBP position weight matrices to find TF motif sites at P < 1e-10)
- **ChIPseeker** (Annotates motif-containing peaks to target genes relative to the transcription start site (500 bp upstream, 3000 bp downstream window))
- **scKINETICS** (Consumes the resulting binary TF–gene candidate mask as its epigenetic prior for GRN inference) — http://github.com/dpeerlab/scKINETICS

## Examples

```
python build_grn_mask.py --peaks atac_healthy_pancreas.bed --motifs cisbp_pwms/ --moods_pvalue 1e-10 --chipseeker_window "-500,3000" --expressed_genes sc_expr_genes.csv --output candidate_regulatory_mask.csv
```

## Evaluation signals

- Mask summary statistics reproduce the reported values: 1876 genes total, 1291 exclusive targets, 585 TFs, 21.6% nonzero entries.
- Number of TFs with motifs in the mask (585) does not exceed the number of CisBP PWMs scanned.
- All retained target genes are present in the expressed-gene set from the single-cell transcriptomic dataset.
- Output file candidate_regulatory_mask.csv is binary (nonzero entries only) and TF-by-gene shaped.
- MOODS scan threshold is consistently P < 1e-10 across all peaks.

## Limitations

- The mask is restricted to healthy-pancreas bulk ATAC-seq peaks; cell-state-specific or condition-specific accessible regions are not represented.
- TSS-window annotation (500 bp upstream / 3000 bp downstream) can misassign distal enhancer peaks, so the mask is a candidate prior, not validated regulatory edges.
- Motif similarity in CisBP can make TF assignments ambiguous for closely related TF families.

## Evidence

- [methods] Scan the ATAC-seq peak sequences with MOODS to identify TF motif binding sites, using the significance threshold P < 1e-10.: [evidence span withheld — anchor status: not_found]
- [methods] Obtain the bulk ATAC-seq peak set from Alonso-Curbelo et al. 2021 (n=6575 healthy-pancreas peaks) and the CisBP transcription factor motif database (position weight matrices).: [evidence span withheld — anchor status: not_found]
- [methods] Annotate motif-containing peaks to genes with ChIPseeker, mapping each peak to its target gene's transcription start site using a window of 500 bp upstream and 3000 bp downstream of the TSS.: [evidence span withheld — anchor status: not_found]
- [results] the GRN mask spans a union of 1876 target and TF genes, of which 1291 genes are exclusively targets and 585 are TFs: [evidence span withheld — anchor status: not_found]
- [intro] leveraging biologically-motivated priors from epigenetic data, gene-gene co-expression, and constraints on cells’ future states imposed by the phenotypic manifold: "leveraging biologically-motivated priors from epigenetic data, gene-gene co-expression, and constraints on cells’ future states imposed by the phenotypic manifold"
