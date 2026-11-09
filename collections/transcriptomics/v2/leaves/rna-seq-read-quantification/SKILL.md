---
name: rna-seq-read-quantification
description: Use when raw FASTQ RNA-seq reads from a model or non-model eukaryote
  must be converted to a KO-level count table, especially when no reference transcriptome
  exists or when cross-version equivalence with Kallisto-based quantification must
  be checked.
when_to_use_negative:
- Input is already a count matrix or gene expression table — use ExpressAnalyst's
  statistical/enrichment modules directly instead of re-quantifying reads
- The biomarker of interest is absent from the KO system (e.g., vitellogenin, an egg
  yolk protein precursor, is not in KO) — KO-based quantification will not detect
  it
- The taxonomic sub-group database is mismatched to the sample (e.g., mapping protist
  reads to the 'Fishes' database) — key functional groups may be missed
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Seq2Fun 2.0
  role: 'Primary quantifier: maps individual translated reads to the KO ortholog database
    (v2.0.2, mismatches=2, min matching length=19, min BLOSUM62=80) with ~2M reads/min
    at 0.94 GB peak memory'
- name: Kallisto
  role: Reference-transcriptome-based quantification baseline (version 0.46.1, minimum
    quality score = 25) used to verify Seq2Fun equivalence
- name: ExpressAnalystR
  role: Downstream R package (synchronized with the ExpressAnalyst web server) for
    normalization (PerformExpressNormalization), QC PCA (PlotDataPCA/PlotMetaPCA),
    and limma DE on the produced count tables
  repo: github.com/xia-lab/ExpressAnalystR
- name: Docker
  role: Dockerized version of the Seq2Fun module for running quantification on personal
    computers
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:454230805f1dcec55244436b61f85c107a1c1746a7787506e7e05f56b5c17e5a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - ExpressAnalyst
  - ExpressAnalystR
  - Kallisto
  - Seq2Fun
  - Seq2Fun 1.0
  - Seq2Fun 2.0
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# rna-seq-read-quantification

## Summary

Quantify RNA-seq expression from raw FASTQ reads by mapping individual translated reads directly to a protein ortholog (KO) database with Seq2Fun 2.0 (as implemented in ExpressAnalyst), bypassing de novo transcript assembly and annotation-transfer; also covers reference-transcriptome quantification with Kallisto as a comparison baseline.

## When to use

Use when raw FASTQ RNA-seq reads from a model or non-model eukaryote must be converted to a KO-level count table, especially when no reference transcriptome exists or when cross-version equivalence with Kallisto-based quantification must be checked.

## When NOT to use

- Input is already a count matrix or gene expression table — use ExpressAnalyst's statistical/enrichment modules directly instead of re-quantifying reads
- The biomarker of interest is absent from the KO system (e.g., vitellogenin, an egg yolk protein precursor, is not in KO) — KO-based quantification will not detect it
- The taxonomic sub-group database is mismatched to the sample (e.g., mapping protist reads to the 'Fishes' database) — key functional groups may be missed

## Inputs

- Raw RNA-seq FASTQ files (e.g., NCBI SRA accessions SRR13332314–SRR13332325 from study SRP299836)
- Appropriate EcoOmicsDB taxonomic sub-group KO database (e.g., 'Fishes'; 29 taxonomic sub-groups)
- Reference transcriptome/genome FASTA when using Kallisto baseline (e.g., GRCz11 GCA_000002035.4, GMGI_Hamer_2.0 GCA_018991925.1)

## Outputs

- KO-level count table per sample ready for normalization and downstream DE/enrichment analysis
- Per-run runtime and peak-memory logs for throughput/memory benchmarking
- Normalized count tables and QC PCA plots (PlotDataPCA/PlotMetaPCA) for method comparison

## How to apply

