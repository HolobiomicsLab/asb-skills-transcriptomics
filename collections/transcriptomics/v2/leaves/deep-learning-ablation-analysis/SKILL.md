---
name: deep-learning-ablation-analysis
description: Use when benchmarking a fusion caller or similar ML-assisted pipeline
  and you must quantify how much a learned filtering component contributes, by comparing
  full-pipeline precision/recall against a variant with the deep-learning model turned
  off on the same inputs.
when_to_use_negative:
- Input is 10X droplet scRNA-seq data rather than Smart-Seq protocol FASTQ files,
  since scFusion and this benchmark are designed for Smart-Seq (_1.fastq/_2.fastq,
  *_1.fq not allowed).
- No simulation ground truth or known fusion set exists, so precision/recall/F-score
  deltas between full and ablated runs cannot be computed.
- The pipeline has no learned/ML component to ablate, so a deep-learning-off comparison
  is meaningless.
tools:
- name: scFusion
  role: 'Pipeline under ablation: run scFusion.py twice — full deep-learning model
    on, and ablation variant with tensorflow/keras model turned off — to measure the
    model''s precision contribution'
  repo: https://github.com/ZijieJin/scFusion
- name: STAR
  role: Spliced aligner used identically for both full and ablated scFusion runs on
    simulated Smart-Seq FASTQ input
  repo: https://github.com/alexdobin/STAR
- name: samtools
  role: Read processing (BAM handling) shared by both ablation arms to keep alignment
    downstream steps identical
- name: bedtools
  role: Genomic interval operations for read processing in both ablation arms
- name: tensorflow/keras
  role: The deep-learning model that is disabled in the ablation variant; its removal
    quantifies the precision gain of learned false-positive filtering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:a7639384d9e0ef77779f662364a294117224e16608365ecaa3dfcd62736d8455
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZijieJin/scFusion
  tools:
  - Python
  - R
  - STAR
  - bedtools
  - pyensembl
  - samtools
  - scFusion
  - stringr
  - tensorflow/keras
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# deep-learning-ablation-analysis

## Summary

Ablation-analysis skill for verifying that a machine-learning component of a bioinformatics pipeline genuinely improves performance: rerun the pipeline with the model disabled on identical inputs and quantify the precision/sensitivity trade-off. In scFusion, this means disabling the tensorflow/keras deep-learning model that filters false-positive fusion candidates in scRNA-seq data.

## When to use

Use when benchmarking a fusion caller or similar ML-assisted pipeline and you must quantify how much a learned filtering component contributes, by comparing full-pipeline precision/recall against a variant with the deep-learning model turned off on the same inputs.

## When NOT to use

- Input is 10X droplet scRNA-seq data rather than Smart-Seq protocol FASTQ files, since scFusion and this benchmark are designed for Smart-Seq (_1.fastq/_2.fastq, *_1.fq not allowed).
- No simulation ground truth or known fusion set exists, so precision/recall/F-score deltas between full and ablated runs cannot be computed.
- The pipeline has no learned/ML component to ablate, so a deep-learning-off comparison is meaningless.

## Inputs

- Simulated Smart-Seq paired-end scRNA-seq FASTQ files (*_1.fastq, *_2.fastq) with simulation ground-truth fusion annotations
- Reference genome FASTA (e.g., hg19.fa, ~3 GB)
- GTF annotation file (Ensembl, NCBI, or UCSC)
- hg19 mappability file (from UCSC hgTables or bundled hg19mappability)
- Trained deep-learning model weights within the pipeline (tensorflow/keras)

## Outputs

- benchmark_results.csv with per-replicate precision, recall, and F-score for full vs. ablation variants across all 6 setups (2 cell counts x 3 depths x 10 replicates)
- Ablation delta report: precision gain range and sensitivity loss when the deep-learning model is disabled

## How to apply

Generate benchmark inputs with known ground truth — here, simulated Smart-Seq paired-end scRNA-seq FASTQ files (500- and 1000-cell datasets at 2, 3, and 4 million reads per cell, 1% artificial chimeric fusion reads, 10 replicates per setting) built with FusionSimulatorToolkit. Run the full pipeline (scFusion.py with STAR >= 2.7.2d alignment, hg19 reference and GTF annotation, hg19 mappability filter) and then the ablation variant with the deep-learning model (tensorflow/keras) turned off, keeping every other parameter identical. Score both variants against the simulation ground truth to compute precision, recall, and F-score, then aggregate across the 10 replicates into benchmark_results.csv. Judge success by the ablation delta: turning off the deep-learning model should reduce precision (article-reported gain of 0.049-0.168) while sensitivity loss stays below 0.04, confirming the model controls false positives without sacrificing recall.

## Related tools

- **scFusion** (Pipeline under ablation: run scFusion.py twice — full deep-learning model on, and ablation variant with tensorflow/keras model turned off — to measure the model's precision contribution) — https://github.com/ZijieJin/scFusion
- **STAR** (Spliced aligner used identically for both full and ablated scFusion runs on simulated Smart-Seq FASTQ input) — https://github.com/alexdobin/STAR
- **samtools** (Read processing (BAM handling) shared by both ablation arms to keep alignment downstream steps identical)
- **bedtools** (Genomic interval operations for read processing in both ablation arms)
- **tensorflow/keras** (The deep-learning model that is disabled in the ablation variant; its removal quantifies the precision gain of learned false-positive filtering)

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- Ablation delta check: disabling the deep-learning model reduces precision (expected gain of 0.049-0.168 for the full model) while sensitivity loss remains below 0.04.
- Full-pipeline baseline check: scFusion reaches precision ~0.921 and F-score ~0.925 in the 1000-cell, 4-million-reads-per-cell simulation, versus 0.434-0.503 precision for bulk methods (Arriba, STAR-Fusion, FusionCatcher, EricScript).
- Faster/simple ablation variant should not outperform the full model on precision in any of the 6 simulated setups (500/1000 cells x 2/3/4 million reads per cell).
- Aggregate reproducibility: results aggregated over 10 replicates per setting land in benchmark_results.csv with stable cross-replicate variance.
- Environment check: tensorflow, keras, and numpy satisfy the README-pinned versions (2.8.0/2.8.0/1.22.3 or 2.3.0/2.4.3/1.18.5) so the on/off comparison tests the intended model.

## Limitations

- Fusions with low expression have reduced sensitivity — common to all RNA-based methods — so ablation deltas at low expression may be noisy regardless of the deep-learning model.
- Detection power is limited for rare fusions in highly heterogeneous tumor samples; such fusions can be overcome only by sequencing more cells and/or sequencing each cell deeper.
- Deep-learning ablation results are grounded in simulated data with 1% artificial chimeric reads; real-data behavior of the ablation delta may differ.

## Evidence

- [intro] Objective and rationale for the deep-learning component: "scFusion employs a statistical model and a deep-learning model to control for false positives."
- [discussion] Effect of disabling the deep-learning model: "When deep-learning model was turned off, the precisions of scFusion reduced"
- [intro] Simulation-based benchmark of sensitivity and false discovery rate: "we first simulated single-cell data based on real scRNA-seq data and found that scFusion achieved high sensitivity while maintaining a low false discovery rate"
- [readme] Pinned deep-learning dependency versions required to reproduce the full vs. ablated comparison: "python module: tensorflow, keras, and numpy (version 2.8.0, 2.8.0, and 1.22.3, respectively)"
- [abstract] Overall performance claim the ablation analysis supports: "scFusion can efficiently and sensitively detect fusions with a low false discovery rate."
