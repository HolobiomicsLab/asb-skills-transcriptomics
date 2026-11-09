---
name: single-cell-gene-fusion-detection
description: Use when Smart-Seq single-cell FASTQ files (*_1.fastq, *_2.fastq) must
  be screened for gene fusions, e.g. recovering known fusions (IgH-WHSC1, TMPRSS2-ERG,
  TCR V(D)J recombinations) or benchmarking against bulk fusion callers.
when_to_use_negative:
- Input is 10X Genomics data — scFusion is built for Smart-Seq protocol data only
- FASTQ files follow the *_1.fq / *_2.fq naming convention, which scFusion does not
  accept
- The goal is detecting lowly expressed or highly rare fusions in highly heterogeneous
  tumors with few sequenced cells, where scFusion's power is limited
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3520
tools:
- name: scFusion
  role: 'Main pipeline: jointly analyzes all single cells with statistical, deep-learning,
    mappability, and ad hoc filters to call gene fusions'
  repo: https://github.com/ZijieJin/scFusion
- name: STAR
  role: Aligns single-cell reads (>= 2.7.2d) and reports chimeric/split alignments
    as fusion candidates
  repo: https://github.com/alexdobin/STAR
- name: samtools
  role: Post-processes sorted STAR BAMs (>= 1.0, tested on 1.10)
- name: bedtools
  role: Genomic interval processing of candidate chimeric reads (tested on 2.29.2)
- name: pyensembl
  role: Annotates candidate fusion gene pairs using GTF gene_name and gene_type fields
  repo: https://github.com/openvax/pyensembl
- name: TensorFlow/Keras
  role: Deep-learning filter to remove false-positive fusion candidates
- name: stringr
  role: R package (>= 3.5) for parsing fusion calls and building contingency tables
    for Fisher's exact tests
- name: pysam
  role: Python BAM handling during candidate extraction (tested on 0.18.0)
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:25406e8c73a713705398c81c9318b2467dd31870247f1d6897a2db65a8c54f8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZijieJin/scFusion
  tools:
  - Python
  - R
  - STAR
  - TensorFlow
  - TensorFlow/Keras
  - bedtools
  - numpy
  - pyensembl
  - pysam
  - samtools
  - scFusion
  - scipy
  - stringr
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.55
n_claims: 20
---

# single-cell-gene-fusion-detection

## Summary

Detect gene fusions at single-cell resolution from Smart-Seq scRNA-seq data using scFusion, which aligns reads with STAR and applies statistical, deep-learning, mappability, and ad hoc filters to control false positives from amplification artifacts.

## When to use

Use when Smart-Seq single-cell FASTQ files (*_1.fastq, *_2.fastq) must be screened for gene fusions, e.g. recovering known fusions (IgH-WHSC1, TMPRSS2-ERG, TCR V(D)J recombinations) or benchmarking against bulk fusion callers.

## When NOT to use

- Input is 10X Genomics data — scFusion is built for Smart-Seq protocol data only
- FASTQ files follow the *_1.fq / *_2.fq naming convention, which scFusion does not accept
- The goal is detecting lowly expressed or highly rare fusions in highly heterogeneous tumors with few sequenced cells, where scFusion's power is limited

## Inputs

- Paired-end Smart-Seq single-cell FASTQ files named *_1.fastq / *_2.fastq (*_1.fq not allowed)
- Reference genome FASTA (e.g. hg19.fa, ~3G)
- GTF annotation file (*.gtf) from Ensembl, NCBI, or UCSC
- hg19 mappability file, 4-column chr/start/end/value format (unzipped in the data folder; optional — filter is disabled if absent)

## Outputs

- Per-cell and per-dataset gene fusion calls with breakpoints and supporting cell counts
- Fusion candidate reads from chimeric/split alignments (STAR/samtools/bedtools intermediates)
- Summary tables (e.g. fusion_results.csv, fig5_fusion_comparison.csv, scfusion_results.csv) and contingency-table statistics such as Fisher's exact test p-values

## How to apply

