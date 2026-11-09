---
name: scrna-seq-cna-simulation
description: Use when you need ground-truth CNA labels to benchmark copy-number callers
  on single-cell RNA-seq, by simulating known events (e.g. allele-specific chr3 loss)
  in a seeded tumor dataset.
when_to_use_negative:
- The dataset already has clonal ground truth from matched scDNA-seq — benchmark directly
  on it instead of simulating
- The analysis goal is biological CNA detection on a real tumor sample, not caller
  benchmarking
- The comparison targets RDR-only CNA events where allelic imbalance carries no signal
  (e.g. copy-neutral events cannot be resolved by inferCNV/CopyKAT)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scCNASimulator
  role: Seeds real scRNA-seq data with injected allele-specific CNA events to produce
    simulated BAMs and ground-truth labels
- name: xcltk
  role: Preprocesses simulated BAMs into RDR read-depth and allele-specific AD/DP
    matrices for all three callers
  repo: https://github.com/hxj5/xcltk
- name: XClone
  role: Primary CNA caller under test; runs RDR, BAF, and Combine modules with trans_t
    = 1e-6 and BAF_denoise = True
  repo: github.com/single-cell-genetics/XClone
- name: InferCNV
  role: Comparator caller scored on the identical simulated RDR inputs (AUROC 0.9568)
- name: Numbat
  role: Comparator caller scored on the identical simulated inputs (AUROC 0.2971;
    missed the chr3 copy loss)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:e1fe670781d21e29b95d2e4917471ab1d223a9a6ccfe0820da90a43854c41bff
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# scrna-seq-cna-simulation

## Summary

Benchmark allele-specific copy-number callers (XClone, Numbat, inferCNV) on simulated scRNA-seq data generated with scCNASimulator, which injects known CNA events (e.g. allele-specific chr3 copy loss in two subclones) into a real seed dataset such as GX109-T1c to create ground-truth labels for AUROC evaluation.

## When to use

Use when you need ground-truth CNA labels to benchmark copy-number callers on single-cell RNA-seq, by simulating known events (e.g. allele-specific chr3 loss) in a seeded tumor dataset.

## When NOT to use

- The dataset already has clonal ground truth from matched scDNA-seq — benchmark directly on it instead of simulating
- The analysis goal is biological CNA detection on a real tumor sample, not caller benchmarking
- The comparison targets RDR-only CNA events where allelic imbalance carries no signal (e.g. copy-neutral events cannot be resolved by inferCNV/CopyKAT)

## Inputs

- Seed tumor scRNA-seq dataset with known/derivable CNA state (e.g. GX109-T1c gastric cancer 10x 5' scRNA-seq)
- scCNASimulator CNA injection specification (chromosome, event type, subclone membership)
- Simulated BAM files from scCNASimulator
- Genome annotation files for hg38 (genes, blocks) and barcodes.tsv

## Outputs

- Simulated BAM with known ground-truth CNA labels per cell and gene
- xcltk RDR read-depth count matrix and allele-specific AD/DP count matrices
- Per-caller CNA calls from XClone (RDR/BAF/Combine), inferCNV, and Numbat
- auroc_benchmark.csv with copy-loss recovery AUROCs per cell and per gene for each caller

## How to apply

Seed scCNASimulator with a real droplet scRNA-seq tumor dataset (e.g. the GX109-T1c gastric cancer 10x 5' sample) and inject the CNA of interest — here an allele-specific copy loss on chromosome 3 across two subclones — to regenerate simulated BAM files. Preprocess the simulated BAM with xcltk to obtain three matrices: the RDR read-depth count matrix and the allele-specific AD/DP matrices (BAF signals). Run XClone on xclone.pp.xclonedata-loaded AnnData (genome_mode = 'hg38_genes' with barcodes.tsv and feature annotation) through its three modules (run_RDR, run_BAF with BAF_denoise = True, run_combine with trans_t = 1e-6), and run inferCNV and Numbat on the identical simulated RDR and allelic inputs. Score copy-loss recovery per cell and per gene against the simulator's ground-truth CNA labels to compute AUROCs for each caller and save them to auroc_benchmark.csv. In the GX109-T1c chr3-loss benchmark, XClone achieved AUROC 0.9964 versus inferCNV 0.9568 and Numbat 0.2971, with Numbat missing the chr3 copy loss entirely — a sanity check that the simulation exposes caller-specific strengths on allele-specific events.

## Related tools

- **scCNASimulator** (Seeds real scRNA-seq data with injected allele-specific CNA events to produce simulated BAMs and ground-truth labels)
- **xcltk** (Preprocesses simulated BAMs into RDR read-depth and allele-specific AD/DP matrices for all three callers) — https://github.com/hxj5/xcltk
- **XClone** (Primary CNA caller under test; runs RDR, BAF, and Combine modules with trans_t = 1e-6 and BAF_denoise = True) — github.com/single-cell-genetics/XClone
- **InferCNV** (Comparator caller scored on the identical simulated RDR inputs (AUROC 0.9568))
- **Numbat** (Comparator caller scored on the identical simulated inputs (AUROC 0.2971; missed the chr3 copy loss))

## Examples

```
conda create -n xclone python=3.9 && conda activate xclone && pip install xclone && pip install -U xcltk
```

## Evaluation signals

- XClone output AnnData loads via xclone.pp.xclonedata with genome_mode = 'hg38_genes' matching the simulated barcodes and gene annotations
- xcltk produces exactly three matrices (RDR total read depth, AD, DP) with consistent cell × gene dimensions across all callers' inputs
- Per-cell/per-gene AUROC against simulator ground-truth labels is computed for all three callers and written to auroc_benchmark.csv
- Expected benchmark result: XClone AUROC 0.9964 > inferCNV 0.9568 > Numbat 0.2971 for the allele-specific chr3 loss on GX109-T1c seed data
- Allele-specific (BAF) signal must reach the comparator callers too, otherwise allele-specific events like chr3 loss will be missed (as Numbat did here)

## Limitations

- Numbat's performance can degrade on simulated allele-specific events because it may sacrifice resolution of clonal CNAs from single-cell to pseudo-bulk level; its robustness 'remain[s] to be further assessed'
- InferCNV does not consider allelic information and hence cannot detect copy-neutral LOH, limiting comparability on allele-specific events
- Python 3.9 (Anaconda) is recommended for stable XClone performance; the environment file xclone0.3.6Python3.9_env.yml documents the pinned versions
- Results are benchmark-specific: simulated subclone compositions and event sizes may not generalize to real tumor samples

## Evidence

- [methods] Step1: :ref:`Install xcltk <xcltk installation>` and run xctlkt :ref:`RDR module <xcltk RDR>` and :ref:`BAF module <xcltk BAF>`, independently.: "Step1: :ref:`Install xcltk <xcltk installation>` and run xctlkt :ref:`RDR module <xcltk RDR>` and :ref:`BAF module <xcltk BAF>`, independently."
- [intro] we introduce XClone, a statistical model to detect allele-specific CNAs on individual cells by modeling the raw read/UMI counts from scRNA-seq data: "we introduce XClone, a statistical model to detect allele-specific CNAs on individual cells by modeling the raw read/UMI counts from scRNA-seq data"
- [readme] XClone is an algorithm to infer allele- and haplotype-specific copy numbers in individual cells from low-coverage and sparse single-cell RNA sequencing data: "XClone is an algorithm to infer allele- and haplotype-specific copy numbers in individual cells from low-coverage and sparse single-cell RNA sequencing data"
- [methods] xconfig.BAF_denoise = True: "xconfig.BAF_denoise = True"
