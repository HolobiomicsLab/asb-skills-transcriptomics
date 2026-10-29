---
name: fusion-call-cross-reference-validation-ccle
description: Use when scRNA-seq fusion calls (e.g., from scFusion on Smart-Seq data)
  need benchmarking against known fusion catalogs (CCLE, TMPRSS2-ERG in prostate cancer)
  or when fusion positivity must be tested for association with a cell-level phenotype.
when_to_use_negative:
- Input is 10X droplet scRNA-seq rather than Smart-Seq data — scFusion is designed
  for Smart-Seq only
- The goal is de novo fusion discovery with no known catalog available for cross-referencing
  or validation
- Fusions of interest are very lowly expressed or rare in highly heterogeneous tumors,
  where all RNA-based methods have reduced sensitivity
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scFusion
  role: Detects fusions at single-cell resolution using a statistical model plus deep-learning
    filter; output calls are cross-referenced against known catalogs
  repo: https://github.com/ZijieJin/scFusion
- name: STAR
  role: Aligns paired-end Smart-Seq FASTQ files to hg19 (>= 2.7.2d) as input to scFusion
  repo: https://github.com/alexdobin/STAR
- name: samtools
  role: Post-processing of STAR alignments before fusion calling
- name: bedtools
  role: Alignment post-processing supporting chimeric-read extraction
- name: pyensembl
  role: Python interface to Ensembl GTF/FASTA annotation used by the scFusion pipeline
  repo: https://github.com/openvax/pyensembl
- name: stringr
  role: R package used in building the fusion-positivity contingency table and Fisher's
    exact test in R >= 3.5
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:568c3803e3fd7fed2caaed6f7ce37de2785cc7c28b5f8b43f7afd240105891f1
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# fusion-call-cross-reference-validation-ccle

## Summary

Validate scFusion single-cell fusion calls by cross-referencing detected fusions against known catalogs such as CCLE and the known TMPRSS2-ERG fusion, and quantify fusion-associated phenotypes (e.g., enzalutamide response) with Fisher's exact tests on cell-level positivity contingency tables.

## When to use

Use when scRNA-seq fusion calls (e.g., from scFusion on Smart-Seq data) need benchmarking against known fusion catalogs (CCLE, TMPRSS2-ERG in prostate cancer) or when fusion positivity must be tested for association with a cell-level phenotype.

## When NOT to use

- Input is 10X droplet scRNA-seq rather than Smart-Seq data — scFusion is designed for Smart-Seq only
- The goal is de novo fusion discovery with no known catalog available for cross-referencing or validation
- Fusions of interest are very lowly expressed or rare in highly heterogeneous tumors, where all RNA-based methods have reduced sensitivity

## Inputs

- STAR-aligned BAM files from paired-end Smart-Seq FASTQ data (*_1.fastq, *_2.fastq; *_1.fq not allowed)
- hg19.fa reference genome (~3G) and GTF annotation (Ensembl/NCBI/UCSC)
- hg19 mappability file (4-column chr/start/end/value from UCSC hgTables) to enable the mappability filter
- Known fusion catalog for cross-referencing (e.g., CCLE fusion calls)
- Per-cell metadata (e.g., enzalutamide treatment status) for contingency-table testing

## Outputs

- Fusion call list with per-cell support (e.g., TMPRSS2-ERG in 27 patient cells; 10 patient fusions, 4 LNCaP fusions)
- Cross-reference table mapping detected fusions to CCLE/known-catalog entries
- 2x2 contingency table (fusion positivity 16.7% of 257 vs 3.7% of 665 cells) with Fisher's exact p-value 0.0047 (Table 2)
- scfusion_results.csv and ERG differential-expression comparison between fusion-positive and fusion-negative cells

## How to apply

Run the scFusion pipeline (scFusion.py with STAR-aligned Smart-Seq BAMs, hg19.fa, GTF annotation, and the hg19mappability file to enable the mappability filter, plus the TensorFlow/Keras deep-learning filter) on the aligned cells, then cross-reference each call against known fusion catalogs: in LNCaP cells all 4 detected fusions should be present in CCLE, and in the prostate patient dataset (922 epithelial cells) 10 fusions should be recovered including TMPRSS2-ERG despite the short 38 bp reads. Build a 2x2 contingency table of fusion positivity versus treatment (TMPRSS2-ERG positive in 16.7% of 257 enzalutamide-treated vs 3.7% of 665 untreated cells) in R (>=3.5, stringr) and compute Fisher's exact test, expecting p=0.0047 (Table 2). Follow up with differential expression testing of ERG between fusion-positive and fusion-negative cells to confirm transcriptional impact at the single-cell level. Rationale: scFusion jointly analyzes all cells with a statistical model and a deep-learning model to control false positives from PCR-generated artificial chimeric reads, so recovery of the known recurrent fusion and expected counts are the primary correctness criteria.

## Related tools

- **scFusion** (Detects fusions at single-cell resolution using a statistical model plus deep-learning filter; output calls are cross-referenced against known catalogs) — https://github.com/ZijieJin/scFusion
- **STAR** (Aligns paired-end Smart-Seq FASTQ files to hg19 (>= 2.7.2d) as input to scFusion) — https://github.com/alexdobin/STAR
- **samtools** (Post-processing of STAR alignments before fusion calling)
- **bedtools** (Alignment post-processing supporting chimeric-read extraction)
- **pyensembl** (Python interface to Ensembl GTF/FASTA annotation used by the scFusion pipeline) — https://github.com/openvax/pyensembl
- **stringr** (R package used in building the fusion-positivity contingency table and Fisher's exact test in R >= 3.5)

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- LNCaP cells yield exactly 4 fusions, all present in the CCLE catalog
- Patient dataset (922 epithelial cells) yields exactly 10 fusions, including TMPRSS2-ERG detected in 27 cells
- Contingency-table percentages reproduce Table 2: fusion positivity in 16.7% (43/257) of treated vs 3.7% of untreated cells
- Fisher's exact test returns p = 0.0047 matching Table 2
- Bulk-method comparison is consistent: bulk reports 23–113 fusions incl. TMPRSS2-ERG in 17–26 cells, scFusion reports a lower count consistent with low FDR

## Limitations

- Reduced sensitivity for fusions with low expression, as for all RNA-based methods (shown in simulations)
- Limited detection power for rare fusions in highly heterogeneous tumor samples; TRAJ20-TRAV1-2 was missed, possibly because it was rare in MAIT cells
- Limitations can only be overcome by sequencing more cells and/or sequencing each cell deeper
- scFusion is for Smart-Seq data rather than 10X; FASTQ files must be named *_1.fastq/_2.fastq (*_1.fq not allowed)

## Evidence

- [methods] joint analysis with catalogs: "Download all scripts, unzip the hg19mappability file in the data folder, and run scFusion.py."
- [intro] deep-learning filter rationale: "scFusion employs a statistical model and a deep-learning model to control for false positives."
- [methods] mappability filter: "If you do not provide this file, scFusion will turn off the mappability filter."
- [readme] input format constraint: "Single cell RNA sequencing files from Smart-Seq protocol. File names should be *_1.fastq, *_2.fastq"
- [discussion] heterogeneity limitation: "The detection power is also limited for rare fusions in highly heterogeneous tumor samples"
