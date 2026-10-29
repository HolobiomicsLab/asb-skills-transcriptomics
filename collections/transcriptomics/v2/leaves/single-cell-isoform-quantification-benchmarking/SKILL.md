---
name: single-cell-isoform-quantification-benchmarking
description: Use when comparing accuracy of single-cell/spatial Nanopore isoform quantification
  pipelines on simulated data with known ground-truth cell-by-isoform expression matrices,
  across sequencing quality or depth regimes.
when_to_use_negative:
- Input is real (non-simulated) single-cell Nanopore data without a known gold-standard
  isoform matrix — per-cell correlation to ground truth cannot be computed
- The goal is downstream alternative-splicing analysis (HVI identification, differential
  alternative splicing) rather than quantification-accuracy comparison — use the Longcell
  analysis package functions instead
- Short-read single-cell RNA-seq data, where Nanopore-specific truncation and error
  modeling does not apply
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: Longcell
  role: Method under evaluation; also the pipeline being benchmarked against competitors
    (adapter search, k-mer barcode matching k=6, minimap2 alignment, Louvain UMI clustering)
  repo: github.com/yuntianf/Longcell
- name: Sicelore2
  role: Competing isoform quantification method run with default parameters (v1),
    supplied the barcode whitelist
- name: FLAMES
  role: Competing isoform quantification method (git commit 774e16ae) run with default
    parameters and the barcode whitelist
- name: wf-single-cell
  role: Competing isoform quantification method run at v0.2.9 with default parameters
- name: IsoQuant
  role: Competing method (v3.3) fed the barcode-identified fastq from Sicelore2; also
    coupled with Longcell as Longcell+IsoQuant
- name: minimap2
  role: Splice-aware aligner (v2.24) used within Longcell against GRCh38 with the
    Gencode v39 junc-bed
- name: seqtk
  role: Read downsampling tool (v1.3) used at sampling rates 0.2–1 to emulate different
    sequencing depths
- name: igraph
  role: Louvain UMI clustering (v1.5.1) within the Longcell quantification pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:2134e7f879fb8be7a0d887867dcd732ad93559e40194473aaa90e808335e7ccb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yuntianf/Longcell
  tools:
  - FLAMES
  - IsoQuant
  - Longcell
  - Sicelore2
  - igraph
  - minimap2
  - seqtk
  - wf-single-cell
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# single-cell-isoform-quantification-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A simulation-based benchmarking protocol for evaluating single-cell Nanopore isoform quantification methods (Longcell vs Sicelore2, FLAMES, IsoQuant, wf-single-cell) using per-cell Spearman correlation against a known gold-standard isoform matrix and isoform-proportion MSE on a MYL6 NAGNAG simulation.

## When to use

Use when comparing accuracy of single-cell/spatial Nanopore isoform quantification pipelines on simulated data with known ground-truth cell-by-isoform expression matrices, across sequencing quality or depth regimes.

## When NOT to use

- Input is real (non-simulated) single-cell Nanopore data without a known gold-standard isoform matrix — per-cell correlation to ground truth cannot be computed
- The goal is downstream alternative-splicing analysis (HVI identification, differential alternative splicing) rather than quantification-accuracy comparison — use the Longcell analysis package functions instead
- Short-read single-cell RNA-seq data, where Nanopore-specific truncation and error modeling does not apply

## Inputs

- Gene expression matrix from the 10X CRCLM sample (basis for the simulated cell-by-isoform matrix)
- MOB Nanopore Visium sample data (basis for full-transcriptome simulation)
- GRCh38 reference genome with Gencode v39 GTF / splice-junction junc-bed
- 10x-style barcode whitelist
- Simulated FASTQ files at low/middle/high quality settings and seqtk downsampled depths (0.2–1)
- MYL6 4-isoform/240-cell NAGNAG simulation dataset

## Outputs

- Per-cell Spearman correlation between ground-truth and predicted cell-by-isoform matrices for each method and quality regime
- Isoform-proportion MSE values for the MYL6 NAGNAG simulation across the three error regimes
- benchmark_table.csv aggregating mean correlations (0.76/0.80/0.82 low/middle/high quality; 0.85/0.87 full transcriptome) and MYL6 MSE values (1.02/1.22/1.48) across methods

