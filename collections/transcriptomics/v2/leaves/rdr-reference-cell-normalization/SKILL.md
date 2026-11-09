---
name: rdr-reference-cell-normalization
description: Use when running XClone's RDR module on a gene-by-cell count matrix that
  contains a known normal/reference cell population, and copy-loss/neutral/gain states
  must be called relative to that baseline.
when_to_use_negative:
- The dataset contains no identifiable normal/reference cell type, so ref_celltype
  cannot be set and baseline normalization is undefined.
- Input is already allele-specific BAF signal (AD/DP matrices) — that is handled by
  the BAF module, not the RDR reference normalization.
- The goal is pseudo-bulk or single-cell DNA-seq CNA calling (e.g. ACT, CHISEL), which
  does not rely on scRNA-seq read-depth ratios.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: XClone
  role: 'Implements the RDR module: reference-cell normalization, gene filtering with
    library-ratio capping, negative-binomial GLM, KNN and HMM smoothing'
  repo: https://github.com/single-cell-genetics/XClone
- name: statsmodels
  role: Fits the negative-binomial GLM on normalized read-depth counts
- name: xcltk
  role: Preprocesses BAM/CRAM into the total read-depth matrix consumed by the RDR
    module
  repo: https://github.com/hxj5/xcltk
- name: xclone-data
  role: Provides processed demo inputs (e.g. TNBC1_scRNA) and example notebooks for
    reproducible RDR runs
  repo: https://github.com/Rongtingting/xclone-data
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:4e64785ac34f6620555ef04dbe1d257f9086e511d6785df8e11c28e2b2f1d562
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/XClone
  tools:
  - XClone
  - scanpy
  - statsmodels
  - xclone-data
  - xcltk
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# RDR reference-cell normalization

## Summary

Normalizes the gene-by-cell scRNA-seq read-depth matrix against a user-specified reference cell type (e.g. 'N' non-tumour cells) before XClone's negative-binomial RDR modeling, so copy-number states are expressed as ratios relative to baseline ploidy.

## When to use

Use when running XClone's RDR module on a gene-by-cell count matrix that contains a known normal/reference cell population, and copy-loss/neutral/gain states must be called relative to that baseline.

## When NOT to use

- The dataset contains no identifiable normal/reference cell type, so ref_celltype cannot be set and baseline normalization is undefined.
- Input is already allele-specific BAF signal (AD/DP matrices) — that is handled by the BAF module, not the RDR reference normalization.
- The goal is pseudo-bulk or single-cell DNA-seq CNA calling (e.g. ACT, CHISEL), which does not rely on scRNA-seq read-depth ratios.

## Inputs

- gene-by-cell total read-depth count matrix (AnnData, e.g. TNBC1_rdr)
- cell-type annotation identifying reference cells (e.g. 'N')
- gene annotation for genome mode ('hg38_genes' via xclone.pp.load_anno)

## Outputs

- reference-normalized RDR AnnData used for negative-binomial GLM fitting
- genes-by-cells-by-states CNA probability tensor (loss/neutral/gain) after KNN and HMM smoothing (RDR_Xdata)

## How to apply

Load the RDR matrix (e.g. xclone.data.tnbc1_rdr()) with gene annotation from xclone.pp.load_anno(genome_mode='hg38_genes'), then configure the module via xclone.XCloneConfig(module='RDR') setting ref_celltype='N', top_n_marker=25, trans_t=1e-6 and start_prob=np.array([0.3, 0.4, 0.3]) before xclone.model.run_RDR(). Within the run, counts are first normalized to the reference cells, then gene filtering applies library-ratio capping (with cell filter limits) so that extreme library-size cells do not distort the baseline. The normalized data feed a negative-binomial GLM (statsmodels) with heuristic estimation of library size, dispersions, and copy loss/neutral/gain ratios; the resulting state probabilities are then smoothed on the KNN cell graph and along genome coordinates with an HMM. The rationale is that read-depth ratios relative to diploid reference cells convert raw counts into ploidy-proportional signals robust to per-cell sequencing depth.

## Related tools

- **XClone** (Implements the RDR module: reference-cell normalization, gene filtering with library-ratio capping, negative-binomial GLM, KNN and HMM smoothing) — https://github.com/single-cell-genetics/XClone
- **statsmodels** (Fits the negative-binomial GLM on normalized read-depth counts)
- **xcltk** (Preprocesses BAM/CRAM into the total read-depth matrix consumed by the RDR module) — https://github.com/hxj5/xcltk
- **xclone-data** (Provides processed demo inputs (e.g. TNBC1_scRNA) and example notebooks for reproducible RDR runs) — https://github.com/Rongtingting/xclone-data

## Examples

```
hg38_genes = xclone.pp.load_anno(genome_mode='hg38_genes'); xconfig = xclone.XCloneConfig(module='RDR'); xconfig.ref_celltype = 'N'; RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file=xconfig)
```

## Evaluation signals

- Config check: xconfig for module='RDR' has ref_celltype set (e.g. 'N') and start_prob=np.array([0.3, 0.4, 0.3]) with trans_t=1e-6 before run_RDR.
- RDR_Xdata exists after run and contains a genes-by-cells-by-states probability tensor with three states (loss/neutral/gain).
- Gene filtering applied: library-ratio capping removed or capped extreme cells without crashing the GLM fit.
- HMM smoothing along genome coordinates respects the configured transition matrix; adjacent-gene state probabilities are spatially coherent (KNN/HMM smoothing engaged via run_KNN=True).
- Downstream combine module accepts the RDR_Xdata and merges with BAF output (merge_loh=True) to emit four-state CNA calls in combine_Xdata.h5ad.

## Limitations

- Requires a pre-annotated reference cell population; performance depends on the purity and correct labeling of reference cells (e.g. 'N' in TNBC1).
- The RDR module alone cannot detect copy-neutral LOH, since it ignores allelic information — BAF module combination is needed.
- Sparse, low-coverage 10x data are noisy; smoothing helps but individual-gene calls remain uncertain, and update notes flag ongoing changes to 'libratio capping and add cell filter limits' and the 'RDR guide ratio correct strategy', so behavior may vary across versions.

## Evidence

- [methods] RDR module normalization to reference cells: [evidence span withheld — anchor status: not_found]
- [intro] HMM along genome coordinates: "horizontally with hidden Markov models (HMM) along the genome coordinates"
- [intro] KNN vertical smoothing: "we employed two orthogonal strategies to smooth the CNA-state assignments on both BAF and RDR modules: horizontally with hidden Markov models (HMM) along the genome coordinates and vertically with a"
- [methods] RDR config ref_celltype: "xconfig.BAF_denoise = True"
- [intro] Read depth ratio module: "this method accounts for two modules: the B-allele frequency (BAF) of heterozygous variants and the sequencing read depth ratio (RDR) of individual genes"
- [readme] README install and RDR input: [evidence span withheld — anchor status: external_doc]
