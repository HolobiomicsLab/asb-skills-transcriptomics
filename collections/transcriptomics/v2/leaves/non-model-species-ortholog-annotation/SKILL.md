---
name: non-model-species-ortholog-annotation
description: Use when the input gene IDs or RNA-seq reads come from a non-model eukaryotic
  species (e.g. salamander) and downstream KEGG/GO pathway analysis requires mapping
  those IDs to model-organism orthologs or KEGG ortholog (KO) IDs.
when_to_use_negative:
- The species already has a well-annotated reference genome/transcriptome and IDs
  can be mapped directly — direct annotation will outperform ortholog transfer
- The analysis requires gene-level resolution for proteins absent from the KO system
  (e.g. vitellogenin is not found in the KO system)
- Reads are from a taxonomic group with no suitable EcoOmicsDB sub-group database
  — mapping to an improper sub-group will miss key functional groups
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3170
tools:
- name: ExpressAnalyst
  role: Web platform that performs the ortholog-based annotation/mapping of non-model
    species gene lists and count data against model-organism gene sets
  repo: github.com/xia-lab/ExpressAnalystR
- name: ExpressAnalystR
  role: R package exposing the annotation functions (AnnotateGeneData, PerformDataAnnot,
    ReadListFile) for local, reproducible ortholog mapping
  repo: github.com/xia-lab/ExpressAnalystR
- name: Seq2Fun
  role: Maps raw RNA-seq reads from non-model species directly to KEGG ortholog (KO)
    databases via translated search, replacing assembly-based annotation
- name: EcoOmicsDB
  role: Ortholog database (29 NCBI-taxonomy-based taxonomic sub-group databases) backing
    all annotation modules for species without a reference transcriptome
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:54964572ca42640563b26a6483487b34fbb80e6e9fc53d12f6fb4fbeb85c4fcf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EcoOmicsDB
  - ExpressAnalyst
  - ExpressAnalystR
  - Seq2Fun
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Non-model species ortholog annotation

## Summary

Annotate non-model species gene/transcript identifiers by mapping them to orthologs in model organisms using ExpressAnalyst's integrated EcoOmicsDB ortholog database, enabling functional (KEGG, GO) analysis for eukaryotes without a reference transcriptome. This replaces de novo transcript assembly plus BLAST-based 'annotation transfer' with direct ortholog-level mapping.

## When to use

Use when the input gene IDs or RNA-seq reads come from a non-model eukaryotic species (e.g. salamander) and downstream KEGG/GO pathway analysis requires mapping those IDs to model-organism orthologs or KEGG ortholog (KO) IDs.

## When NOT to use

- The species already has a well-annotated reference genome/transcriptome and IDs can be mapped directly — direct annotation will outperform ortholog transfer
- The analysis requires gene-level resolution for proteins absent from the KO system (e.g. vitellogenin is not found in the KO system)
- Reads are from a taxonomic group with no suitable EcoOmicsDB sub-group database — mapping to an improper sub-group will miss key functional groups

## Inputs

- Gene list from a non-model eukaryotic species (e.g. salamander DEG list, 'datalist1')
- Raw RNA-seq FASTQ reads from a species without a reference transcriptome
- Count table or gene expression matrix with non-model species identifiers
- Choice of EcoOmicsDB taxonomic sub-group database (based on NCBI taxonomy)

## Outputs

- Entrez/ortholog-mapped gene identifiers
- KO-level quantification or annotated count matrix
- Mapped dataset object (qs file named after the input, e.g. 'datalist1') suitable for PerformGSEA enrichment analysis

## How to apply

