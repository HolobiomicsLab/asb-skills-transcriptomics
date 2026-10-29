---
name: vdj-recombination-identification
description: Use when analyzing single-cell T cell scRNA-seq (Smart-Seq FASTQ) and
  the goal is to detect V(D)J recombination fusions marking MAIT cells that bulk RNA
  fusion methods miss.
when_to_use_negative:
- Data are bulk RNA-seq rather than single-cell Smart-Seq scRNA-seq; scFusion is designed
  for Smart-Seq single-cell data, not 10X or bulk inputs
- FASTQ files are named *_1.fq instead of *_1.fastq, which the scFusion pipeline explicitly
  disallows
- The goal is to detect rare fusions such as TRAJ20-TRAV1-2 in a small cell population
  — scFusion may miss these (TRAJ20-TRAV1-2 was not identified, possibly because it
  was rare in MAIT cells)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2830
tools:
- name: scFusion
  role: Primary pipeline that jointly analyzes all single cells to call gene fusions,
    including V(D)J recombinations, using statistical and deep-learning false-positive
    filters
  repo: https://github.com/ZijieJin/scFusion
- name: STAR
  role: Aligns single-cell chimeric reads (version >= 2.7.2d, tested on 2.7.2d and
    2.7.8a) to hg19 to generate candidate fusion alignments
  repo: https://github.com/alexdobin/STAR
- name: samtools
  role: Processes STAR alignments in the scFusion.py pipeline (version >= 1.0, tested
    on 1.10)
- name: bedtools
  role: Applies interval operations, including the mappability filter, to alignment
    records (tested on version 2.29.2)
- name: pyensembl
  role: Handles Ensembl GTF annotation for annotating fusion gene partners (e.g.,
    TRAJ/TRAV segments)
  repo: https://github.com/openvax/pyensembl
- name: pysam
  role: Python module used in the scFusion pipeline for BAM file parsing (tested on
    version 0.18.0)
- name: tensorflow/keras/numpy
  role: Run the deep-learning model that filters scFusion false-positive fusion candidates
    (versions 2.8.0, 2.8.0, 1.22.3)
- name: scipy
  role: Python option for Fisher's exact test on the SLC4A10-vs-fusion 2x2 contingency
    table
- name: stringr (R)
  role: R package used for parsing in the Fisher's exact test validation step (R >=
    3.5)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:de295978e971fa1ed14aa90274b23fb1c3603b0233508ab13be68c3568033f6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZijieJin/scFusion
  tools:
  - Python
  - R
  - STAR
  - bedtools
  - numpy
  - pyensembl
  - pysam
  - samtools
  - scFusion
  - scipy
  - stringr
  - stringr (R)
  - tensorflow/keras/numpy
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 20
---

# vdj-recombination-identification

## Summary

Detect invariant TCR V(D)J gene recombinations (e.g., TRAJ33-TRAV1-2, TRAJ12-TRAV1-2) from Smart-Seq scRNA-seq data using scFusion, where bulk-oriented fusion callers like STAR-Fusion fail, and validate them against the MAIT cell marker SLC4A10 via Fisher's exact test.

## When to use

Use when analyzing single-cell T cell scRNA-seq (Smart-Seq FASTQ) and the goal is to detect V(D)J recombination fusions marking MAIT cells that bulk RNA fusion methods miss.

## When NOT to use

- Data are bulk RNA-seq rather than single-cell Smart-Seq scRNA-seq; scFusion is designed for Smart-Seq single-cell data, not 10X or bulk inputs
- FASTQ files are named *_1.fq instead of *_1.fastq, which the scFusion pipeline explicitly disallows
- The goal is to detect rare fusions such as TRAJ20-TRAV1-2 in a small cell population — scFusion may miss these (TRAJ20-TRAV1-2 was not identified, possibly because it was rare in MAIT cells)

## Inputs

- 2355 single-cell T cell Smart-Seq FASTQ file pairs (*_1.fastq / *_2.fastq) from GEO
- hg19 reference genome FASTA (hg19.fa, ~3G)
- GTF annotation file from Ensembl, NCBI, or UCSC
- hg19 mappability file (chr, start, end, value 4-column format, unzipped in data folder)

## Outputs

- scFusion fusion call table for the T cell dataset (8 detected TCR recombinations with per-fusion cell counts, e.g., TRAJ33-TRAV1-2 in 126 cells, TRAJ12-TRAV1-2 in 20 cells)
- Fisher's exact test result for SLC4A10 expression vs. TCR fusion detection (P < 10^-16)
- Comparison of scFusion V(D)J calls vs. STAR-Fusion candidate fusions (STAR-Fusion V(D)J fraction = 1.8%)

