---
name: go-enrichment-analysis-splicing-genes
description: Use when fortuna alt.tsv (or any novel-event table keyed by gene_id)
  has produced a list of genes with novel splicing events and you need functional
  interpretation or enrichment testing.
when_to_use_negative:
- The event table has no gene-level assignment (gene_id) — enrichment requires mapping
  events back to genes via the preprocessed GTF
- Events have not been filtered by support depth (e.g. RPM >= 1), since low-support
  events dominate the count and bias the gene set
- You want to test differential exon usage (DEXSeq) rather than characterize the functional
  identity of genes with novel events
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: fortuna
  role: Produces alt.tsv novel-event annotations that carry the gene (gene_id) and
    transcript_id fields used to build the gene set for enrichment
  repo: https://github.com/canzarlab/fortuna
- name: DEXSeq
  role: Reference unit-of-analysis for aggregating signature counts to larger splicing
    units at gene level (E1), complementing the gene-set interpretation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:e4da6b00eb7a33788bd0680369a34cc7e8bb300f544c1b3895794029fedeedd6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/fortuna
  tools:
  - DEXSeq
  - fortuna
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# go-enrichment-analysis-splicing-genes

## Summary

Runs gene ontology enrichment on the set of genes harboring novel splicing events detected by fortuna, and tabulates events in functionally targeted genes (e.g. Drosophila sarcomere genes) to interpret whether unannotated splicing is enriched in biological functions.

## When to use

Use when fortuna alt.tsv (or any novel-event table keyed by gene_id) has produced a list of genes with novel splicing events and you need functional interpretation or enrichment testing.

## When NOT to use

- The event table has no gene-level assignment (gene_id) — enrichment requires mapping events back to genes via the preprocessed GTF
- Events have not been filtered by support depth (e.g. RPM >= 1), since low-support events dominate the count and bias the gene set
- You want to test differential exon usage (DEXSeq) rather than characterize the functional identity of genes with novel events

## Inputs

- alt.tsv from fortuna --quant/--pairs (novel events with chr coordinates, gene_id, transcript_id, event type, read support)
- preprocessed GTF (sample.rg.gtf) with gene_id fields for gene-set construction
- per-tissue event table classified as shared vs tissue-specific at RPM >= 1

## Outputs

- GO enrichment results for genes harboring novel splicing events
- per-gene tabulation of novel events (e.g. sarcomere genes: Mhc, bt, Unc-89, up, Tm1, wupA, Strn-Mlck, sls)

## How to apply

After fortuna --quant/--pairs steps produce per-tissue alt.tsv files (which record chromosome coordinates, gene gene_id, event type such as ES/IR/AA/AD/AP/IE/XX, and supporting read counts, with IDs taken from the preprocessed sample.rg.gtf), extract the set of unique genes harboring novel events. Apply the RPM >= 1 support-depth filter used in the event-classification step before defining the gene set, then run GO enrichment on that gene list against the organism background. Tabulate the number of novel events per gene of interest — in the Drosophila pupal-tissue study these were sarcomere genes (Mhc, bt, Unc-89, up, Tm1, wupA, Strn-Mlck, sls) — and check the result against expected baselines (>10 novel events in sarcomere genes in the reported analysis).

## Related tools

- **fortuna** (Produces alt.tsv novel-event annotations that carry the gene (gene_id) and transcript_id fields used to build the gene set for enrichment) — https://github.com/canzarlab/fortuna
- **DEXSeq** (Reference unit-of-analysis for aggregating signature counts to larger splicing units at gene level (E1), complementing the gene-set interpretation)

## Evaluation signals

- Every gene in the enrichment input must match a gene_id from the preprocessed sample.rg.gtf (not an invented identifier)
- Novel-event counts per target gene should exceed the study baseline (>10 novel events in sarcomere genes in the Drosophila analysis)
- The gene set derives from events passing the RPM >= 1 support-depth filter used for the shared/tissue-specific classification
- Reproducing Fig. 6 / Supplementary Table S10 and Fig. S8-S9 distributions (~2% of events supported by >=150 reads, ~7.5% shared, 13%/15%/48% IFM/leg/brain-specific) should cohere with the enriched gene list

## Limitations

- Novel events are guessed from annotated splice sites only; genes without detectable novel events are excluded from the gene set regardless of their biology
- Over 60% of novel events are supported by <=5 reads, so support-depth filtering changes which genes enter the enrichment
- The tabulated sarcomere gene list (Mhc, bt, Unc-89, up, Tm1, wupA, Strn-Mlck, sls) is Drosophila/tissue-specific and must be replaced for other organisms

## Evidence

- [other] GO enrichment step in the Drosophila task workflow: [evidence span withheld — anchor status: not_found]
- [readme] gene_id provenance in alt.tsv: "Just like subexon IDs in the cnt.tsv file, chromosome, gene (gene_id field), and transcript IDs (transcript_id field) are taken from the input GTF file."
- [readme] novel event annotation capability: "In addition, fortuna annotates all novel events (E2) based on precise definitions of event types."
- [readme] novel events map to genes with support counts: "which means that on chromosome 2 there exists a novel intron between coordinates 190984393 and 190986854 on gene Locus_18066"
- [abstract] Drosophila tissue-specific finding: "We further used fortuna to identify novel, tissue-specific splicing events in Drosophila."