Skip transcript assembly and annotation transfer entirely: ExpressAnalyst aligns individual translated reads (via Seq2Fun) to a high-resolution protein ortholog database built from ~12.8 million protein-coding genes of 687 KEGG organisms, integrated through EcoOmicsDB's 29 taxonomic sub-group databases. For a gene-list workflow, upload the non-model gene list (e.g. salamander DEGs in 'datalist1') and use the MapListIds/ReadListFile functions to convert identifiers to ortholog IDs against the appropriate taxonomic sub-group database before running over-representation analysis against KEGG and GO gene sets. For count-table workflows, use AnnotateGeneData/PerformDataAnnot with the correct organism, data type, and ID type, choosing a gene-level summary appropriate to the data (sum for RNA-seq, mean/median for microarray). Selecting the correct taxonomic sub-group database is a critical decision point — improper selection (e.g. mapping protist reads to the 'fishes' database) will miss key functional groups. Note that KO-based coverage is partial even in model organisms (76.16% of human and 61.40% of zebrafish genes carry KOs), so some genes will remain unmapped.

## Related tools

- **ExpressAnalyst** (Web platform that performs the ortholog-based annotation/mapping of non-model species gene lists and count data against model-organism gene sets) — github.com/xia-lab/ExpressAnalystR
- **ExpressAnalystR** (R package exposing the annotation functions (AnnotateGeneData, PerformDataAnnot, ReadListFile) for local, reproducible ortholog mapping) — github.com/xia-lab/ExpressAnalystR
- **Seq2Fun** (Maps raw RNA-seq reads from non-model species directly to KEGG ortholog (KO) databases via translated search, replacing assembly-based annotation)
- **EcoOmicsDB** (Ortholog database (29 NCBI-taxonomy-based taxonomic sub-group databases) backing all annotation modules for species without a reference transcriptome)

## Examples

```
dataSets <- AnnotateGeneData("E-GEOD-25713.txt", "mmu", "entrez");
```

## Evaluation signals

- Mapped ortholog IDs are searchable against model-organism KEGG/GO gene sets (e.g. salamander DEGs recover the reported 27 up- and 81 down-regulated enriched pathways under FDR < 0.05 with ≥5 DEGs)
- readDataset('datalist1') (or the dataset-named qs file) returns a dataSet object with populated annotation fields rather than an empty mapping
- Diagnostic QC plots (box plot, PCA) generated after annotation show expected sample structure for the mapped data
- Coverage check: expect a fraction of genes to remain unmapped given KO coverage limits (~76% human, ~61% zebrafish protein-coding genes carry KOs)
- SanityCheckData completes without errors when the correct organism and ID type are supplied (e.g. 'mmu', 'entrez')

## Limitations

- Reads from the same gene may map to similar but different orthologs in different species, requiring care when interpreting taxonomically diverse comparisons
- KO-based annotation has limited transcriptome coverage; biomarkers like vitellogenin are absent from the KO system
- Improper EcoOmicsDB sub-group database selection (e.g. protist reads mapped to the 'fishes' database) can miss key functional groups
- Published reference genomes of non-model organisms still contain many uncharacterized IDs, limiting downstream annotation

## Evidence

- [abstract] All modules are integrated with EcoOmicsDB, an ortholog database that enables comprehensive analysis for species without a reference transcriptome.: "All modules are integrated with EcoOmicsDB, an ortholog database that enables comprehensive analysis for species without a reference transcriptome."
- [intro] Annotation transfer via BLAST-like algorithms is the conventional approach ExpressAnalyst replaces.: "Annotation is usually performed with Basic Local Alignment Search Tool (BLAST) like algorithms through a process called “annotation transfer”"
- [intro] Typical assembly/annotation steps replaced by direct ortholog alignment.: "the typical steps of transcript assembly and annotation are replaced with the alignment of individual translated reads directly to a comprehensive, high-resolution protein ortholog database"
- [discussion] EcoOmicsDB structure is taxonomic sub-group databases.: "EcoOmicsDB currently consists of 29 taxonomic sub-group databases based on the NCBI taxonomy system"
- [intro] KO coverage limitation in model organisms.: "the human genome has 19,648 protein-coding genes, of which only 14,964 (76.16%) are annotated with KOs"
- [discussion] Improper database selection can miss functional groups.: "if the protein database is not selected properly (e.g., reads from a protist sample are mapped to the “fishes” database), key functional groups may be missed"
- [readme] README gene-list qs file naming ('datalist1').: "For gene list, the format qs file is named "datalist1". use this function to access:"
