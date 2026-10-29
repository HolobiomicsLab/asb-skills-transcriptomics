---
name: topic-based-gene-set-selection
description: Use when an interpretable scRNA-seq topic model (MIRA/CODAL) has been
  fitted and you need a gene set representing one topic's biological program, plus
  a background, to test motif or pathway enrichment among topic-ranked genes.
when_to_use_negative:
- The gene set is already defined by an external source (e.g. curated marker list
  or differential expression table) — ranking by topic contribution would duplicate
  or distort it
- No interpretable topic model has been fitted, or topic loadings are confounded by
  batch effects because technical correction (CODAL) was not applied
- The question requires genome-wide genes rather than the HVG-filtered panel (genes
  failing the 30-count filter are absent from the model and cannot be ranked)
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: MIRA
  role: Variational topic modeling of single-cell RNA-seq; produces the topic-gene
    contribution weights from which the gene set is ranked
  repo: https://github.com/cistrome/MIRA
- name: CODAL
  role: Batch-correction augmentation of the MIRA topic model so topic loadings reflect
    biological rather than technical effects
  repo: https://mira-multiome.readthedocs.io
- name: scanpy
  role: Preprocessing (count filtering, normalize_total with target_sum=10000, log+1,
    HVG selection at minimum dispersion 0.3) yielding the 2841-gene panel the topic
    model is trained on
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:66e97727ad19c8719fd7f653481466e9adb9cd50c97d8e406c8ca72d261b8452
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cistrome/MIRA
  tools:
  - CODAL
  - MIRA
  - scanpy
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# topic-based-gene-set-selection

## Summary

Extracts a biologically coherent gene set by ranking genes on their contribution to a single interpretable topic in a CODAL/MIRA expression topic model (e.g. the top 200 genes of topic 4, upregulated in proerythroblast cells), retaining the remaining highly variable genes as a matched background set for downstream enrichment testing.

## When to use

Use when an interpretable scRNA-seq topic model (MIRA/CODAL) has been fitted and you need a gene set representing one topic's biological program, plus a background, to test motif or pathway enrichment among topic-ranked genes.

## When NOT to use

- The gene set is already defined by an external source (e.g. curated marker list or differential expression table) — ranking by topic contribution would duplicate or distort it
- No interpretable topic model has been fitted, or topic loadings are confounded by batch effects because technical correction (CODAL) was not applied
- The question requires genome-wide genes rather than the HVG-filtered panel (genes failing the 30-count filter are absent from the model and cannot be ranked)

## Inputs

- Trained MIRA/CODAL expression topic model (e.g. on the NEURIPS bone marrow multimodal dataset)
- Highly variable gene annotation (2841 genes with UCSC TSS annotations)
- Topic index of interest (e.g. topic 4, proerythroblast-upregulated) and gene-by-topic contribution weights

## Outputs

- Foreground gene set: top 200 genes ranked by topic-4 contribution
- Background gene set: remaining 2641 highly variable genes
- Gene sets consumed downstream by probabilistic in silico deletion motif enrichment and Wilcoxon testing (e.g. fig4d_motif_enrichment_table.csv)

## How to apply

After training the CODAL-augmented MIRA topic model on the NEURIPS bone marrow multimodal dataset (2841 highly variable genes with UCSC TSS annotations), rank genes by their contribution to the topic of interest — here topic 4, the proerythroblast-associated topic — and take the top 200 as the foreground gene set. Use the remaining 2641 highly variable genes as the background set so enrichment tests (e.g. Wilcoxon on in-silico-deletion motif association scores) compare topic genes against equally filtered genes. Because CODAL disentangles technical from biological effects via mutual information regularization, topic gene loadings reflect biology rather than batch, which is the rationale for deriving gene sets from the corrected model. Validate that the selected topic corresponds to the intended cell state (e.g. proerythroblast upregulation) before treating its genes as the target program.

## Related tools

- **MIRA** (Variational topic modeling of single-cell RNA-seq; produces the topic-gene contribution weights from which the gene set is ranked) — https://github.com/cistrome/MIRA
- **CODAL** (Batch-correction augmentation of the MIRA topic model so topic loadings reflect biological rather than technical effects) — https://mira-multiome.readthedocs.io
- **scanpy** (Preprocessing (count filtering, normalize_total with target_sum=10000, log+1, HVG selection at minimum dispersion 0.3) yielding the 2841-gene panel the topic model is trained on)

## Examples

```
topic4 = model.decoder.weight[4].argtopn(200); foreground = adata.var_names[topic4]; background = adata.var_names[list(set(range(2841)) - set(topic4))]
```

## Evaluation signals

- Foreground size equals the chosen cutoff (200 genes) and background equals the remaining 2641 highly variable genes; the two sets partition the model's gene panel
- The selected topic's top genes are upregulated in the expected cell state (proerythroblast) when inspected against cell-type annotations
- Downstream Wilcoxon enrichment of known regulator motifs (TAL1-GATA1, GATA family, TAL1-TCF3) against this background reproduces significant, Bonferroni-corrected enrichments, with far stronger significance when the topic model was CODAL-corrected (3 orders of magnitude)
- All ranked genes carry UCSC TSS annotations, since the model was restricted to the 2841 HVGs that do

## Limitations

- Gene sets are restricted to highly variable genes passing the filters (cells < 400 counts and genes < 30 counts filtered; dispersion threshold 0.3), so lowly expressed regulators outside the panel are invisible to the topic ranking
- Topic identity is dataset- and run-specific; the proerythroblast program is topic 4 in this analysis and must be re-identified in new datasets
- Authors caution that batch-confounded or perturbation-induced states should be validated with orthogonal biological evidence to rule out under-correction of technical effects

## Evidence

- [other] Extract the top 200 genes ranked by topic-4 contribution (the Proerythroblast-upregulated set) from the CODAL expression topic model of the NEURIPS bone marrow dataset, using the remaining 2641 highly variable genes as the background set.: [evidence span withheld — anchor status: not_found]
- [methods] HVG filtering threshold used to define the gene universe for topic ranking.: "highly variable genes found with a minimum dispersion threshold of 0.3, yielding 3500 highly variable genes"
- [methods] Genes must carry TSS annotations to enter the model and thus be rankable by topic contribution.: "2841 genes which were both highly variable and had UCSC TSS annotations78"
- [abstract] Rationale for deriving gene sets from the CODAL-corrected topic model.: "uses a mutual information regularization technique to explicitly disentangle factors related to technical and biological effects"
- [readme] Downstream check that the gene set supports biologically meaningful motif enrichment.: "MIRA includes methods for: * Multimodal topic modeling"
