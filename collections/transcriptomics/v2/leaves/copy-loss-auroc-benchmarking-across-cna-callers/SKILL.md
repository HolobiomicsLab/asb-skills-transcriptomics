---
name: copy-loss-auroc-benchmarking-across-cna-callers
description: Use when a simulated scRNA-seq dataset has known ground-truth CNA labels
  (e.g. allele-specific chr3 loss injected across subclones) and you must quantify
  which caller (XClone, InferCNV, Numbat) recovers the loss via AUROC.
when_to_use_negative:
- The input is real tumour data without ground-truth CNA labels — AUROC benchmarking
  requires simulator-provided truth labels
- Only one CNA caller is to be run; the skill is specifically a cross-caller comparison
  on identical inputs
- The CNA event of interest is copy-neutral LOH only — InferCNV and CopyKAT cannot
  detect copy-neutral LOH since they ignore allelic information, so AUROC comparison
  against them is not meaningful
tools:
- name: XClone
  role: Primary allele-specific CNA caller being benchmarked; runs RDR, BAF, and Combine
    modules on xcltk-generated matrices
  repo: https://github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit converting the simulated SAM/BAM into RDR and allele-specific
    AD/DP count matrices for all callers
  repo: https://github.com/hxj5/xcltk
- name: scCNASimulator
  role: Generates the simulated BAM with known ground-truth CNA labels by injecting
    allele-specific chr3 copy loss into subclones
- name: InferCNV
  role: Read-depth-only comparator caller run on the identical simulated RDR inputs
- name: Numbat
  role: Pseudo-bulk haplotype-based comparator caller run on the identical simulated
    inputs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:4df1029c0dc0cf56a83d5c89f478b9688599f15e2fe9aca30d47ee9a9f3bcf9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/XClone
  tools:
  - InferCNV
  - Numbat
  - XClone
  - scCNASimulator
  - xcltk
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# copy-loss-auroc-benchmarking-across-cna-callers

## Summary

Benchmarking skill for comparing allele-specific copy-loss detection accuracy of scRNA-seq CNA callers (XClone, InferCNV, Numbat) on simulated ground-truth data, scored per cell/gene via AUROC against simulator CNA labels.

## When to use

Use when a simulated scRNA-seq dataset has known ground-truth CNA labels (e.g. allele-specific chr3 loss injected across subclones) and you must quantify which caller (XClone, InferCNV, Numbat) recovers the loss best via AUROC.

## When NOT to use

- The input is real tumour data without ground-truth CNA labels — AUROC benchmarking requires simulator-provided truth labels
- Only one CNA caller is to be run; the skill is specifically a cross-caller comparison on identical inputs
- The CNA event of interest is copy-neutral LOH only — InferCNV and CopyKAT cannot detect copy-neutral LOH since they ignore allelic information, so AUROC comparison against them is not meaningful

## Inputs

- Simulated BAM from scCNASimulator (GX109-T1c seed, allele-specific chr3 copy loss in two subclones)
- Ground-truth CNA labels from the simulator (per cell and per gene)
- xcltk output matrices: RDR total read-depth matrix and BAF allele-specific AD/DP matrices
- Barcode (barcodes.tsv) and feature annotation files for hg38_genes genome mode
- Cell/subclone annotations

## Outputs

- auroc_benchmark.csv with per-caller copy-loss AUROC (XClone, InferCNV, Numbat)
- XClone CNA state calls from RDR, BAF, and Combine modules
- Per-cell and per-gene copy-loss recovery scores against simulator ground truth

## How to apply

Seed scCNASimulator with real tumour scRNA-seq (e.g. the GX109-T1c gastric cancer 10x 5' dataset) and inject the CNA event of interest — here an allele-specific copy loss on chromosome 3 across two subclones — so ground-truth per-cell/per-gene labels are known. Run xcltk preprocessing on the simulated BAM to produce the RDR read-depth matrix and the allele-specific AD/DP matrices, then load them into AnnData with xclone.pp.xclonedata (genome_mode='hg38_genes' plus barcodes/feature annotation) and attach cell annotations via xclone.pp.extra_anno. Execute all three XClone modules (run_RDR, run_BAF, run_combine) with documented settings such as trans_t = 1e-6 and xconfig.BAF_denoise = True, and run InferCNV and Numbat on the identical simulated RDR and allelic inputs so the comparison is input-controlled. Score copy-loss recovery per cell and per gene against the simulator's ground-truth CNA labels to compute AUROCs, e.g. XClone 0.9964 vs InferCNV 0.9568 vs Numbat 0.2971 (Numbat entirely missed the chr3 loss), and save to auroc_benchmark.csv.

## Related tools

- **XClone** (Primary allele-specific CNA caller being benchmarked; runs RDR, BAF, and Combine modules on xcltk-generated matrices) — https://github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit converting the simulated SAM/BAM into RDR and allele-specific AD/DP count matrices for all callers) — https://github.com/hxj5/xcltk
- **scCNASimulator** (Generates the simulated BAM with known ground-truth CNA labels by injecting allele-specific chr3 copy loss into subclones)
- **InferCNV** (Read-depth-only comparator caller run on the identical simulated RDR inputs)
- **Numbat** (Pseudo-bulk haplotype-based comparator caller run on the identical simulated inputs)

## Examples

```
conda create -n xclone python=3.9; conda activate xclone; pip install xclone; pip install -U xcltk; python -c "import xclone; xconfig = xclone.XCloneConfig(module='RDR'); RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file=xconfig)"
```

## Evaluation signals

- AUROC for XClone's chr3 copy-loss recovery should approach the reported 0.9964 on the GX109-T1c chr3-loss simulation
- All three callers must consume identical simulated RDR and allelic inputs — any difference in AUROC must come from the caller, not the input
- Numbat's expected failure mode on this benchmark is AUROC near chance (0.2971) with chr3 copy loss entirely missed, matching the published finding
- auroc_benchmark.csv exists and contains one AUROC value per caller scored against the simulator's ground-truth labels
- XClone run reproduces only with documented config settings applied (e.g. trans_t = 1e-6, BAF_denoise = True, module='RDR'/'BAF'/'Combine')

## Limitations

- Numbat may sacrifice resolution of clonal CNAs from a single-cell level to a pseudo-bulk level, and its performance and robustness remain to be further assessed
- The benchmark covers one simulated CNA event (allele-specific chr3 copy loss) and may not generalise to other CNA types or datasets
- XClone requires a specific environment (Python 3.9 recommended, per xclone0.3.6Python3.9_env.yml) and is being updated frequently at this stage

## Evidence

- [intro] Its high accuracy will be demonstrated through comprehensive benchmarking with four cancer data sets and one simulated data set: "Its high accuracy will be demonstrated through comprehensive benchmarking with four cancer data sets and one simulated data set"
- [methods] RDR module config: "RDR_Xdata = xclone.model.run_RDR(RDR_adata,
                config_file = xconfig)"
- [methods] BAF_denoise setting: "xconfig.BAF_denoise = True"
- [intro] Numbat pseudo-bulk limitation: "However, this method may sacrifice the resolution of clonal CNAs from a single-cell level to a pseudo-bulk level and its performance and robustness remain to be further assessed"
- [readme] xcltk three input matrices: "It takes three matrices as input: the allele-specific *AD* and *DP* matrices
(BAF signals) and the *total read depth* matrix (RDR signals)."
