---
name: pcawg-fusion-artifact-score-validation
description: Use when scoring candidate gene fusions from Smart-Seq scRNA-seq data
  for technical artifact likelihood, or when validating PCAWG fusion calls by assigning
  an artifact probability and filtering at a 0.5 cutoff.
when_to_use_negative:
- Input reads are 10X scRNA-seq data — scFusion and this artifact-scoring workflow
  target Smart-Seq data only
- FASTQ files are named *_1.fq — the scFusion pipeline requires the *.fastq extension
  and will not accept *_1.fq
- The mappability filter file is unavailable and a fully unfiltered candidate set
  is desired — omitting the UCSC file silently disables the mappability filter rather
  than erroring
tools:
- name: scFusion
  role: Pipeline producing candidate fusion features that the bi-LSTM artifact scorer
    consumes; applies the deep-learning artifact model and ad hoc filters
  repo: ZijieJin/scFusion
- name: STAR >= 2.7.2d
  role: Aligns single-cell Smart-Seq FASTQ pairs to the hg19 reference genome to support
    chimeric-read detection
  repo: github.com/alexdobin/STAR
- name: samtools >= 1.0
  role: Processes STAR alignments before chimeric/fusion-supporting candidate extraction
- name: bedtools
  role: Extracts chimeric/fusion-supporting candidate reads from processed alignments
- name: pyensembl
  role: Annotates fusion candidates from the GTF ('gene_name', 'gene_type'/'gene_biotype'
    fields)
  repo: github.com/openvax/pyensembl
- name: TensorFlow/Keras
  role: Builds and trains the bi-LSTM model predicting artifact vs. real fusions (tested
    with tensorflow 2.8.0/keras 2.8.0/numpy 1.22.3)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:b39403089c6499c430c415c59aa946cb2504cea05fdcf056ff11ad47d1a5530d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZijieJin/scFusion
  tools:
  - Keras
  - Python
  - R
  - STAR >= 2.7.2d
  - TensorFlow
  - TensorFlow/Keras
  - bedtools
  - numpy
  - pyensembl
  - pysam
  - samtools
  - samtools >= 1.0
  - scFusion
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# pcawg-fusion-artifact-score-validation

## Summary

Train and validate a bi-directional LSTM (TensorFlow/Keras) artifact-scoring model, built on scFusion candidate fusion features, that distinguishes technical chimeric artifacts from real fusions, then apply it to PCAWG fusion calls to filter artifacts. Validation benchmarks on six public scRNA-seq datasets target a median ROC AUC of 0.884 and PR AUPR of 0.913.

## When to use

Use when scoring candidate gene fusions from Smart-Seq scRNA-seq data for technical artifact likelihood, or when validating PCAWG fusion calls by assigning an artifact probability and filtering at a 0.5 cutoff.

## When NOT to use

- Input reads are 10X scRNA-seq data — scFusion and this artifact-scoring workflow target Smart-Seq data only
- FASTQ files are named *_1.fq — the scFusion pipeline requires the *.fastq extension and will not accept *_1.fq
- The mappability filter file is unavailable and a fully unfiltered candidate set is desired — omitting the UCSC file silently disables the mappability filter rather than erroring

## Inputs

- Paired Smart-Seq single-cell FASTQ files named *_1.fastq and *_2.fastq (e.g. 1_1.fastq, 1_2.fastq; *_1.fq not allowed)
- Six publicly available scRNA-seq datasets (GEO accessions from refs 23,30-34) for model training/validation
- hg19 reference genome FASTA (~3G, e.g. hg19.fa)
- GTF annotation file (Ensembl/NCBI/UCSC) with 'gene_name' and 'gene_type'/'gene_biotype' fields
- 4-column UCSC hg19 mappability file (hg19mappability, from genome.ucsc.edu hgTables)
- PCAWG fusion calls for artifact scoring

## Outputs

- Per-dataset ROC AUC and precision-recall AUPR for held-out fusion candidates
- Artifact scores (< 0.5 = likely artifact) assigned to each PCAWG fusion call
- model_evaluation_metrics.csv with per-dataset ROC AUC, PR AUPR, and PCAWG artifact-score summary

## How to apply

Retrieve the six public scRNA-seq datasets (GEO accessions from refs 23,30-34) as paired Smart-Seq FASTQ files named *_1.fastq and *_2.fastq, align each pair to hg19 with STAR (>= 2.7.2d), and use samtools and bedtools to extract chimeric/fusion-supporting candidate reads. Annotate candidates with pyensembl using the GTF 'gene_name' and 'gene_type'/'gene_biotype' fields and apply the 4-column UCSC hg19 mappability filter (omitting the file disables the filter). Build the bi-LSTM in TensorFlow/Keras, train on candidate fusion features to predict artifact vs. real fusions, and score held-out candidates per dataset, computing ROC AUC and PR AUPR. Apply the trained model to PCAWG fusion calls and check that >90% of fusions receive artifact scores < 0.5, then write per-dataset metrics to model_evaluation_metrics.csv. Turning off the deep-learning model reduces scFusion precision, so the LSTM artifact score is the key discriminator beyond the ad hoc filters.

## Related tools

- **scFusion** (Pipeline producing candidate fusion features that the bi-LSTM artifact scorer consumes; applies the deep-learning artifact model and ad hoc filters) — ZijieJin/scFusion
- **STAR >= 2.7.2d** (Aligns single-cell Smart-Seq FASTQ pairs to the hg19 reference genome to support chimeric-read detection) — github.com/alexdobin/STAR
- **samtools >= 1.0** (Processes STAR alignments before chimeric/fusion-supporting candidate extraction)
- **bedtools** (Extracts chimeric/fusion-supporting candidate reads from processed alignments)
- **pyensembl** (Annotates fusion candidates from the GTF ('gene_name', 'gene_type'/'gene_biotype' fields)) — github.com/openvax/pyensembl
- **TensorFlow/Keras** (Builds and trains the bi-LSTM model predicting artifact vs. real fusions (tested with tensorflow 2.8.0/keras 2.8.0/numpy 1.22.3))

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- Median ROC AUC across the six public datasets equals 0.884
- Median precision-recall AUPR across the six datasets equals 0.913
- Fraction of PCAWG fusions with artifact score < 0.5 exceeds 90% (Fig 2c-f)
- All input FASTQs follow the *_1.fastq / *_2.fastq naming convention (no *_1.fq files)
- model_evaluation_metrics.csv is produced with per-dataset ROC AUC, PR AUPR, and the PCAWG artifact-score summary

## Limitations

- Deep-learning model is essential for precision: turning it off reduced scFusion precision, and many potential artifacts are also filtered only by the ad hoc filters
- Reduced sensitivity for fusions with low expression, common to all RNA-based methods (shown in simulation)
- Limited detection power for rare fusions in highly heterogeneous tumor samples; inherent limitations can be overcome only by sequencing more cells and/or deeper per cell

## Evidence

- [intro] deep learning controls false positives: "scFusion employs a statistical model and a deep-learning model to control for false positives."
- [methods] STAR alignment requirement: "STAR (https://github.com/alexdobin/STAR) >= 2.7.2d (tested on 2.7.2d and 2.7.8a)"
- [methods] mappability filter optional: "If you do not provide this file, scFusion will turn off the mappability filter."
- [discussion] deep learning improves precision: "When deep-learning model was turned off, the precisions of scFusion reduced"
- [readme] TensorFlow/Keras versions: "python module: tensorflow, keras, and numpy (version 2.8.0, 2.8.0, and 1.22.3, respectively)"
