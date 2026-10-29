---
name: rdr-baf-cna-state-combination
description: Use when both an RDR_Xdata and a BAF_merge_Xdata from XClone runs on
  the same cells/genes exist and per-cell haplotype-aware copy-number states (including
  copy-neutral LOH) are needed.
when_to_use_negative:
- 'Only one signal type is available (e.g., no BAF/allele data): combine cannot run
  and RDR alone will miss copy-neutral LOH.'
- The goal is bulk or pseudo-bulk clonal CNA calling (e.g., Numbat-style); XClone
  combine is designed for per-cell resolution from scRNA-seq counts.
- Inputs are raw SAM/BAM/CRAM files rather than preprocessed AD/DP/RDR matrices —
  run the xcltk preprocessing pipeline and RDR/BAF modules first.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3518
tools:
- name: XClone
  role: Main package; run_combine integrates RDR and BAF module outputs into final
    CNA states
  repo: github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit that generates the AD, DP, and total read-depth matrices
    consumed by the RDR and BAF modules prior to combination
  repo: github.com/hxj5/xcltk
- name: statsmodels
  role: Fits the negative-binomial GLM in the RDR module whose state probabilities
    feed into the combination step
- name: scanpy
  role: AnnData/h5ad ecosystem used to load, configure, and save the RDR, BAF, and
    combined Xdata objects
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:9383cbb52b69037acdb901cc9d6b30afed90cf44ef48c4cd652d9dd31e4cf4f1
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
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# rdr-baf-cna-state-combination

## Summary

Combines XClone's RDR-module read-depth CNA probabilities with BAF-module allele-ratio states via run_combine to emit final allele-specific per-gene-per-cell CNA calls, optionally merging LOH and flagging whole-genome duplication.

## When to use

Use when both an RDR_Xdata and a BAF_merge_Xdata from XClone runs on the same cells/genes exist and per-cell haplotype-aware copy-number states (including copy-neutral LOH) are needed.

## When NOT to use

- Only one signal type is available (e.g., no BAF/allele data): combine cannot run and RDR alone will miss copy-neutral LOH.
- The goal is bulk or pseudo-bulk clonal CNA calling (e.g., Numbat-style); XClone combine is designed for per-cell resolution from scRNA-seq counts.
- Inputs are raw SAM/BAM/CRAM files rather than preprocessed AD/DP/RDR matrices — run the xcltk preprocessing pipeline and RDR/BAF modules first.

## Inputs

- RDR_Xdata: genes-by-cells AnnData with CNA-state probabilities from run_RDR (negative-binomial GLM + KNN + HMM smoothing)
- BAF_merge_Xdata: genes-by-cells AnnData with phased BAF allele states mapped back to gene scale from run_BAF
- xconfig: XCloneConfig objects for RDR, BAF, and combine modules (e.g., ref_celltype='N', top_n_marker=25, trans_t=1e-6, merge_loh=True)

## Outputs

- combine_Xdata.h5ad: AnnData with final four-state per-gene-per-cell CNA calls and smoothed probability layers
- Whole-genome duplication (WGD) warning when duplication signatures are present

## How to apply

Run the RDR module (negative-binomial GLM with statsmodels, KNN smoothing, and HMM along genome coordinates) and the BAF module (three-step haplotyping, CNV_N_components=5, BAF_denoise=True, start_prob=[0.2,0.15,0.3,0.15,0.2]) separately, each producing per-gene-per-cell state probabilities. Then call xclone.model.run_combine(RDR_Xdata, BAF_merge_Xdata, config_file=xconfig) with merge_loh=True and copygain_correct=False, so RDR copy-gain/loss evidence and BAF allele-imbalance (LOH) evidence are reconciled into the final combined states. Enable merge_loh to fold BAF loss-of-heterozygosity calls into the combined states (important since RDR alone cannot detect copy-neutral LOH), and watch for the emitted WGD warning when duplication signatures are present. Save the result as combine_Xdata.h5ad and validate that it contains per-gene-per-cell CNA state calls plus smoothed probability layers.

## Related tools

- **XClone** (Main package; run_combine integrates RDR and BAF module outputs into final CNA states) — github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit that generates the AD, DP, and total read-depth matrices consumed by the RDR and BAF modules prior to combination) — github.com/hxj5/xcltk
- **statsmodels** (Fits the negative-binomial GLM in the RDR module whose state probabilities feed into the combination step)
- **scanpy** (AnnData/h5ad ecosystem used to load, configure, and save the RDR, BAF, and combined Xdata objects)

## Examples

```
combine_Xdata = xclone.model.run_combine(RDR_Xdata, BAF_merge_Xdata, config_file=xconfig)
```

## Evaluation signals

- combine_Xdata.h5ad exists and loads as an AnnData with per-gene-per-cell CNA state calls spanning the four combined states.
- The combined object contains smoothed probability layers alongside the hard state calls.
- merge_loh=True yields copy-neutral LOH calls in regions where RDR is neutral but BAF shows allelic imbalance (e.g., widespread LOH in TNBC1).
- The WGD warning is emitted when genome-wide duplication signatures are present (e.g., astrocytoma sample).
- Combined states agree with per-module intermediate results: gain/loss from RDR probabilities and LOH from BAF states are reconciled without unexplained state conflicts.

## Limitations

- RDR alone cannot detect copy-neutral LOH because it ignores allelic information, so combination quality depends on BAF phasing success.
- XClone is being updated frequently at this stage; behavior and defaults (e.g., copygain_correct, CNV_N_components) may change between versions, so keep the package up to date and pin the environment.
- Coverage of a single variant is extremely low in scRNA-seq, so BAF signals are noisy and rely on the module's denoising and phasing before combination.

## Evidence

- [intro] Two modules combined into final states: "this method accounts for two modules: the B-allele frequency (BAF) of heterozygous variants and the sequencing read depth ratio (RDR) of individual genes"
- [intro] KNN + HMM smoothing strategy: "we employed two orthogonal strategies to smooth the CNA-state assignments on both BAF and RDR modules: horizontally with hidden Markov models (HMM) along the genome coordinates"
- [methods] Combine call signature: "RDR_Xdata = xclone.model.run_RDR(RDR_adata,
                config_file = xconfig)"
- [intro] InferCNV/CopyKAT LOH blind spot motivating combination: "However, both InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH)."
- [abstract] WGD detection capability: "and potentially indicate whole genome duplication, therefore enabling the discovery of corresponding subclones and the dissection of their phenotypic impacts"
- [readme] XClone repo and install path: "pip install git+https://github.com/single-cell-genetics/XClone"
