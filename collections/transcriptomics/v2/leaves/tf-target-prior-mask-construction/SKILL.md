---
name: tf-target-prior-mask-construction
description: Use when building a candidate TF-target prior network from ATAC-seq peak
  sets (e.g. scKINETICS epigenetic prior), where TF motif scan results must be converted
  to a binary TF-by-gene mask restricted to expressed genes.
when_to_use_negative:
- Target genes cannot be cross-referenced with a single-cell expression dataset to
  filter for expressed genes, since the mask is intended to be restricted to expressed
  genes
- Only single-cell ATAC data without a compatible bulk peak set is available, as the
  described pipeline operates on bulk ATAC-seq peaks
- The goal is data-driven GRN inference without epigenetic priors, e.g. inference
  relying only on gene-gene co-expression or phenotypic manifold constraints
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MOODS
  role: Scans ATAC-seq peak sequences against CisBP position-weight matrices to find
    TF motif binding sites at P < 1e-10
- name: ChIPseeker
  role: Annotates motif-containing peaks to target genes by mapping peaks to TSSs
    within 500 bp upstream / 3000 bp downstream
- name: scKINETICS
  role: 'Downstream consumer: the binary TF-by-gene mask is supplied as the epigenetic
    prior for GRN and velocity inference via expectation-maximization'
  repo: http://github.com/dpeerlab/scKINETICS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:62f6e0997ce4ac683cc60b303e634915aa7c8240fdbd5c1ed89082fdd40745c2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChIPseeker
  - MOODS
  - scKINETICS
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# tf-target-prior-mask-construction

## Summary

Construct a binary TF-by-gene candidate regulatory mask to serve as the epigenetic prior for scKINETICS, by scanning bulk ATAC-seq peaks with TF position-weight matrices and mapping motif-containing peaks to genes near their TSS. This grounds GRN inference in chromatin accessibility rather than co-expression alone.

## When to use

Use when building a candidate TF-target prior network from ATAC-seq peak sets (e.g. scKINETICS epigenetic prior), where TF motif scan results must be converted to a binary TF-by-gene mask restricted to expressed genes.

## When NOT to use

- Target genes cannot be cross-referenced with a single-cell expression dataset to filter for expressed genes, since the mask is intended to be restricted to expressed genes
- Only single-cell ATAC data without a compatible bulk peak set is available, as the described pipeline operates on bulk ATAC-seq peaks
- The goal is data-driven GRN inference without epigenetic priors, e.g. inference relying only on gene-gene co-expression or phenotypic manifold constraints

## Inputs

- Bulk ATAC-seq peak set (e.g. 6575 healthy-pancreas peaks from Alonso-Curbelo et al. 2021)
- TF position-weight matrices from the CisBP transcription factor motif database
- Single-cell transcriptomic dataset used to define the set of expressed genes
- Gene annotation (TSS coordinates) for peak-to-gene mapping with ChIPseeker

## Outputs

- Binary TF-by-gene candidate regulatory mask (candidate_regulatory_mask.csv) with 1876 genes, 1291 exclusive targets, 585 TFs, and 21.6% nonzero entries
- Summary statistics of the mask (number of genes, exclusive targets, TFs, fraction of nonzero entries)

## How to apply

Start from a bulk ATAC-seq peak set (in the scKINETICS acute pancreatitis application, 6575 healthy-pancreas peaks from Alonso-Curbelo et al. 2021) and TF position-weight matrices from the CisBP motif database. Scan peak sequences with MOODS to identify TF motif binding sites at significance P < 1e-10. Annotate motif-containing peaks to genes with ChIPseeker, mapping each peak to the transcription start site of its target gene within a window of 500 bp upstream and 3000 bp downstream of the TSS. Filter the resulting peak-to-gene assignments to retain only target genes expressed in the single-cell transcriptomic dataset, then assemble the TF-target assignments into a binary TF-by-gene candidate mask. Validate the rebuilt mask against reported summary statistics (1876 genes, 1291 exclusive targets, 585 TFs, 21.6% nonzero entries) before saving it as candidate_regulatory_mask.csv for use in scKINETICS EM estimation.

## Related tools

- **MOODS** (Scans ATAC-seq peak sequences against CisBP position-weight matrices to find TF motif binding sites at P < 1e-10)
- **ChIPseeker** (Annotates motif-containing peaks to target genes by mapping peaks to TSSs within 500 bp upstream / 3000 bp downstream)
- **scKINETICS** (Downstream consumer: the binary TF-by-gene mask is supplied as the epigenetic prior for GRN and velocity inference via expectation-maximization) — http://github.com/dpeerlab/scKINETICS

## Evaluation signals

- Rebuilt mask reproduces the reported union of 1876 target and TF genes
- Mask contains 1291 exclusively target genes and 585 TFs
- Fraction of nonzero entries equals 21.6%
- Output file candidate_regulatory_mask.csv exists with a binary TF-by-gene schema (nonzero entries restricted to TF rows)
- All retained target genes are expressed in the single-cell transcriptomic dataset

## Limitations

- The TSS-based peak-to-gene window (500 bp upstream, 3000 bp downstream) misses distal enhancer-promoter assignments, so the mask is a restricted candidate set rather than an exhaustive regulatory map
- Masks built from bulk ATAC-seq (e.g. healthy-pancreas peaks) may not capture cell-state-specific accessibility in the single-cell data
- Only expressed target genes are retained, so regulators of poorly expressed or unmeasured genes are excluded from the prior
- The scKINETICS repository has no changelog, so version-specific behavior of mask-input handling is not documented

## Evidence

- [abstract] leveraging biologically-motivated priors from epigenetic data, gene–gene coexpression, and constraints on cells' future states imposed by the phenotypic manifold: [evidence span withheld — anchor status: not_found]
- [methods] Scan the ATAC-seq peak sequences with MOODS to identify TF motif binding sites, using the significance threshold P < 1e-10.: [evidence span withheld — anchor status: not_found]
- [methods] Annotate motif-containing peaks to genes with ChIPseeker, mapping each peak to its target gene's transcription start site using a window of 500 bp upstream and 3000 bp downstream of the TSS.: [evidence span withheld — anchor status: not_found]
- [readme] The demo shows the main functionalities of the package including input file generation, GRN construction, EM estimation, visualizations, and the TF ablation experiment.: [evidence span withheld — anchor status: external_doc]
- [results] the GRN mask spans a union of 1876 target and TF genes, of which 1291 genes are exclusively targets and 585 are TFs: [evidence span withheld — anchor status: not_found]
