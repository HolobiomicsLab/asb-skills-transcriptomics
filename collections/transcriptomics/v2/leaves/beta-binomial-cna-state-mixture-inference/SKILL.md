---
name: beta-binomial-cna-state-mixture-inference
description: Use when phased allelic counts (integer 'AD'/'DP' layers) are available
  for scRNA-seq cells and you need to infer per-cell, per-bin allele-imbalance CNA
  state probabilities that are robust to sparse, low-coverage variant counts.
when_to_use_negative:
- Input lacks allele-specific counts (only a total read-depth matrix is available)
  — use the RDR module instead, as InferCNV/CopyKAT-style RDR-only methods cannot
  detect copy-neutral LOH
- Data is bulk or pseudo-bulk at clone level rather than individual cells — XClone's
  beta-binomial mixture is designed to keep resolution at the single-cell level
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: XClone
  role: 'Runs the BAF module: beta-binomial mixture fitting plus HMM/KNN smoothing
    via xclone.model.run_BAF'
  repo: github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit that generates the allele-specific AD/DP matrices and
    total read depth matrix consumed by the BAF module
  repo: github.com/hxj5/xcltk
- name: scanpy
  role: Anndata-based single-cell framework in which the BAF anndata objects and layers
    are handled
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:0c38c2afa825d970564667c15fff718b0ce3eac4eea1e9f2c3c4aa77252e3679
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - XClone
  - scanpy
  - xcltk
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# beta-binomial-cna-state-mixture-inference

## Summary

Fits a beta-binomial mixture model over the allele-based CNA states (3 or 5 allele bias states) to model B-allele counts of each phased mega-gene feature in each cell within XClone's BAF module, then smooths state probabilities horizontally with an HMM along genome coordinates and vertically with a KNN cell-cell graph.

## When to use

Use when phased allelic counts (integer 'AD'/'DP' layers) are available for scRNA-seq cells and you need to infer per-cell, per-bin allele-imbalance CNA state probabilities that are robust to sparse, low-coverage variant counts.

## When NOT to use

- Input lacks allele-specific counts (only a total read-depth matrix is available) — use the RDR module instead, as InferCNV/CopyKAT-style RDR-only methods cannot detect copy-neutral LOH
- Data is bulk or pseudo-bulk at clone level rather than individual cells — XClone's beta-binomial mixture is designed to keep resolution at the single-cell level

## Inputs

- BAF anndata with integer allelic 'AD' and 'DP' layers (e.g. xclone.data.tnbc1_b() demo data or xclone.pp.xclonedata on AD.mtx/DP.mtx)
- Feature annotation (chr/start/stop/arm) loaded via xclone.pp.load_anno with genome_mode 'hg19_genes' or 'hg38_genes'
- XCloneConfig object with module='BAF' and phasing_region_key='chr', phasing_len=100

## Outputs

- BAF_merge_Xdata anndata carrying a bins-by-cells-by-states probability tensor over all 5 configured allele bias states, saved as BAF_merge_Xdata.h5ad

## How to apply

After XClone's three-step haplotyping (SNP-to-gene population phasing, EM-based phasing of ~100 consecutive genes into gene bins, and dynamic-programming phasing of gene bins to chromosome arms), configure xclone.XCloneConfig(module='BAF') with theo_neutral_BAF=0.5, concentration=35.5 (lower=20, upper=100), CNV_N_components=5, start_prob=[0.2, 0.15, 0.3, 0.15, 0.2], trans_t=1e-6 with the matching 5x5 trans_prob, and BAF_denoise=True. Fit the beta-binomial mixture over the 5 allele bias states to model B-allele counts of each phased mega-gene feature in each cell. Smooth the resulting state probabilities horizontally with the hidden Markov model along genome coordinates and vertically with the KNN cell-cell connectivity graph, yielding a bins-by-cells-by-states probability tensor. Execute xclone.model.run_BAF(BAF_adata, config_file=xconfig) and save the returned BAF_merge_Xdata as an .h5ad file.

## Related tools

- **XClone** (Runs the BAF module: beta-binomial mixture fitting plus HMM/KNN smoothing via xclone.model.run_BAF) — github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit that generates the allele-specific AD/DP matrices and total read depth matrix consumed by the BAF module) — github.com/hxj5/xcltk
- **scanpy** (Anndata-based single-cell framework in which the BAF anndata objects and layers are handled)

## Examples

```
BAF_merge_Xdata = xclone.model.run_BAF(BAF_adata, config_file=xconfig)
```

## Evaluation signals

- The returned BAF_merge_Xdata carries a probability tensor over all 5 configured states for every bin and cell (probabilities per bin-cell sum to 1)
- Genome coordinates of bins follow the phased chromosome-arm structure produced by the three-step haplotyping (phasing_len=100 gene bins collapsed to chromosome arms)
- HMM smoothing respects trans_t=1e-6 transition sparsity: adjacent bins rarely flip between distant CNA states
- Output can be reloaded as BAF_merge_Xdata.h5ad and matched to the demo results in the xclone-data repository (e.g. TNBC1 widespread LOH)

## Limitations

- HoneyBADGER and CaSpER-style independent use of individual SNPs suffers from high noise because coverage of a single variant is extremely low; the beta-binomial mixture must therefore operate on aggregated phased mega-gene features, not raw SNPs
- Allele-based states depend on accurate phasing; mis-phased genes propagate to chromosome-arm mega-features
- XClone is being updated and improved frequently at this stage, so parameters (e.g. concentration bounds 20–100) may change across versions; check the release history

## Evidence

- [intro] Three-step haplotyping in the BAF module: "we implemented a three-step haplotyping, from individual SNPs to a gene by population-based phasing, from consecutive genes (100 by default) to a gene bin by an Expectation-Maximization algorithm"
- [intro] HMM/KNN smoothing: "horizontally with hidden Markov models (HMM) along the genome coordinates and vertically with a k-nearest-neighbors (KNN) cell-cell connectivity graph"
- [methods] run_BAF invocation: "BAF_merge_Xdata = xclone.model.run_BAF(BAF_adata,
                config_file = xconfig)"
- [readme] xcltk input matrices: "It takes three matrices as input: the allele-specific *AD* and *DP* matrices
(BAF signals) and the *total read depth* matrix (RDR signals)."
- [intro] Low coverage rationale: "given the coverage of a single variant is extremely low, their independent use of individual single nucleotide polymorphisms (SNPs) often suffers from high noise."