## How to apply

Simulate a cell-by-isoform expression matrix (e.g., 1000 isoforms from 187 highly variable genes of the 10X CRCLM gene expression matrix), generate full-length transcripts with 10x-style adapter, cell barcode and UMI at the 5' end, then layer realistic Nanopore artifacts: Bernoulli-geometric end truncation (probability 0.6, length ~ Geometric(0.05)), GC-dependent amplification (A = k*sigmoid(0.5*(G-50)) + 2, k = 20), a second pore-blocking truncation round, and four-state Markov-chain sequencing errors (correct/indel/del/wrong-base) with low/middle/high quality transition matrices. Build a full-transcriptome simulation from the MOB Nanopore Visium sample at high quality and downsample the FASTQ with seqtk v1.3 at rates 0.2–1 to test depth effects. Run Longcell (k-mer barcode matching with k=6, minimap2 v2.24 splice alignment vs GRCh38 with Gencode v39 junc-bed, igraph v1.5.1 Louvain UMI clustering, misalignment correction, scatter reduction) plus its IsoQuant-coupled variant, and competitors FLAMES (commit 774e16ae), Sicelore2 v1, wf-single-cell v0.2.9, and IsoQuant v3.3 with default parameters, supplying the barcode whitelist to FLAMES, Sicelore2, and Longcell. Fully join each method's output matrix with the ground truth by isoform, compute per-cell correlations, and compute isoform-proportion MSE for the MYL6 4-isoform/240-cell NAGNAG simulation; aggregate results into benchmark_table.csv.

## Related tools

- **Longcell** (Method under evaluation; also the pipeline being benchmarked against competitors (adapter search, k-mer barcode matching k=6, minimap2 alignment, Louvain UMI clustering)) — github.com/yuntianf/Longcell
- **Sicelore2** (Competing isoform quantification method run with default parameters (v1), supplied the barcode whitelist)
- **FLAMES** (Competing isoform quantification method (git commit 774e16ae) run with default parameters and the barcode whitelist)
- **wf-single-cell** (Competing isoform quantification method run at v0.2.9 with default parameters)
- **IsoQuant** (Competing method (v3.3) fed the barcode-identified fastq from Sicelore2; also coupled with Longcell as Longcell+IsoQuant)
- **minimap2** (Splice-aware aligner (v2.24) used within Longcell against GRCh38 with the Gencode v39 junc-bed)
- **seqtk** (Read downsampling tool (v1.3) used at sampling rates 0.2–1 to emulate different sequencing depths)
- **igraph** (Louvain UMI clustering (v1.5.1) within the Longcell quantification pipeline)

## Examples

```
devtools::install_github("yuntianf/Longcell",dependencies=TRUE); library(Longcell)
```

## Evaluation signals

- Reproduced mean per-cell Spearman correlations match the reported 0.76/0.80/0.82 (low/middle/high quality) and 0.85/0.87 (full transcriptome)
- MYL6 NAGNAG mean MSE values match the reported 1.02 (Longcell), 1.22, and 1.48 across the three error regimes, with Longcell attaining the lowest
- Each method's output cell-by-isoform matrix fully joins with the ground truth by isoform before correlation is computed
- Longcell ranks at or near the top of the compared methods in every quality and depth regime in benchmark_table.csv

## Limitations

- Results are simulation-based; simulated error and truncation models (Bernoulli-geometric truncation, GC-dependent amplification, four-state Markov errors) may not fully capture real Nanopore signal characteristics
- The low per-base Nanopore sequencing accuracy (85–99%) inflates UMI counts under fixed-edit-distance collapsing in competitor methods, which is part of the modeled behavior but complicates fair comparison
- Further research is required to reveal the biological mechanisms of intra-cell isoform switching; the benchmark measures quantification accuracy, not biological validity

## Evidence

- [discussion] we correct for read truncation and mapping errors by taking a "consensus" among reads assigned to the same UMI, which correspond to PCR duplicates: [evidence span withheld — anchor status: not_found]
- [readme] If you use Longcell for published work, please cite our manuscript: [evidence span withheld — anchor status: external_doc]
