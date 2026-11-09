---
name: haplotype-phasing-em-gene-binning
description: Use when building allele-specific CNA calls from scRNA-seq allelic counts
  (AD/DP matrices) and single-SNP BAF signals are too sparse/noisy, requiring aggregation
  from SNPs to genes, gene bins, and chromosome arms before HMM smoothing.
when_to_use_negative:
- The analysis goal is read-depth-based CNA calling without allelic information —
  use the RDR module, since InferCNV/CopyKAT-style RDR approaches cannot detect copy-neutral
  LOH but this skill targets B-allele imbalance.
- Only bulk (not single-cell) data or single-cell DNA-seq is available — this skill
  is built on scRNA-seq allelic UMI counts from platforms such as 10x Genomics and
  SMART-seq via xcltk preprocessing.
- Variants are unphased individual SNPs being analyzed independently — the point of
  this skill is to avoid the high-noise independent-SNP approach used by HoneyBADGER/CaSpER.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_2640
tools:
- name: XClone
  role: Implements the BAF module's three-step phasing (population phasing, EM gene
    binning, dynamic programming to chromosome arm), beta-binomial mixture fitting,
    and HMM/KNN smoothing via xclone.model.run_BAF.
  repo: https://github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit (re-designed from cellsnp-lite) that generates the allele-specific
    AD and DP matrices consumed by the BAF phasing pipeline.
  repo: https://github.com/hxj5/xcltk
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:6cf8e21ed81d9de369860939078f9c255f933e5e836b5925d71d71dc173d7410
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/XClone
  tools:
  - XClone
  - scanpy
  - xcltk
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# haplotype-phasing-em-gene-binning

## Summary

XClone's three-level haplotyping strategy for the BAF module: population-based phasing of individual SNPs to genes, EM-based aggregation of consecutive genes (100 by default) into gene bins, and dynamic-programming aggregation of gene bins to whole chromosome arms, enabling allele-imbalance CNA calling from sparse scRNA-seq data.

## When to use

Use when building allele-specific CNA calls from scRNA-seq allelic counts (AD/DP matrices) and single-SNP BAF signals are too sparse/noisy, requiring aggregation from SNPs to genes, gene bins, and chromosome arms before HMM smoothing.

## When NOT to use

- The analysis goal is read-depth-based CNA calling without allelic information — use the RDR module, since InferCNV/CopyKAT-style RDR approaches cannot detect copy-neutral LOH but this skill targets B-allele imbalance.
- Only bulk (not single-cell) data or single-cell DNA-seq is available — this skill is built on scRNA-seq allelic UMI counts from platforms such as 10x Genomics and SMART-seq via xcltk preprocessing.
- Variants are unphased individual SNPs being analyzed independently — the point of this skill is to avoid the high-noise independent-SNP approach used by HoneyBADGER/CaSpER.

## Inputs

- BAF anndata object with integer allelic 'AD' and 'DP' layers (e.g. from xclone.data.tnbc1_b() demo data or xclone.pp.xclonedata on AD.mtx/DP.mtx)
- Feature annotation with chromosome, start, stop, and arm loaded via xclone.pp.load_anno(genome_mode='hg19_genes' or 'hg38_genes')
- xclone.XCloneConfig(module='BAF') with phasing_region_key, phasing_len=100, theo_neutral_BAF=0.5, concentration=35.5, CNV_N_components=5, start/trans probabilities, and BAF_denoise=True

## Outputs

- Phased BAF anndata with genes collapsed into 100-gene bins and chromosome-arm-level phasing
- BAF_merge_Xdata anndata carrying a bins-by-cells-by-states probability tensor over all 5 configured allele-bias states, saved as BAF_merge_Xdata.h5ad

## How to apply

Load a BAF anndata with integer allelic 'AD' and 'DP' layers plus feature annotation (chr/start/stop/arm) via xclone.pp.load_anno (e.g. genome_mode 'hg19_genes' or 'hg38_genes'). Configure xclone.XCloneConfig(module='BAF') with phasing_region_key='chr', phasing_len=100, theo_neutral_BAF=0.5, concentration=35.5 (lower=20, upper=100), CNV_N_components=5, start_prob=[0.2, 0.15, 0.3, 0.15, 0.2], trans_t=1e-6 with the matching 5x5 trans_prob, and BAF_denoise=True. Then run the three-step phasing in order: SNP-to-gene population phasing, EM-based phasing that collapses consecutive genes (phasing_len=100 by default) into gene bins, and dynamic-programming phasing of gene bins up to the chromosome arm. On the phased mega-gene features, fit the beta-binomial mixture over the allele-based CNA states (3 or 5 allele bias states; 5-state mode per config) and smooth state probabilities horizontally with an HMM along genome coordinates and vertically with the KNN cell-cell connectivity graph.

## Related tools

- **XClone** (Implements the BAF module's three-step phasing (population phasing, EM gene binning, dynamic programming to chromosome arm), beta-binomial mixture fitting, and HMM/KNN smoothing via xclone.model.run_BAF.) — https://github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit (re-designed from cellsnp-lite) that generates the allele-specific AD and DP matrices consumed by the BAF phasing pipeline.) — https://github.com/hxj5/xcltk

## Examples

```
xconfig = xclone.XCloneConfig(module='BAF'); xconfig.phasing_len = 100; xconfig.CNV_N_components = 5; BAF_merge_Xdata = xclone.model.run_BAF(BAF_adata, config_file = xconfig)
```

## Evaluation signals

- The phasing must complete all three levels: SNPs→genes (population phasing), genes→100-gene bins (EM), bins→chromosome arms (dynamic programming); confirm bin counts reflect phasing_len=100.
- BAF_merge_Xdata must carry the probability tensor over all 5 configured allele-bias states (CNV_N_components=5) for every bin and every cell.
- The neutral state should center on theo_neutral_BAF=0.5, and the beta-binomial concentration must stay within the configured bounds (lower=20, upper=100).
- Smoothed state probabilities should be spatially coherent along genome coordinates (HMM with trans_t=1e-6) and across cells (KNN graph) — check that abrupt single-bin state jumps are suppressed.
- Saved BAF_merge_Xdata.h5ad should load and match the demo outputs (e.g. TNBC1/BCH869 notebooks in xclone-data) for reproducibility.

## Limitations

- Individual SNPs suffer from high noise due to extremely low coverage of a single variant in scRNA-seq, which is precisely why aggregation to gene bins and chromosome arms is required.
- Results depend on prior phasing quality; the pipeline starts from reference-based phasing and heterozygous variants only.
- XClone is being updated and improved frequently at this stage; users should keep software up to date and check the release history/tutorials.

## Evidence

- [intro] Three-step haplotyping in the BAF module: "we implemented a three-step haplotyping, from individual SNPs to a gene by population-based phasing, from consecutive genes (100 by default) to a gene bin by an Expectation-Maximization algorithm"
- [intro] HMM and KNN smoothing: "horizontally with hidden Markov models (HMM) along the genome coordinates and vertically with a"
- [methods] run_BAF invocation: "BAF_merge_Xdata = xclone.model.run_BAF(BAF_adata,
                config_file = xconfig)"
- [readme] xcltk generates AD/DP matrices: "It takes three matrices as input: the allele-specific *AD* and *DP* matrices
(BAF signals) and the *total read depth* matrix (RDR signals)."
- [intro] Noise motivation for aggregation: "given the coverage of a single variant is extremely low, their independent use of individual single nucleotide polymorphisms (SNPs) often suffers from high noise."
