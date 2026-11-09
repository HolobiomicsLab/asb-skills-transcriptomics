---
name: snatac-seq-cell-type-activity-matrix-handling
description: Use when a pooled snATAC-seq candidate enhancer BED (e.g. 286,777 heart
  enhancers) has per-cell-type activity columns and you need cell-type-resolved enhancer–gene
  links with a generalized ABC score.
when_to_use_negative:
- When the goal is TF affinity computation and per-gene TF score summarisation rather
  than enhancer–gene ABC-scored interactions (use STARE's TF-affinity part instead
  of the separate ABC-scoring mode).
- When enhancer activity comes from bulk assays rather than per-cell-type snATAC-seq
  accessibility columns — the cell-type-resolved setup with one -n activity column
  per cell type would not apply.
- When genome build or Hi-C reference differs from the Fulco et al. hg19 average matrix
  and its matching enhancer coordinates — coordinate system must be consistent across
  BED, GTF and contact files.
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: STARE
  role: Framework containing the generalized/adapted Activity-By-Contact scoring module
    used to score pooled enhancer candidates against genes per cell type
  repo: https://github.com/schulzlab/STARE
- name: stare-abc
  role: Bioconda package providing STARE_ABCpp for separate ABC-scoring with adapted
    activity and all_tss mode
  repo: https://github.com/SchulzLab/STARE
- name: Juicer
  role: Provides data extraction used by STARE's Juicebox_normalization.sh to derive
    normalized contact files from the average hg19 Hi-C matrix
  repo: https://github.com/aidenlab/juicer
- name: bedtools
  role: Genome arithmetic companion used for interval operations in the STARE workflow
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:8b5ebdfc6db94f262c88e4155276b23eeac653cc82000cfdee11e5d83811f4ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - Juicer
  - STARE
  - bedtools
  - stare-abc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# snatac-seq-cell-type-activity-matrix-handling

## Summary

Run cell-type-specific enhancer–gene interaction scoring on a pooled single-nucleus ATAC-seq candidate enhancer set by supplying each cell type's accessibility column as the enhancer activity input to the generalized (adapted) ABC model in STARE, then quantify how many gABC-scored interactions arise per cell type and how much they overlap across cell types.

## When to use

Use when a pooled snATAC-seq candidate enhancer BED (e.g. 286,777 heart enhancers) has per-cell-type activity columns and you need cell-type-resolved enhancer–gene links with a generalized ABC score.

## When NOT to use