## How to apply

Obtain the Smart-Seq FASTQ pairs named *_1.fastq/_2.fastq (e.g., 1_1.fastq, 1_2.fastq; *_1.fq is not allowed), stage hg19.fa, a GTF annotation, and the unzipped hg19mappability file in the data folder, then run scFusion.py, which aligns cells with STAR (>= 2.7.2d), processes alignments with samtools (>= 1.0) and bedtools (2.29.2), applies the 4-column mappability filter, and filters candidates with a statistical model, a deep-learning model (tensorflow/keras/numpy), and ad hoc filters against amplification-artifact chimeric reads. Compile fusion calls and check for TCR recombinations such as TRAJ33-TRAV1-2 (126 cells) and TRAJ12-TRAV1-2 (20 cells) among the 8 detected TCR recombinations in the 2355-cell T cell dataset. Validate biologically by building a 2x2 contingency table of SLC4A10 expression versus fusion detection in R (stringr) or Python scipy and running Fisher's exact test, expecting P < 10^-16, and by confirming only 1.8% of STAR-Fusion candidates are V(D)J recombinations.

## Related tools

- **scFusion** (Primary pipeline that jointly analyzes all single cells to call gene fusions, including V(D)J recombinations, using statistical and deep-learning false-positive filters) — https://github.com/ZijieJin/scFusion
- **STAR** (Aligns single-cell chimeric reads (version >= 2.7.2d, tested on 2.7.2d and 2.7.8a) to hg19 to generate candidate fusion alignments) — https://github.com/alexdobin/STAR
- **samtools** (Processes STAR alignments in the scFusion.py pipeline (version >= 1.0, tested on 1.10))
- **bedtools** (Applies interval operations, including the mappability filter, to alignment records (tested on version 2.29.2))
- **pyensembl** (Handles Ensembl GTF annotation for annotating fusion gene partners (e.g., TRAJ/TRAV segments)) — https://github.com/openvax/pyensembl
- **pysam** (Python module used in the scFusion pipeline for BAM file parsing (tested on version 0.18.0))
- **tensorflow/keras/numpy** (Run the deep-learning model that filters scFusion false-positive fusion candidates (versions 2.8.0, 2.8.0, 1.22.3))
- **scipy** (Python option for Fisher's exact test on the SLC4A10-vs-fusion 2x2 contingency table)
- **stringr (R)** (R package used for parsing in the Fisher's exact test validation step (R >= 3.5))

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- The fusion table reproduces the 8 TCR recombinations with reported per-fusion cell counts (TRAJ33-TRAV1-2 supported by 126 cells, TRAJ12-TRAV1-2 by 20 cells)
- Fisher's exact test of SLC4A10 expression against fusion detection yields P < 10^-16, matching Table 1
- Only 1.8% of STAR-Fusion candidate fusions correspond to V(D)J recombinations
- No other bulk RNA-seq fusion method detects any of these V(D)J recombinations
- All input FASTQ pairs follow the *_1.fastq / *_2.fastq naming convention and the mappability filter file is present as a 4-column chr/start/end/value table

## Limitations

- TRAJ20-TRAV1-2 was not identified, possibly because it was rare in MAIT cells
- scFusion has reduced sensitivity for fusions with low expression, as is the case for all RNA-based methods
- Detection power is limited for rare fusions in highly heterogeneous samples; inherent limitations can be overcome only by sequencing more cells and/or deeper
- The mappability filter is turned off if the mappability file is not provided, potentially increasing false positives
- Turning off the deep-learning model reduced the precision of scFusion, and many potential artifacts are only removed by the ad hoc filters

## Evidence

- [abstract] In a T cell dataset, scFusion detects the invariant TCR gene recombinations in mucosal-associated invariant T cells that many methods developed for bulk data fail to detect: "In a T cell dataset, scFusion detects the invariant TCR gene recombinations in mucosal-associated invariant T cells that many methods developed for bulk data fail to detect"
- [results] the two most frequent V(D)J recombinations: [evidence span withheld — anchor status: not_found]
- [results] Fisher's exact test: [evidence span withheld — anchor status: not_found]
- [methods] STAR version requirements: "STAR (https://github.com/alexdobin/STAR) >= 2.7.2d (tested on 2.7.2d and 2.7.8a)"
- [readme] FASTQ naming convention: "Single cell RNA sequencing files from Smart-Seq protocol. File names should be *_1.fastq, *_2.fastq (e.g. 1_1.fastq, 1_2.fastq, 2_1.fastq, 2_2.fastq)"
- [discussion] TRAJ20-TRAV1-2 missed: "The TRAJ20-TRAV1-2 was not identified possibly because it was rare in MAIT cells"
