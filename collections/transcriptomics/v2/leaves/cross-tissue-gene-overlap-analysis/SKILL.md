---
name: cross-tissue-gene-overlap-analysis
description: Use when you have per-tissue ASE SNP call sets (e.g. VADT sig_meta_analysis_variants.txt
  files) mapped to gene IDs and need to test whether the same genes show ASE across
  multiple tissues and what pathways they share.
when_to_use_negative:
- Input is still at the ASE SNP level — SNP overlap across tissues was only 3.7% in
  this study; aggregation to the gene level is required before comparing tissues
- The variant call sets are not filtered by VADT criteria (quality score >= 20, SNP
  read count threshold of 20, Benjamini-Hochberg FDR cutoff 0.05), so the informative-SNP
  denominators for ASE scores are invalid
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: VADT (VCF ASE Detection Tool)
  role: Produces the filtered, binomial-tested, FDR-controlled per-tissue ASE SNP
    call sets whose gene mappings feed the cross-tissue overlap
  repo: https://github.com/mjtiv/VADT
- name: Ensembl Variant Effect Predictor (VEP)
  role: Maps significant ASE SNPs to Ensembl gene IDs to build the per-tissue gene
    sets
- name: STRING
  role: Builds the PPI network and KEGG pathway enrichment for the 208 shared high-ASE-score
    genes with symbols
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:c2d3bbd2507174e23e90845c885e0170da0e8d694969177995ef68f87b623b5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - Ensembl Variant Effect Predictor (VEP)
  - STRING
  - VADT (VCF ASE Detection Tool)
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# cross-tissue-gene-overlap-analysis

## Summary

Compares ASE gene sets (genes containing ASE SNPs) across three broiler chicken tissues (breast muscle, abdominal fat, liver) to quantify conserved cis-regulatory overlap, then characterizes the shared gene set via protein-protein interaction and KEGG pathway enrichment in STRING.

## When to use

Use when you have per-tissue ASE SNP call sets (e.g. VADT sig_meta_analysis_variants.txt files) mapped to gene IDs and need to test whether the same genes show ASE across multiple tissues and what pathways they share.

## When NOT to use

- Input is still at the ASE SNP level — SNP overlap across tissues was only 3.7% in this study; aggregation to the gene level is required before comparing tissues
- The variant call sets are not filtered by VADT criteria (quality score >= 20, SNP read count threshold of 20, Benjamini-Hochberg FDR cutoff 0.05), so the informative-SNP denominators for ASE scores are invalid

## Inputs

- Per-tissue VADT meta-analysis ASE SNP call sets (sig_meta_analysis_variants.txt for breast muscle, abdominal fat, liver)
- SNP-to-gene mappings from Ensembl VEP against the Gallus_gallus_5.0 build
- Informative (filtered) SNP counts per gene per tissue
- Gene symbol assignments for the >=50% global ASE score subset

## Outputs

- Three-tissue ASE gene overlap count and percentage (1715 genes, 20.1%)
- Global gene ASE scores averaged across tissues
- STRING PPI network statistics (nodes, edges, average node degree, PPI enrichment p-value)
- KEGG pathway enrichment table (string_kegg_pathway_enrichment.tsv with pathway counts and FDR-adjusted p-values)

## How to apply

Parse per-tissue significant ASE variant files from the VADT meta-analysis and map ASE SNPs to Ensembl gene IDs using Ensembl VEP, yielding gene sets (4784 breast muscle, 5709 abdominal fat, 4095 liver). Intersect the three gene sets and compute the percentage overlap — the article reports SNP-level sharing of only 3.7% (2196 SNPs) rising to 20.1% (1715 genes) at the gene level, so the gene-level aggregation is the key analytic decision point. For shared genes, compute a normalized gene ASE score as (ASE SNPs / informative SNPs) x 100 per tissue, average across the three tissues, and select genes with a global ASE score >= 50% (344 genes, 208 with assigned gene symbols). Upload the 208 gene symbols to STRING to test PPI enrichment (expected p < 1.0e-16; 176 nodes, 374 edges, average node degree 4.25) and extract KEGG pathways, expected to include ribosomes (count = 18, FDR p = 8.62e-15), metabolic pathways (count = 32, FDR p = 3.33e-07) and oxidative phosphorylation (count = 11, FDR p = 8.93e-07), all converging on translation-related biology.

## Related tools

- **VADT (VCF ASE Detection Tool)** (Produces the filtered, binomial-tested, FDR-controlled per-tissue ASE SNP call sets whose gene mappings feed the cross-tissue overlap) — https://github.com/mjtiv/VADT
- **Ensembl Variant Effect Predictor (VEP)** (Maps significant ASE SNPs to Ensembl gene IDs to build the per-tissue gene sets)
- **STRING** (Builds the PPI network and KEGG pathway enrichment for the 208 shared high-ASE-score genes with symbols)

## Examples

```
shared = set(muscle_genes) & set(fat_genes) & set(liver_genes); print(len(shared), len(shared) / len(muscle_genes | fat_genes | liver_genes))  # expect 1715 genes, 20.1%
```

## Evaluation signals

- Three-tissue gene overlap must equal 20.1% (1715 shared genes)
- Gene subset selected at global ASE score >= 50% must be 344 genes, of which 208 have assigned gene symbols
- STRING PPI enrichment p < 1.0e-16 with 176 nodes, 374 edges and average node degree 4.25
- Top KEGG pathways must match ribosomes (count = 18, FDR p = 8.62e-15), metabolic pathways (count = 32, FDR p = 3.33e-07) and oxidative phosphorylation (count = 11, FDR p = 8.93e-07)
- Gene-level overlap (20.1%) must exceed SNP-level overlap (~3.7%) as expected from the article's reported comparison

## Limitations

- No eQTL data available; association of ASE SNPs with translation improvements is speculative
- Low overlap between RNA-seq variant calls and the 600K chicken genotyping panel excluded a large number of SNPs, attributed to limitations of commercial genotyping panels
- Mapping issues with muscle tissue samples (higher percentage of STAR 'too short' reads, avg. 15.39%) were the biggest pipeline problem and may bias the breast muscle gene set

## Evidence

- [abstract] ASE gene overlap quantification: "When ASE genes (genes containing ASE SNPs) were compared between tissues, the overlap among all three tissues increased to 20.1%."
- [abstract] Translation convergence rationale: "Our results indicate that ASE genes show tissue-specific enrichment patterns, but all three tissues showed enrichment for pathways involved in translation."
- [methods] SNP-level vs gene-level contrast: "Only 2196 (3.7%) SNPs were shared among all three tissues (Fig. 4)"
- [readme] VADT produces the ASE call sets: "Program takes in a raw VCF file, filters the data and then performs various statistical analysis for detection of allele specific expression (ASE)"
- [abstract] Informative SNP filtering for ASE scores: "On average ~ 174,000 SNPs in each tissue passed our filtering criteria and were considered informative, of which ~ 24,000 (~ 14%) showed ASE."