For Seq2Fun quantification, run Seq2Fun 2.0.2 on each FASTQ against the appropriate EcoOmicsDB taxonomic sub-group KO database with 'maximum number of mismatches' = 2, 'minimum matching length' = 19, and 'minimum matching BLOSUM62 score' = 80; internally the tool performs read QC and paired-end joining, translates each cleaned read in all reading frames, filters peptides by length (longest retained, ties by BLOSUM62 score), then seeds a 7-amino-acid match and extends backwards from C to N terminal. For reference-based quantification of organisms with assemblies (e.g., zebrafish GRCz11 GCA_000002035.4 or lobster GMGI_Hamer_2.0 GCA_018991925.1), run Kallisto 0.46.1 with the minimum quality score parameter set to 25 as a comparison baseline. The benchmark shows Seq2Fun 2.0 sustains ~2 million reads per minute at 0.94 GB peak memory (down from 1.49 GB in v1.0). To validate equivalence, normalize both count tables, compare QC PCA variance structures, and run limma DE with FDR < 0.05 and |log2FC| > 1.5 — Seq2Fun reproduced almost identical PCA structures and relative DEG counts for lobster and zebrafish.

## Related tools

- **Seq2Fun 2.0** (Primary quantifier: maps individual translated reads to the KO ortholog database (v2.0.2, mismatches=2, min matching length=19, min BLOSUM62=80) with ~2M reads/min at 0.94 GB peak memory)
- **Kallisto** (Reference-transcriptome-based quantification baseline (version 0.46.1, minimum quality score = 25) used to verify Seq2Fun equivalence)
- **ExpressAnalystR** (Downstream R package (synchronized with the ExpressAnalyst web server) for normalization (PerformExpressNormalization), QC PCA (PlotDataPCA/PlotMetaPCA), and limma DE on the produced count tables) — github.com/xia-lab/ExpressAnalystR
- **Docker** (Dockerized version of the Seq2Fun module for running quantification on personal computers)

## Examples

```
Rscript -e 'devtools::install_github("xia-lab/ExpressAnalystR", build = TRUE, build_opts = c("--no-resave-data", "--no-manual"))'
```

## Evaluation signals

- For equivalence workflows, PCA variance structures from Seq2Fun and Kallisto count tables are almost identical (compare against Supplementary Figs. 2-3, Tables 1-2)
- Relative numbers of DEGs (FDR < 0.05, |log2FC| > 1.5) per treatment contrast agree between Seq2Fun and Kallisto quantification
- Throughput benchmark: ~2 million reads per minute, peak memory near 0.94 GB for v2.0 vs 1.49 GB for v1.0 across the 12 zebrafish samples
- Complete per-sample time/peak-memory table produced for all 12 samples (SRR13332314–SRR13332325) under both Seq2Fun versions
- PCA/box-plot QC diagnostics (qc_pca_*.png, qc_boxplot_*.png) show expected sample separation by metadata of interest before DE

## Limitations

- Reads from the same gene may map to similar but different orthologs across species, requiring care when interpreting taxonomically diverse comparisons
- KO coverage is incomplete (61.40% of zebrafish and 76.16% of human protein-coding genes have KOs), and genes like vitellogenin are absent from the KO system
- Adding more than 16 threads may not improve performance due to an I/O constraint; a high-speed drive could relieve it
- Improper sub-group database selection (e.g., protist reads mapped to the 'Fishes' database) can miss key functional groups; the 'Fishes' database may need zebrafish sequences removed for cross-species mapping tests

## Evidence

- [intro] We previously developed a command-line algorithm, Seq2Fun for mapping RNA-seq reads from eukaryotic species to KEGG ortholog (KO) databases with a translated search: "We previously developed a command-line algorithm, Seq2Fun for mapping RNA-seq reads from eukaryotic species to KEGG ortholog (KO) databases with a translated search"
- [discussion] Seq2Fun parameters: "“maximum number of mismatches” = 2, “minimum matching length” = 19, and “minimum matching BLOSUM62 score” = 80."
- [discussion] Seq2Fun algorithm steps: "The first step in the Seq2Fun algorithm is to conduct reads quality control and join paired-end reads."
- [discussion] Kallisto quality parameter: "The minimum quality score parameter was set to 25."
- [results] throughput/memory: [evidence span withheld — anchor status: not_found]
- [readme] README normalization/QC functions: "PlotDataBox("estrogen.txt", "qc_boxplot_", 72, "png");
PlotDataPCA("estrogen.txt", "qc_pca_", 72, "png");"