Place paired-end Smart-Seq FASTQ files, the hg19 reference genome FASTA (~3G), a GTF annotation (Ensembl/NCBI/UCSC), and the unzipped hg19mappability 4-column file (chr/start/end/value) in the data folder, then run scFusion.py with 64 GB RAM and 8 cores per task. scFusion aligns each cell with STAR >= 2.7.2d, processes sorted BAMs with samtools >= 1.0 and bedtools 2.29.2 to extract chimeric/split reads, and annotates candidates with pyensembl. Candidates are filtered jointly across all single cells (leveraging fusions shared among cells) by a statistical model, a deep-learning model (tensorflow/keras), the mappability filter, and ad hoc filters that suppress amplification-artifact chimeric reads. Validate calls against known catalogs, e.g. all 27 PCAWG spike-in fusions recovered with only 24 extra calls, 38 fusions (94.7% immunoglobulin) in multiple myeloma with IgH-WHSC1 breakpoints at 1902353 (SMM0) and 1905943 (RRMM2), and 10 patient fusions including TMPRSS2-ERG in 27 prostate cancer cells.

## Related tools

- **scFusion** (Main pipeline: jointly analyzes all single cells with statistical, deep-learning, mappability, and ad hoc filters to call gene fusions) — https://github.com/ZijieJin/scFusion
- **STAR** (Aligns single-cell reads (>= 2.7.2d) and reports chimeric/split alignments as fusion candidates) — https://github.com/alexdobin/STAR
- **samtools** (Post-processes sorted STAR BAMs (>= 1.0, tested on 1.10))
- **bedtools** (Genomic interval processing of candidate chimeric reads (tested on 2.29.2))
- **pyensembl** (Annotates candidate fusion gene pairs using GTF gene_name and gene_type fields) — https://github.com/openvax/pyensembl
- **TensorFlow/Keras** (Deep-learning filter to remove false-positive fusion candidates)
- **stringr** (R package (>= 3.5) for parsing fusion calls and building contingency tables for Fisher's exact tests)
- **pysam** (Python BAM handling during candidate extraction (tested on 0.18.0))

## Examples

```
docker run -v $PWD:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py
```

## Evaluation signals

- Spike-in recovery: scFusion reports all 27 spike-in fusions with only 24 additional fusions, whereas bulk methods report 310–9044 with only 0.5–17.7% bulk-support (Fig 5)
- Multiple myeloma: recurrent IgH-WHSC1 called with breakpoints 1902353 in all 52 SMM0 cells and 1905943 in all 20 RRMM2 cells; 38 total fusions, 94.7% involving immunoglobulin genes; WHSC1 overexpression in fusion-positive cells (Fig 7d-e)
- Prostate cancer: 4 fusions in LNCaP (all in CCLE) and 10 in patient data including TMPRSS2-ERG in 27 cells; fusion positivity (16.7% vs 3.7%) vs enzalutamide gives Fisher's exact p = 0.0047 (Table 2)
- T cell dataset: 8 TCR recombinations detected, top being TRAJ33-TRAV1-2 (126 cells) and TRAJ12-TRAV1-2 (20 cells); SLC4A10 association P < 10^-16 (Table 1); only 1.8% of STAR-Fusion candidates are V(D)J recombinations
- Ablation check: turning off the deep-learning model reduces precision, confirming all filters are active

## Limitations

- Reduced sensitivity for fusions with low expression, common to all RNA-based methods
- Limited detection power for rare fusions in highly heterogeneous tumor samples; overcome only by sequencing more and/or deeper cells
- TRAJ20-TRAV1-2 was not identified, possibly because it was rare in MAIT cells
- WHSC1 fusions were not found in single RRMM2 cells — WHSC1 expression may be activated by other mechanisms or the fusion was missed
- Without the mappability file, scFusion turns off the mappability filter

## Evidence

- [readme] scFusion is a computational pipeline for detecting gene fusions at single-cell resolution (For Smart-Seq data rather than 10X).: [evidence span withheld — anchor status: external_doc]
- [intro] statistical and deep-learning models control false positives: "scFusion employs a statistical model and a deep-learning model to control for false positives."
- [intro] joint analysis boosts shared fusions: "the power for detecting fusions shared among multiple single cells can be improved if all single cells are jointly analyzed"
- [readme] Software prerequisites: "STAR >= 2.7.2d (tested on 2.7.2d and 2.7.8a)"
- [methods] mappability filter optional: "If you do not provide this file, scFusion will turn off the mappability filter."
- [abstract] bulk-data methods miss MAIT TCR recombinations: "In a T cell dataset, scFusion detects the invariant TCR gene recombinations in mucosal-associated invariant T cells that many methods developed for bulk data fail to detect"
- [abstract] WHSC1 overexpression finding: "in a multiple myeloma dataset, scFusion detects the known recurrent fusion IgH-WHSC1, which is associated with overexpression of the WHSC1 oncogene."
- [discussion] low sensitivity for low expression: "scFusion has reduced sensitivity for fusions with low expression (as shown in the simulation), just as is the case for all RNA-based methods"
