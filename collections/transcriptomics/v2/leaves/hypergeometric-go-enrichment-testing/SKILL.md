---
name: hypergeometric-go-enrichment-testing
description: Use when you have a gene list of interest derived from a filtering step
  (e.g., 533 cell-type-specific IPA events mapped to their host genes) and a defined
  background universe of genes, and you need GO term enrichment significance.
when_to_use_negative:
- When analyzing differential IPA usage dynamics (e.g., control vs LPS-stimulated
  monocytes) rather than gene-set over-representation — the article uses DRIMSeq differential
  transcript usage for that question instead
- When no defined background universe exists or the gene list is not a subset of the
  background (hypergeometric assumptions violated)
- When the input is raw scRNA-seq reads or BAM alignments — gene-level enrichment
  testing requires a processed, filtered gene list first
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: InPACT
  role: Upstream producer of the novel IPA event and cell-type-specific event tables
    whose host genes form the enrichment foreground set
  repo: https://github.com/YY-TMU/InPACT
- name: DRIMSeq
  role: Complementary differential transcript usage analysis on IPA isoforms; GO enrichment
    follows differential/usage-based gene selection in the paper's workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:0526b2a3887c083a7172da4bcd505a7f24e147d018ce257777a2ac80286ca25e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - DRIMSeq
  - InPACT
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# hypergeometric-go-enrichment-testing

## Summary

Statistical skill for testing whether a defined subset of genes (e.g., genes harboring cell-type-specific IPA events identified by InPACT in the 486-cell human fetal bone marrow scRNA-seq dataset) is significantly over-represented among Gene Ontology terms, using the hypergeometric (over-representation) test.

## When to use

Use when you have a gene list of interest derived from a filtering step (e.g., 533 cell-type-specific IPA events mapped to their host genes) and a defined background universe of genes, and you need GO term enrichment significance.

## When NOT to use

- When analyzing differential IPA usage dynamics (e.g., control vs LPS-stimulated monocytes) rather than gene-set over-representation — the article uses DRIMSeq differential transcript usage for that question instead
- When no defined background universe exists or the gene list is not a subset of the background (hypergeometric assumptions violated)
- When the input is raw scRNA-seq reads or BAM alignments — gene-level enrichment testing requires a processed, filtered gene list first

## Inputs

- Gene list of interest, e.g., host genes of cell-type-specific IPA events (533 events from the fetal bone marrow analysis)
- Background gene universe, e.g., all genes passing the TPM >= 1 abundance filter in the IPA usage table
- GO term-to-gene association annotation (GO ontology annotation file)
- Optional: IPA usage table (TSV) output by InPACT_quantify

## Outputs

- TSV table of GO enrichment terms with hypergeometric p-values and enriched gene counts for genes harboring cell-type-specific IPA events
- Foreground/background gene set summaries per GO term

## How to apply

After IPA event identification and quantification, filter low-abundance genes (TPM < 1) and designate cell-type-specific IPA events by comparing IPA usage between cells of the same cell type and those of other cell types. Map the retained cell-type-specific IPA events to their host gene symbols to form the foreground gene set. Define the background universe as all genes with valid IPA usage measurements in the analysis. For each GO term, count its genes in the foreground and background and compute a hypergeometric tail probability P(X >= k) for observing at least k foreground genes from that term; the resulting p-values indicate which biological processes are significantly enriched among genes harboring cell-type-specific IPA events. Validate by ensuring the output is a valid TSV of GO enrichment terms and that foreground/background sets are disjoint and correctly sized (background ⊇ foreground).

## Related tools

- **InPACT** (Upstream producer of the novel IPA event and cell-type-specific event tables whose host genes form the enrichment foreground set) — https://github.com/YY-TMU/InPACT
- **DRIMSeq** (Complementary differential transcript usage analysis on IPA isoforms; GO enrichment follows differential/usage-based gene selection in the paper's workflow)

## Examples

```
from scipy.stats import hypergeom; p = hypergeom.sf(k - 1, M, n, N)  # M=background genes, n=GO term size, N=foreground size, k=overlap
```

## Evaluation signals

- Output GO enrichment table is a valid TSV and lists enriched GO terms with p-values for genes harboring cell-type-specific IPA events
- Foreground gene set is a subset of the background universe (all genes evaluated in the IPA usage comparison)
- Enrichment foreground size matches the reported cell-type-specific IPA gene count consistent with 533 events in the fetal bone marrow analysis
- No genes with TPM < 1 contribute to the foreground or background sets
- Enriched terms have hypergeometric p-values within (0, 1], and the most enriched terms are reproducible across permutations of the event designation step

## Limitations

- The hypergeometric test treats genes as independent and does not account for GO term redundancy or gene-set correlation; multiple-testing correction must be applied explicitly
- Results depend on the preceding TPM >= 1 abundance filter and the cell-type-specific IPA designation; enrichment conclusions are only valid for the same filtered universe
- The paper reports the test for genes harboring cell-type-specific IPA events in a single 486-cell dataset (12 cell types); power is limited for rare cell types

## Evidence

- [methods] Hypergeometric GO enrichment on IPA event genes: [evidence span withheld — anchor status: not_found]
- [methods] Abundance filter preceding enrichment: "low-abundance genes with TPM < 1 were filtered out"
- [methods] Differential analysis preceding GO enrichment in related workflow: "we applied DRIMSeq to conduct differential transcript usage analysis between control and LPS stimulated group"
- [results] Cell-type-specific IPA events identified by usage comparison: [evidence span withheld — anchor status: not_found]
