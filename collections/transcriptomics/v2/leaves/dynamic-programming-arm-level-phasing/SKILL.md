---
name: dynamic-programming-arm-level-phasing
description: Reference for reconstructing XClone's BAF phasing hierarchy when a workflow
  must collapse gene-bin-level B-allele counts to chromosome-arm mega-features before
  beta-binomial CNA-state calling.
when_to_use_negative:
- The input is already a gene-expression count matrix without allelic AD/DP layers
  — use the RDR module instead, since BAF phasing requires heterozygous variant counts.
- Per-SNP or per-gene allelic calls are needed at high resolution rather than arm-level
  CNA states — dynamic-programming arm phasing sacrifices local resolution to gain
  signal.
- Variants lack population or reference-based phasing information, since this step
  operates on already-phased gene bins, not raw unphased SNPs.
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3518
tools:
- name: XClone
  role: Implements the BAF module's three-step haplotyping (population SNP-to-gene
    phasing, EM local phasing of 100-gene bins, dynamic programming to chromosome
    arms) and the beta-binomial HMM/KNN model via xclone.model.run_BAF
  repo: https://github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit that generates the allele-specific AD/DP matrices consumed
    by the BAF phasing pipeline from SAM/BAM/CRAM files
  repo: https://github.com/hxj5/xcltk
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:e6b8d8cf21b63ccaceec9d88c15cc8b166bc9bb4ddcdec5bce09498bedc425fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/XClone
  tools:
  - XClone
  - scanpy
  - xcltk
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Dynamic programming arm-level phasing

## Summary

The global-phasing step of XClone's BAF module aggregates allele-imbalance signals from EM-phased gene bins (100 genes by default) up to whole chromosome arms using dynamic programming, boosting haplotype-aware CNA signal from sparse scRNA-seq allelic counts.

## When to use

Reference for reconstructing XClone's BAF phasing hierarchy when a workflow must collapse gene-bin-level B-allele counts to chromosome-arm mega-features before beta-binomial CNA-state calling.

## When NOT to use

- The input is already a gene-expression count matrix without allelic AD/DP layers — use the RDR module instead, since BAF phasing requires heterozygous variant counts.
- Per-SNP or per-gene allelic calls are needed at high resolution rather than arm-level CNA states — dynamic-programming arm phasing sacrifices local resolution to gain signal.
- Variants lack population or reference-based phasing information, since this step operates on already-phased gene bins, not raw unphased SNPs.

## Inputs

- BAF anndata with integer allelic 'AD' and 'DP' layers (e.g. xclone.data.tnbc1_baf() demo data, or AD.mtx/DP.mtx from xcltk with barcodes)
- Feature annotation with chr/start/stop/arm loaded via xclone.pp.load_anno (genome_mode 'hg19_genes' or 'hg38_genes')
- XCloneConfig(module='BAF') with phasing_region_key='chr', phasing_len=100, theo_neutral_BAF=0.5, concentration=35.5, CNV_N_components=5

## Outputs

- BAF_merge_Xdata (.h5ad) carrying a bins-by-cells-by-states probability tensor over the 5 configured allele-imbalance CNA states for every chromosome-arm mega-gene feature and cell

## How to apply

After SNP-to-gene population phasing and EM-based local phasing of consecutive genes (phasing_len=100) into gene bins, run the global phasing step with dynamic programming to aggregate gene bins into chromosome-arm mega-gene features, using the 'chr' phasing_region_key from feature annotation loaded via xclone.pp.load_anno (hg19_genes/hg38_genes modes with chr/start/stop/arm). The rationale is that single SNPs at scRNA-seq coverage are extremely noisy, so hierarchical aggregation strengthens the allelic-imbalance signal before fitting the beta-binomial mixture over the 5 configured allele-based CNA states (concentration 35.5, bounds 20–100). Execute the full BAF module via xclone.model.run_BAF(BAF_adata, config_file=xconfig) with BAF_denoise=True; the returned BAF_merge_Xdata should carry a bins-by-cells-by-states probability tensor smoothed horizontally with an HMM (trans_t=1e-6) along genome coordinates and vertically with the KNN cell-cell graph.

## Related tools

- **XClone** (Implements the BAF module's three-step haplotyping (population SNP-to-gene phasing, EM local phasing of 100-gene bins, dynamic programming to chromosome arms) and the beta-binomial HMM/KNN model via xclone.model.run_BAF) — https://github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit that generates the allele-specific AD/DP matrices consumed by the BAF phasing pipeline from SAM/BAM/CRAM files) — https://github.com/hxj5/xcltk

## Examples

```
BAF_merge_Xdata = xclone.model.run_BAF(BAF_adata, config_file=xconfig)
```

## Evaluation signals

- BAF_merge_Xdata returned by run_BAF carries a probability tensor over all 5 configured allele states for every bin and cell.
- Gene bins before global phasing contain 100 consecutive genes by default (phasing_len=100), and mega-gene features map to whole chromosome arms from the arm annotation.
- Config values match the paper defaults: theo_neutral_BAF=0.5, concentration=35.5 (lower=20, upper=100), start_prob=[0.2, 0.15, 0.3, 0.15, 0.2], trans_t=1e-6 with the corresponding 5x5 trans_prob.
- Downstream CNA calls reproduce expected allele-specific patterns, e.g. widespread LOH in TNBC1 and allele-specific loss in the glioma BCH896 sample.
- Output saves cleanly as BAF_merge_Xdata.h5ad and aligns with genome coordinates from the hg19/hg38 gene annotation.

## Limitations

- HoneyBADGER- and CaSpER-style independent use of individual SNPs suffers high noise due to extremely low coverage of a single variant; this aggregation trades resolution for signal.
- Numbat-style haplotype aggregation may sacrifice single-cell resolution of clonal CNAs to a pseudo-bulk level — arm-level phasing similarly averages within chromosome arms.
- Only heterozygous variants contribute; copy-neutral LOH detection depends on quality of the upstream reference-based phasing.

## Evidence

- [intro] Three-step haplotyping in the BAF module: from individual SNPs to a gene by population-based phasing, from consecutive genes to a gene bin by an Expectation-Maximization algorithm, and from gene bins to a chromosome arm by dynamic programming.: "we implemented a three-step haplotyping, from individual SNPs to a gene by population-based phasing, from consecutive genes (100 by default) to a gene bin by an Expectation-Maximization algorithm"
- [methods] BAF module run command: "BAF_merge_Xdata = xclone.model.run_BAF(BAF_adata,
                config_file = xconfig)"
- [intro] Rationale for aggregating allelic information: "given the coverage of a single variant is extremely low, their independent use of individual single nucleotide polymorphisms (SNPs) often suffers from high noise."
- [readme] xcltk generates the AD/DP matrices consumed by BAF phasing: "It takes three matrices as input: the allele-specific *AD* and *DP* matrices
(BAF signals) and the *total read depth* matrix (RDR signals)."
