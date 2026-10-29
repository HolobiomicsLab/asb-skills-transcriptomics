---
name: bam-level-allele-specific-read-injection
description: 'Use when a CNA-caller benchmark needs controlled ground truth: inject
  a known allele-specific chr3 copy loss into real 10x 5'' scRNA-seq data and measure
  AUROC-based detection recovery.'
when_to_use_negative:
- Input is real tumour scRNA-seq where natural CNAs are the object of study — this
  skill fabricates synthetic events with known labels for benchmarking, not CNA discovery.
- 'The data lack allelic (SNP) information: inferCNV and CopyKAT do not consider allelic
  information and cannot detect copy-neutral LOH, so an allele-specific injection
  design will not transfer.'
- The goal is running CNA detection on a single dataset rather than comparing caller
  accuracy against simulator ground truth.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: scCNASimulator
  role: Seeds the GX109-T1c 10x 5' scRNA-seq data with an allele-specific chr3 copy
    loss in two subclones and regenerates the simulated BAM with ground-truth labels
- name: xcltk
  role: Preprocesses the simulated SAM/BAM/CRAM into the three input matrices (total
    read depth RDR; allele-specific AD/DP for BAF)
  repo: https://github.com/hxj5/xcltk
- name: XClone
  role: 'Primary caller under test: run_RDR, run_BAF, and run_combine modules with
    trans_t=1e-6 and BAF_denoise=True produce the copy-loss calls scored by AUROC'
  repo: https://github.com/single-cell-genetics/XClone
- name: InferCNV
  role: Comparator copy-loss caller run on the identical simulated RDR inputs
- name: Numbat
  role: Comparator allele-aware caller run on the identical simulated inputs; benchmarked
    for whether it recovers the chr3 loss
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:9e707cf613f46998d293285da95e6c5e57288b58940d4ef71d849a6ba358c14c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hxj5/xcltk
  tools:
  - InferCNV
  - Numbat
  - XClone
  - scCNASimulator
  - xcltk
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# bam-level-allele-specific-read-injection

## Summary

Benchmark-enabling skill: seed a real 10x 5' scRNA-seq dataset (GX109-T1c gastric cancer) with a synthetic allele-specific copy loss on chromosome 3 across two subclones using scCNASimulator, regenerate the simulated BAM, and score XClone, inferCNV, and Numbat copy-loss recovery (AUROC) against the simulator's ground-truth CNA labels.

## When to use

Use when a CNA-caller benchmark needs controlled ground truth: inject a known allele-specific chr3 copy loss into real 10x 5' scRNA-seq data and measure AUROC-based detection recovery.

## When NOT to use

- Input is real tumour scRNA-seq where natural CNAs are the object of study — this skill fabricates synthetic events with known labels for benchmarking, not CNA discovery.
- The data lack allelic (SNP) information: inferCNV and CopyKAT do not consider allelic information and cannot detect copy-neutral LOH, so an allele-specific injection design will not transfer.
- The goal is running CNA detection on a single dataset rather than comparing caller accuracy against simulator ground truth.

## Inputs

- GX109-T1c gastric cancer 10x Genomics 5' scRNA-seq dataset (BAM)
- scCNASimulator CNA specification (allele-specific chr3 copy loss, two subclones)
- hg38 genome annotations (hg38_genes, hg38_blocks), barcodes.tsv, and feature annotation files
- Cell annotation table for xclone.pp.extra_anno

## Outputs

- Simulated BAM with injected allele-specific chr3 copy loss and ground-truth CNA labels
- xcltk RDR total read-depth count matrix
- xcltk allele-specific AD and DP matrices (BAF signals)
- AnnData object assembled with xclone.pp.xclonedata
- auroc_benchmark.csv with per-caller (XClone, inferCNV, Numbat) copy-loss AUROCs

## How to apply

