---
name: hmm-knn-cna-state-smoothing
description: Use when calling per-gene, per-cell copy-number states from sparse, low-coverage
  scRNA-seq read-depth (RDR) or B-allele-frequency (BAF) matrices where raw signals
  are too noisy and need smoothing on gene-coordinate and cell-neighborhood graphs.
when_to_use_negative:
- Input is already a smoothed CNA probability tensor or combined XClone output (combine_Xdata.h5ad)
  — re-smoothing would double-count neighborhood information
- Data are bulk or pseudo-bulk (e.g., Numbat-style aggregated clones) where single-cell
  KNN smoothing over cells is not applicable
- No reference/normal cells are available to anchor the RDR normalization (ref_celltype
  undefined)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_2640
tools:
- name: XClone
  role: Implements run_RDR and run_BAF with HMM genome-coordinate smoothing and KNN
    cell-graph smoothing; configuration via XCloneConfig(module='RDR'/'BAF')
  repo: https://github.com/single-cell-genetics/XClone
- name: xcltk
  role: Generates the RDR read-depth matrix and BAF allelic AD/DP matrices from BAM/CRAM
    files that feed the smoothed HMM/KNN pipeline
  repo: https://github.com/hxj5/xcltk
- name: statsmodels
  role: Fits the negative-binomial GLM within the RDR module prior to HMM/KNN smoothing
- name: scanpy
  role: AnnData handling and KNN cell-connectivity graph construction used by the
    vertical smoothing step
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_008
  - task_007
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:9f4bc8a1c37e25a1f967f81731a426917d4a5d6b32eefcf907c6b0f533c2b012
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/XClone
  tools:
  - CaSpER
  - CopyKAT
  - InferCNV
  - Numbat
  - XClone
  - scanpy
  - statsmodels
  - xcltk
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# hmm-knn-cna-state-smoothing

## Summary

XClone's orthogonal smoothing strategy for CNA-state inference from scRNA-seq: hidden Markov model smoothing along genome coordinates (horizontal) combined with KNN cell-cell connectivity graph smoothing (vertical), applied to both the RDR and BAF modules to strengthen noisy read-depth and allelic-imbalance signals before combining them into haplotype-aware CNA calls.

## When to use

Use when calling per-gene, per-cell copy-number states from sparse, low-coverage scRNA-seq read-depth (RDR) or B-allele-frequency (BAF) matrices where raw signals are too noisy and need smoothing on gene-coordinate and cell-neighborhood graphs.

## When NOT to use

- Input is already a smoothed CNA probability tensor or combined XClone output (combine_Xdata.h5ad) — re-smoothing would double-count neighborhood information
- Data are bulk or pseudo-bulk (e.g., Numbat-style aggregated clones) where single-cell KNN smoothing over cells is not applicable
- No reference/normal cells are available to anchor the RDR normalization (ref_celltype undefined)

## Inputs

- Gene-by-cell expression read/UMI count matrix (AnnData) for the RDR module, with cell annotations (cell_anno_key, ref_celltype for normal cells)
- Allelic AD/DP count AnnData from xcltk preprocessing (BAF module), with feature annotation (chr/start/stop/arm) loaded via xclone.pp.load_anno(genome_mode='hg38_genes')
- XCloneConfig settings: trans_t=1e-6, start_prob (3-state for RDR, 5-state for BAF), run_KNN=True

## Outputs

- Genes-by-cells-by-states CNA probability tensor from the RDR module (RDR_Xdata, .h5ad)
- Bins-by-cells-by-states probability tensor from the BAF module (BAF_merge_Xdata.h5ad)
- Per-gene, per-cell smoothed CNA-state (loss/neutral/gain) probability layers for downstream run_combine

## How to apply

