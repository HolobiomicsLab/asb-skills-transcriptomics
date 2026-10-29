---
name: baf-gene-scale-mapping
description: Use when running XClone's BAF module on phased scRNA-seq allele counts
  (AD/DP matrices) and the phased mega-gene-level allele states must be projected
  to per-gene resolution before run_combine merges them with RDR CNA probabilities.
when_to_use_negative:
- Input lacks heterozygous SNP allele counts (AD/DP matrices) — BAF mapping and downstream
  LOH detection require allelic information, which read-depth-only RDR cannot supply
- You only need copy gain/loss from read depth and do not intend to combine with the
  RDR module or detect copy-neutral LOH
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0199
tools:
- name: XClone
  role: Provides the BAF model (run_BAF) that fits five allele states on phased mega-genes
    and maps them back to gene scale for combination with RDR
  repo: github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit that generates the allele-specific AD/DP matrices feeding
    the BAF module
  repo: https://github.com/hxj5/xcltk
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:0ad11efdabd889a440e45b008ad3fcf70868a81763659f11b62cf613002d8f8c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - XClone
  - scanpy
  - statsmodels
  - xcltk
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# baf-gene-scale-mapping

## Summary

Maps XClone BAF module allele-state results from phased mega-gene (chromosome-arm/gene-bin) scale back to individual gene scale so that BAF-derived LOH/copy states can be combined with the RDR module's genes-by-cells-by-states tensor into the final per-gene-per-cell CNA calls.

## When to use

Use when running XClone's BAF module on phased scRNA-seq allele counts (AD/DP matrices) and the phased mega-gene-level allele states must be projected to per-gene resolution before run_combine merges them with RDR CNA probabilities.

## When NOT to use

- Input lacks heterozygous SNP allele counts (AD/DP matrices) — BAF mapping and downstream LOH detection require allelic information, which read-depth-only RDR cannot supply
- You only need copy gain/loss from read depth and do not intend to combine with the RDR module or detect copy-neutral LOH

## Inputs

- BAF_adata AnnData with allele-specific AD and DP count matrices from xcltk BAF preprocessing
- Phased mega-gene blocks keyed by chromosome (phasing_region_key='chr') from the BAF haplotyping steps
- Gene annotation (e.g. xclone.pp.load_anno(genome_mode='hg38_genes')) for aligning mega-gene bins to genes
- XCloneConfig with module='BAF', CNV_N_components=5, BAF_denoise=True, start_prob=np.array([0.2, 0.15, 0.3, 0.15, 0.2])

## Outputs

- BAF_merge_Xdata: AnnData at gene scale with per-gene-per-cell five-state BAF allele-state probabilities
- Gene-scale BAF states ready for run_combine with RDR_Xdata to produce four-state CNA calls (combine_Xdata.h5ad)

## How to apply

After the BAF module aggregates heterozygous SNPs through the three-step haplotyping (SNPs to gene by population-based phasing, 100 consecutive genes to a gene bin by EM, gene bins to a chromosome arm by dynamic programming, keyed by phasing_region_key='chr'), run xclone.model.run_BAF with a config where module='BAF', CNV_N_components=5, BAF_denoise=True, start_prob=np.array([0.2, 0.15, 0.3, 0.15, 0.2]) and a transition probability matrix built from trans_t. The BAF run fits five allele states over the phased mega-genes and then maps these states back to the gene scale, producing a BAF_merge_Xdata AnnData with per-gene-per-cell BAF-state probabilities aligned to the gene annotation (e.g. hg38_genes loaded via xclone.pp.load_anno). The BAF_denoise=True flag suppresses noisy allele-state calls in sparse cells before mapping. The gene-scale BAF output is then passed with the RDR output to xclone.model.run_combine (merge_loh=True, copygain_correct=False) to emit the final four combined CNA states.

## Related tools

- **XClone** (Provides the BAF model (run_BAF) that fits five allele states on phased mega-genes and maps them back to gene scale for combination with RDR) — github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit that generates the allele-specific AD/DP matrices feeding the BAF module) — https://github.com/hxj5/xcltk

## Examples

```
BAF_merge_Xdata = xclone.model.run_BAF(BAF_adata, config_file=xconfig)
```

## Evaluation signals

- BAF_merge_Xdata exists and its gene axis matches the gene annotation (e.g. hg38_genes) used by the RDR module, confirming mega-gene-to-gene mapping succeeded
- Each gene/cell pair carries probabilities over the five configured BAF states (start_prob summing to 1: 0.2, 0.15, 0.3, 0.15, 0.2)
- Combined output combine_Xdata.h5ad contains per-gene-per-cell CNA calls over the four combined states, with LOH states from BAF merged (merge_loh=True)
- Gene-scale BAF states are consistent with chromosome-arm-level phased calls (no gene assigned to a state contradicting its mega-gene block)
- Denoised output shows fewer isolated per-cell allele-state flips than the raw fit, consistent with BAF_denoise=True

## Limitations

- HoneyBADGER- and CaSpER-style use of individual SNPs suffers high noise from extremely low per-variant coverage; the BAF module mitigates but depends on adequate heterozygous SNP coverage and phasing quality
- Widespread LOH datasets (e.g. TNBC1) reduce available heterozygous variants, weakening BAF signal in affected regions
- The BAF module relies on population-based phasing of individual SNPs; phasing errors propagate through the gene-bin and chromosome-arm aggregation before gene-scale mapping

## Evidence

- [intro] Three-step haplotyping aggregates phased SNPs: "we implemented a three-step haplotyping, from individual SNPs to a gene by population-based phasing, from consecutive genes (100 by default) to a gene bin by an Expectation-Maximization algorithm"
- [intro] Modules are combined to generate final CNA states: "this method accounts for two modules: the B-allele frequency (BAF) of heterozygous variants and the sequencing read depth ratio (RDR) of individual genes"
- [intro] Low per-SNP coverage motivates aggregation before gene-scale mapping: "given the coverage of a single variant is extremely low, their independent use of individual single nucleotide polymorphisms (SNPs) often suffers from high noise."
- [readme] README confirms allele inputs and combined-signal design: [evidence span withheld — anchor status: external_doc]
