---
name: subclonal-allele-specific-cna-groundtruth-evaluation
description: Use when a simulated scRNA-seq dataset with known ground-truth subclonal
  CNA labels (e.g., scCNASimulator chr3 loss on GX109-T1c) is available and caller
  accuracy must be quantified by AUROC.
when_to_use_negative:
- The dataset has no ground-truth CNA labels (real tumour samples without simulator-injected
  events), so AUROC against truth cannot be computed.
- The analysis goal is de novo CNA discovery or clonal reconstruction rather than
  caller benchmarking against known events.
- Input is bulk or spatial transcriptomics rather than single-cell RNA-seq count/BAM
  data supported by the callers.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2640
tools:
- name: XClone
  role: Primary caller under evaluation; RDR/BAF/Combine modules detect allele-specific
    copy loss from simulated inputs
  repo: https://github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit producing RDR, AD, and DP matrices from the simulated
    BAM as XClone inputs
  repo: https://github.com/hxj5/xcltk
- name: InferCNV
  role: RDR-only comparator caller scored against the same ground-truth labels
- name: Numbat
  role: Pseudo-bulk haplotype-phased comparator caller scored against the same ground-truth
    labels
- name: scCNASimulator
  role: Seeds the GX109-T1c dataset and injects allele-specific chr3 copy loss in
    two subclones to create ground-truth simulation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:e4453a9b013404a19989f7c808681b33ccd3567df5507728a4ecbd8a8e535914
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# subclonal-allele-specific-cna-groundtruth-evaluation

## Summary

Benchmarkes allele-specific CNA callers (XClone, inferCNV, Numbat) against simulator-provided ground-truth CNA labels, scoring per-cell/per-gene copy-loss recovery by AUROC (e.g., XClone 0.9964 vs inferCNV 0.9568 vs Numbat 0.2971 on GX109-T1c chr3 loss).

## When to use

Use when a simulated scRNA-seq dataset with known ground-truth subclonal CNA labels (e.g., scCNASimulator chr3 loss on GX109-T1c) is available and caller accuracy must be quantified by AUROC.

## When NOT to use

- The dataset has no ground-truth CNA labels (real tumour samples without simulator-injected events), so AUROC against truth cannot be computed.
- The analysis goal is de novo CNA discovery or clonal reconstruction rather than caller benchmarking against known events.
- Input is bulk or spatial transcriptomics rather than single-cell RNA-seq count/BAM data supported by the callers.

## Inputs

- Simulated scRNA-seq BAM from scCNASimulator with ground-truth subclonal CNA labels (e.g., allele-specific chr3 copy loss in two subclones seeded from GX109-T1c)
- xcltk-generated RDR read-depth count matrix and allele-specific AD/DP count matrices
- barcodes.tsv and feature annotation files (hg38_genes genome mode) plus cell annotations via xclone.pp.extra_anno

## Outputs

- Per-caller AUROC values for copy-loss detection (XClone, inferCNV, Numbat)
- auroc_benchmark.csv with per-cell and per-gene copy-loss recovery scores against ground truth

## How to apply

Run each caller on identical simulated inputs: XClone via xcltk preprocessing (RDR matrix plus allele-specific AD/DP matrices) loaded with xclone.pp.xclonedata (genome_mode='hg38_genes') and the run_RDR/run_BAF/run_combine modules with trans_t=1e-6 and BAF_denoise=True; inferCNV and Numbat use the same RDR and allelic inputs. Score copy-loss recovery per cell and per gene against the simulator's ground-truth CNA labels to compute AUROCs for each caller. Compare AUROCs to determine which method detects the injected allele-specific copy loss; a low AUROC (e.g., Numbat's 0.2971) indicates the loss was missed. Save the comparison table (auroc_benchmark.csv) for reproducibility.

## Related tools

- **XClone** (Primary caller under evaluation; RDR/BAF/Combine modules detect allele-specific copy loss from simulated inputs) — https://github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit producing RDR, AD, and DP matrices from the simulated BAM as XClone inputs) — https://github.com/hxj5/xcltk
- **InferCNV** (RDR-only comparator caller scored against the same ground-truth labels)
- **Numbat** (Pseudo-bulk haplotype-phased comparator caller scored against the same ground-truth labels)
- **scCNASimulator** (Seeds the GX109-T1c dataset and injects allele-specific chr3 copy loss in two subclones to create ground-truth simulation)

## Examples

```
RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file = xconfig)
```

## Evaluation signals

- XClone AUROC for chr3 copy-loss detection should reach ~0.9964 on the GX109-T1c two-subclone simulation, versus ~0.9568 for inferCNV and ~0.2971 for Numbat.
- Per-cell and per-gene recovery scores must align with the simulator's ground-truth CNA label matrix (cells/genes labeled as chr3 loss should score high).
- All three callers must be run on the identical simulated RDR and AD/DP inputs for a fair benchmark.
- Results table (auroc_benchmark.csv) must contain one AUROC per caller and be reproducible from the seeded simulation.
- A near-chance or below-chance AUROC (e.g., <0.5, as with Numbat here) signals the caller entirely missed the injected copy loss and should be reported as a detection failure.

## Limitations

- Numbat may sacrifice the resolution of clonal CNAs from a single-cell level to a pseudo-bulk level and can entirely miss the chr3 copy loss (AUROC 0.2971) in this setting.
- InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH), limiting them to RDR-only events.
- The benchmark reflects one simulated scenario (allele-specific chr3 loss in two subclones on GX109-T1c); performance may not generalize to other event types or datasets.

## Evidence

- [results] XClone showed superior copy-loss detection (AUROC 0.9964) versus inferCNV (0.9568) and Numbat (0.2971): [evidence span withheld — anchor status: not_found]
- [methods] Load the matrices into AnnData with xclone.pp.xclonedata: [evidence span withheld — anchor status: not_found]
- [methods] Score copy-loss recovery per cell and per gene against the simulator's ground-truth CNA labels: [evidence span withheld — anchor status: not_found]
- [readme] xcltk implements a preprocessing pipeline to generate the three matrices from SAM/BAM/CRAM files: [evidence span withheld — anchor status: external_doc]
- [intro] this method accounts for two modules: the B-allele frequency (BAF) of heterozygous variants and the sequencing read depth ratio (RDR): [evidence span withheld — anchor status: not_found]