- When the goal is TF affinity computation and per-gene TF score summarisation rather than enhancer–gene ABC-scored interactions (use STARE's TF-affinity part instead of the separate ABC-scoring mode).
- When enhancer activity comes from bulk assays rather than per-cell-type snATAC-seq accessibility columns — the cell-type-resolved setup with one -n activity column per cell type would not apply.
- When genome build or Hi-C reference differs from the Fulco et al. hg19 average matrix and its matching enhancer coordinates — coordinate system must be consistent across BED, GTF and contact files.

## Inputs

- Pooled candidate enhancer BED file with one activity column per cell type (e.g. 286,777 Hocker 2021 human heart snATAC-seq candidate enhancers)
- Normalized Hi-C contact files in bin|bin|contact format, per chromosome, gzipped, at 5000 bp bin size (from the Fulco et al. 2019 average hg19 Hi-C matrix, KR-normalized)
- Full gene annotation in GTF format (Gencode)
- Cell-type identity/activity matrix from the snATAC-seq dataset defining which accessibility column corresponds to each of the eight heart cell types

## Outputs

- Per-cell-type gABC-scored interaction files (*_ABCpp_scoredInteractions_*.txt.gz) with interactions passing the 0.02 cut-off
- Per-cell-type counts of gABC-scored interactions (expect mean ~408,846, SD ~12,200)
- Fraction of each cell type's interactions shared with all other cell types (expect ~23.6%)
- Pairwise cell-type intersection counts, including the aCM–vCM intersection (expect 39,707 interactions)

## How to apply

First convert the Fulco et al. (2019) average hg19 Hi-C matrix (average_hic.v2.191020.tar.gz) into per-chromosome gzipped bin|bin|contact files at 5000 bp resolution using STARE's Juicebox_normalization.sh script with KR normalization. For each cell type, run STARE in separate ABC-scoring mode (STARE_ABCpp) with the pooled enhancer BED via -b, that cell type's activity column via -n, full Gencode GTF via -a, window size 5,000,000 bp (-w), contact folder (-f), bin size 5000 (-k), all_tss mode (-i, using all annotated transcription start sites), adapted activity enabled (-q True, which rescales enhancer activity by the number of contacts to candidate target genes), and score cut-off 0.02 (-t 0.02). Collect the *_ABCpp_scoredInteractions_*.txt.gz outputs per cell type, count interactions, and compute the fraction of each cell type's interactions shared with all other cell types plus pairwise intersections (notably aCM–vCM). The rationale is that the adapted activity adjustment and all-TSS mode improve accuracy on single-cell data where activity is estimated per cell type rather than from bulk assays.

## Related tools

- **STARE** (Framework containing the generalized/adapted Activity-By-Contact scoring module used to score pooled enhancer candidates against genes per cell type) — https://github.com/schulzlab/STARE
- **stare-abc** (Bioconda package providing STARE_ABCpp for separate ABC-scoring with adapted activity and all_tss mode) — https://github.com/SchulzLab/STARE
- **Juicer** (Provides data extraction used by STARE's Juicebox_normalization.sh to derive normalized contact files from the average hg19 Hi-C matrix) — https://github.com/aidenlab/juicer
- **bedtools** (Genome arithmetic companion used for interval operations in the STARE workflow) — https://github.com/arq5x/bedtools2

## Examples

```
STARE_ABCpp -b heart_enhancers.bed -n <celltype>_activity.tsv -a gencode.gtf -w 5000000 -f contacts_hg19_kr/ -k 5000 -i all_tss -q True -t 0.02
```

## Evaluation signals

- Mean number of gABC-scored interactions across the eight heart cell types should be ~408,846 (SD ~12,200) at cut-off 0.02.
- Mean fraction of a cell type's interactions shared with all other cell types should be ~23.6%.
- The aCM–vCM two-cell-type intersection should equal 39,707 interactions.
- Each output *_ABCpp_scoredInteractions_*.txt.gz file must contain only interactions surpassing the 0.02 cut-off for the given activity column.
- Contact files must be KR-normalized, gzipped, per chromosome, with 5000 bp bins matching the -k 5000 setting and the hg19 Fulco et al. matrix.

## Limitations

- The activity input is enhancer accessibility per cell type from snATAC-seq; accuracy depends on the adapted-activity rescaling and on using all annotated TSSs (-i all_tss) rather than a single canonical TSS.
- Results are tied to the Fulco et al. (2019) average hg19 Hi-C matrix and 5000 bp bins; a different contact source (e.g. the Gschwind et al. 2023 averaged hg38 matrix, ENCFF134PUN) requires matched hg38 coordinates.
- Window size (-w 5,000,000 bp) and the 0.02 cut-off are configurable thresholds; changing them changes the number of predicted interactions and expected validation values.

## Evidence

- [methods] If you're only interested in the ABC-scored interactions, you can also call that part independently.: "If you're only interested in the ABC-scored interactions, you can also call that part independently."
- [abstract] The ABC score can yield higher accuracy by adapting enhancer activity according to the number of contacts the enhancer has to its candidate target genes and by considering all annotated transcription start sites of a gene: [evidence span withheld — anchor status: not_found]
- [methods] ./Code/Juicebox_normalization.sh -j <path_juicer_jar_file> -h <hic_path> -n KR -d <out_path> -c <chromosomes> -b <bin_size>: [evidence span withheld — anchor status: not_found]
- [methods] Size of the window around your candidate regions in which genes are considered for the adapted activity adjustment: [evidence span withheld — anchor status: not_found]
- [readme] STARE is a framework that combines two tasks, which can also work independently: "STARE is a framework that combines two tasks, which can also work independently"
- [abstract] We combined our generalized ABC model with TF binding information and illustrated an analysis of a single-cell ATAC-seq dataset of the human heart: "We combined our generalized ABC model with TF binding information and illustrated an analysis of a single-cell ATAC-seq dataset of the human heart"