Run the RDR module (xclone.model.run_RDR) on the gene-by-cell expression count AnnData with XCloneConfig(module='RDR') using trans_t=1e-6, start_prob=np.array([0.3, 0.4, 0.3]), ref_celltype='N', and top_n_marker=25; counts are normalized against reference cells, filtered with library-ratio capping, and fitted with a negative-binomial GLM (statsmodels). Smooth the resulting CNA-state probabilities along the KNN cell-connectivity graph (run_KNN=True) and then with an HMM along genome coordinates using the configured transition matrix, yielding a genes-by-cells-by-states probability tensor. Apply the same HMM/KNN smoothing in the BAF module (run_BAF) with the 5-state allele-bias model, start_prob=[0.2, 0.15, 0.3, 0.15, 0.2], trans_t=1e-6, concentration=35.5, and BAF_denoise=True, operating on phased mega-gene features. The rationale, per the paper, is that smoothing on both orthogonal graphs strengthens read-depth and allelic-imbalance signals that would otherwise be lost to single-cell sparsity.

## Related tools

- **XClone** (Implements run_RDR and run_BAF with HMM genome-coordinate smoothing and KNN cell-graph smoothing; configuration via XCloneConfig(module='RDR'/'BAF')) — https://github.com/single-cell-genetics/XClone
- **xcltk** (Generates the RDR read-depth matrix and BAF allelic AD/DP matrices from BAM/CRAM files that feed the smoothed HMM/KNN pipeline) — https://github.com/hxj5/xcltk
- **statsmodels** (Fits the negative-binomial GLM within the RDR module prior to HMM/KNN smoothing)
- **scanpy** (AnnData handling and KNN cell-connectivity graph construction used by the vertical smoothing step)

## Examples

```
xconfig = xclone.XCloneConfig(module='RDR'); xconfig.trans_t = 1e-6; xconfig.start_prob = np.array([0.3, 0.4, 0.3]); RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file=xconfig)
```

## Evaluation signals

- RDR_Xdata contains a genes-by-cells-by-states probability tensor with per-gene loss/neutral/gain calls; BAF_merge_Xdata carries the probability tensor over all 5 configured states for every bin and cell
- Smoothed calls reproduce known clonal CNA events, e.g., chr5 gains in ATC2 subclone 2, chr7 and chr19 gains and chr6p loss, concordant with InferCNV and CopyKAT
- Downstream subclone clustering (XClustering) on the combined profiles reaches ARI ~0.998 vs inferCNV and ~0.961 vs CopyKAT on ATC2, and ARI 0.965 vs CopyKAT Clone_ID on TNBC1
- State probabilities sum to 1 across states per gene/cell and transition smoothing respects trans_t=1e-6 along genome coordinates
- BCH869 benchmark: smoothed XClone calls yield AUROC 0.925 for copy gain (vs 0.853 InferCNV, 0.845 Numbat) against WGS-derived ground truth

## Limitations

- Performance depends on availability of normal reference cells (ref_celltype='N') for RDR normalization; tumor purity as low as 2% (ATC2) is challenging though still resolved by smoothing
- BAF module requires heterozygous SNP coverage and population-based phasing; low per-SNP coverage is why raw allelic signals need the three-step phasing before smoothing
- XClone requires Python 3.7 or >=3.9 (3.9 recommended for stable performance) and is under frequent active development, so results may vary across versions

## Evidence

- [abstract] strengthens the signals of read depth and allelic imbalance by effective smoothing on cell neighborhood and gene coordinate graphs to detect haplotype-aware CNAs: "strengthens the signals of read depth and allelic imbalance by effective smoothing on cell neighborhood and gene coordinate graphs to detect haplotype-aware CNAs"
- [intro] horizontally with hidden Markov models (HMM) along the genome coordinates and vertically with a k-nearest-neighbors (KNN) cell-cell connectivity graph: "horizontally with hidden Markov models (HMM) along the genome coordinates and vertically with a k-nearest-neighbors (KNN) cell-cell connectivity graph"
- [other] Smooth the CNA-state probabilities along the KNN cell-connectivity graph (run_KNN=True) followed by a hidden Markov model along genome coordinates: [evidence span withheld — anchor status: not_found]
- [methods] RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file = xconfig): "RDR_Xdata = xclone.model.run_RDR(RDR_adata,
                config_file = xconfig)"
- [readme] XClone requires Python 3.7 or Python >=3.9 (Recommend 3.9 for stable performance in latest version): "XClone requires Python 3.7 or Python >=3.9 (Recommend 3.9 for stable performance in latest version)."