Create a Python 3.9 conda environment (XClone's recommended stable version) and install xclone via pip, xcltk via `pip install -U xcltk`, plus the inferCNV and Numbat comparators. Seed scCNASimulator with the GX109-T1c gastric cancer 10x 5' scRNA-seq dataset and inject an allele-specific copy loss on chromosome 3 across two subclones to regenerate the simulated BAM carrying ground-truth CNA labels. Run xcltk's preprocessing pipeline on the simulated BAM to produce three matrices — the total read depth matrix (RDR signals) and the allele-specific AD and DP matrices (BAF signals) — then load them into AnnData via xclone.pp.xclonedata with genome_mode='hg38_genes', barcodes.tsv, and feature annotation files, attaching cell annotations with xclone.pp.extra_anno. Execute xclone.model.run_RDR, run_BAF, and run_combine under xclone.XCloneConfig with documented settings trans_t=1e-6 and BAF_denoise=True, and run inferCNV and Numbat on the identical simulated RDR and allelic inputs. Score copy-loss recovery per cell and per gene against the ground-truth labels into auroc_benchmark.csv; the expected pattern is XClone (AUROC 0.9964) > inferCNV (0.9568) > Numbat (0.2971, which entirely misses the chr3 copy loss).

## Related tools

- **scCNASimulator** (Seeds the GX109-T1c 10x 5' scRNA-seq data with an allele-specific chr3 copy loss in two subclones and regenerates the simulated BAM with ground-truth labels)
- **xcltk** (Preprocesses the simulated SAM/BAM/CRAM into the three input matrices (total read depth RDR; allele-specific AD/DP for BAF)) — https://github.com/hxj5/xcltk
- **XClone** (Primary caller under test: run_RDR, run_BAF, and run_combine modules with trans_t=1e-6 and BAF_denoise=True produce the copy-loss calls scored by AUROC) — https://github.com/single-cell-genetics/XClone
- **InferCNV** (Comparator copy-loss caller run on the identical simulated RDR inputs)
- **Numbat** (Comparator allele-aware caller run on the identical simulated inputs; benchmarked for whether it recovers the chr3 loss)

## Examples

```
conda create -n xclone python=3.9; conda activate xclone; pip install xclone; pip install -U xcltk
```

## Evaluation signals

- auroc_benchmark.csv exists and reports AUROC for all three callers against the simulator's ground-truth labels; expected ordering XClone (0.9964) > inferCNV (0.9568) > Numbat (0.2971, Numbat entirely missing the chr3 copy loss)
- xcltk produces exactly three matrices — total read depth (RDR) and allele-specific AD/DP (BAF) — with dimensions matching the simulated BAM's cells and genes
- AnnData loads via xclone.pp.xclonedata with genome_mode='hg38_genes' and barcodes.tsv/feature annotation files resolve without missing entries
- Ground-truth copy-loss labels are confined to chromosome 3 in the two injected subclones; all other chromosomes and non-subclone cells must be labelled neutral
- XClone config uses the documented trans_t=1e-6 and BAF_denoise=True across the RDR, BAF, and Combine module runs

## Limitations

- Results are simulation-dependent: Numbat's pseudo-bulk resolution can entirely miss a single injected event (chr3 loss), so single-event benchmarks may not reflect performance on complex, multi-clonal CNA landscapes.
- InferCNV and CopyKAT do not use allelic information and hence cannot detect copy-neutral LOH, limiting the benchmark to read-depth-detectable copy losses.
- XClone recommends Python 3.9 for stable performance; other versions may be unstable.
- The Rongtingting/xclone-data repository is explicitly 'preliminary' with limited documentation.

## Evidence

- [methods] Seed scCNASimulator with the GX109-T1c gastric cancer 10x 5' scRNA-seq dataset and inject an allele-specific copy loss on chromosome 3 across two subclones to regenerate the simulated BAM.: [evidence span withheld — anchor status: not_found]
- [methods] Run the three XClone modules with documented settings trans_t=1e-6 and BAF_denoise=True.: [evidence span withheld — anchor status: not_found]
- [results] In the GX109-T1c benchmark, XClone achieved AUROC 0.9964 for chr3 copy-loss detection versus inferCNV 0.9568 and Numbat 0.2971.: [evidence span withheld — anchor status: not_found]
- [readme] xcltk generates the three XClone input matrices from SAM/BAM/CRAM files.: [evidence span withheld — anchor status: external_doc]
- [readme] XClone consumes three matrices: allele-specific AD/DP (BAF) and total read depth (RDR).: [evidence span withheld — anchor status: external_doc]
- [intro] Numbat may sacrifice single-cell CNA resolution to a pseudo-bulk level, explaining its benchmark failure mode.: "However, this method may sacrifice the resolution of clonal CNAs from a single-cell level to a pseudo-bulk level"
