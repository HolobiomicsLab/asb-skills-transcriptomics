---
name: geo-microarray-data-retrieval
description: Use when the analysis needs a real-world disease gene list (e.g. for
  enrichment testing) and GEO accession IDs are known or discoverable via MeSH-term
  queries, as with RA microarray datasets GSE12021/GSE55457/GSE55584/GSE55235.
when_to_use_negative:
- The input is already a gene list or DE table — skip retrieval and go directly to
  enrichment (process_input_text/compute_sets)
- The data of interest are single-cell expression data — E.PAGE instead accepts a
  Seurat object via process_input_seurat rather than GEO microarray retrieval
- The target data are genetic variants (SNPs, CNVs, DNA methylations) — the article
  states these data types are not yet covered
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: Gene Expression Omnibus (GEO)
  role: Source repository for the microarray datasets (GSE12021, GSE55457, GSE55584,
    GSE55235) queried to obtain RA and healthy-control expression data
- name: E.PAGE
  role: Downstream consumer of the retrieved DE gene list; gene symbols are passed
    to process_input_text (capitalize = TRUE) for enrichment computation
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: R 4.0.3
  role: Environment in which dataset retrieval outputs are formatted and the E.PAGE
    enrichment workflow is executed
- name: MSigDB C7
  role: Alternative/complementary source of differentially expressed gene names extracted
    alongside GEO
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:193c3aab7b931b1b360bd96ab1ba490b78017e21cef835e25033477693826ef7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PAGE
  tools:
  - E.PAGE
  - Gene Expression Omnibus (GEO)
  - MSigDB C7
  - R 4.0.3
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# geo-microarray-data-retrieval

## Summary

Retrieve publicly deposited microarray datasets from the Gene Expression Omnibus (GEO) and assemble a differentially expressed gene list (e.g. 229 DE genes from 45 rheumatoid arthritis samples vs 29 healthy controls across GSE12021, GSE55457, GSE55584 and GSE55235) to serve as input for downstream enrichment analysis.

## When to use

Use when the analysis needs a real-world disease gene list (e.g. for enrichment testing) and GEO accession IDs are known or discoverable via MeSH-term queries, as with RA microarray datasets GSE12021/GSE55457/GSE55584/GSE55235.

## When NOT to use

- The input is already a gene list or DE table — skip retrieval and go directly to enrichment (process_input_text/compute_sets)
- The data of interest are single-cell expression data — E.PAGE instead accepts a Seurat object via process_input_seurat rather than GEO microarray retrieval
- The target data are genetic variants (SNPs, CNVs, DNA methylations) — the article states these data types are not yet covered

## Inputs

- GEO accession IDs (e.g. GSE12021, GSE55457, GSE55584, GSE55235)
- GEO microarray raw/processed expression data and sample metadata
- Case/control sample labels (e.g. 45 RA samples vs 29 healthy controls)
- Optionally, DE gene lists from MSigDB C7 as a cross-check

## Outputs

- Consolidated differentially expressed gene list (e.g. 229 DE genes for rheumatoid arthritis)
- Sample-level metadata table mapping samples to condition labels
- Text-formatted gene list ready for E.PAGE process_input_text

## How to apply

Identify the GEO accession(s) for the condition of interest — in the E.PAGE validation this was done via MeSH-term queries of the GEO database (a limitation the authors note: the study was 'restricted to MeSH-term queries of GEO'). Download the raw microarray datasets (here GSE12021, GSE55457, GSE55584, GSE55235) and the associated sample metadata to establish case/control labels (45 RA samples vs 29 healthy controls). Perform differential expression analysis across the datasets to assemble a consolidated DE gene list (229 genes for RA). Format the gene symbols as a space- or text-delimited list and pass them to the enrichment step via E.PAGE's process_input_text with capitalize = TRUE in R 4.0.3. Verify that the retrieved sample counts and DE gene count match the published figures before proceeding to enrichment.

## Related tools

- **Gene Expression Omnibus (GEO)** (Source repository for the microarray datasets (GSE12021, GSE55457, GSE55584, GSE55235) queried to obtain RA and healthy-control expression data)
- **E.PAGE** (Downstream consumer of the retrieved DE gene list; gene symbols are passed to process_input_text (capitalize = TRUE) for enrichment computation) — https://github.com/AhmedMehdiLab/E.PAGE
- **R 4.0.3** (Environment in which dataset retrieval outputs are formatted and the E.PAGE enrichment workflow is executed)
- **MSigDB C7** (Alternative/complementary source of differentially expressed gene names extracted alongside GEO)

## Examples

```
genes <- "GENE1 GENE2 GENE3"; input <- process_input_text(genes, capitalize = TRUE); results <- compute_sets(input)
```

## Evaluation signals

- Sample counts match the published design: 45 RA samples versus 29 healthy controls
- The assembled DE gene list contains the expected 229 genes for the RA datasets GSE12021, GSE55457, GSE55584 and GSE55235
- Gene symbols are correctly capitalized for E.PAGE matching (process_input_text with capitalize = TRUE)
- Downstream enrichment reproduces published Table 3 values: 'Infection type: Acute' padj 8.49E-34 (fold enrichment 2.63) and 'Immune response' padj 1.03E-27 (fold enrichment 2.87), within rounding tolerance
- Retrieved accessions resolve and download without sample-label mismatches between datasets

## Limitations

- The study is limited to the use of MeSH terms to query the GEO database for differential gene expression data
- Dataset bias is possible due to limited input data types, such as similarities between infection and tissue damage-associated immune responses
- Microarray datasets span multiple studies; batch effects and platform differences across GSE accessions are not addressed in the described workflow

## Evidence

- [other] Retrieve the rheumatoid arthritis microarray datasets GSE12021, GSE55457, GSE55584 and GSE55235 from GEO and assemble the 229 DE genes from 45 RA samples versus 29 healthy controls.: "microarray datasets; GSE12021, GSE55457, GSE55584 and GSE55235"
- [methods] Names of Differentially expressed genes were extracted from GEO and MSigDB C7 databases.: "Names of Differentially expressed genes were extracted from GEO and MSigDB C7 databases."
- [readme] Gene symbols are normalized before enrichment via E.PAGE's text input processing with capitalization enabled.: "genes <- "ftl ApoE CTSZ"
input <- process_input_text(genes, capitalize = TRUE)"
- [discussion] The source-data discovery strategy was constrained to MeSH-term queries of GEO, limiting recall.: "Our study is limited to the use of MeSH terms to query GEO database for differential gene expression data."
- [intro] E.PAGE accepts a list of genes of interest and finds enriched environmental pathways using a curated database.: "This is a data analysis package allowing the user with a list of genes of interest to find enriched environmental pathways using a curated database."
