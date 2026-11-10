# Release log: transcriptomics v2

| Stage | Done at | Repository commit | Digests |
|---|---|---|---|
| gate | 2026-11-09T15:04:49Z | fc4e0b59ab819d5d2c938c080824fc801c0b15d0 | `{"candidate_tree": "sha256:43fd4603c689e547ddb7d33feac72af5f92bbf9715ae95863778c7e8a6680ba9", "gate_report": "513f5e4d4c8c7008bce181da245816e08d0289e181bfa36f823b12e9bbaa8b11"}` |
| deposit-plan | 2026-11-09T15:04:52Z | fc4e0b59ab819d5d2c938c080824fc801c0b15d0 | `{"archive_sha256": "c21b9f2152188109f1f994e9301e80729b65dd0488774802ea92d7ca9e765067"}` |
| deposit-draft | 2026-11-09T21:06:01Z | fc4e0b59ab819d5d2c938c080824fc801c0b15d0 | `{"archive_sha256": "c21b9f2152188109f1f994e9301e80729b65dd0488774802ea92d7ca9e765067"}` |
| record-doi | 2026-11-10T10:42:46Z | fc4e0b59ab819d5d2c938c080824fc801c0b15d0 | `{"CITATION.cff": "2630c4109d22e8bc31e2fbc13278b7ad259f37262f6314af4dd3b1906796a4c7", "README.md": "db8a5ea1970d094a145feadab2941fe552fe6e8177f3111886de13b2820b4298", "RELEASE.yaml": "8e5c0aa91a5af5d097a074ac78a293760c62c695e4fff7cc2184c7e9d157a6bc", "marketplace.json": "65e6d9251ba8781c8f52160a34fbad6e04b7ec453f143ad5cc2594cc9b6f85a3", "plugin.json": "daa9a07c5ac801e9b44e735f474b2f006375d533ae09728279618aef0a78ac9d"}` |
| receipt | 2026-11-10T10:49:56Z | fc4e0b59ab819d5d2c938c080824fc801c0b15d0 | `{"receipt": "89b7811153dee39c9005389c9222bb397b8ae8da7e11ed6eff1124438394b0d1"}` |

## Owner actions

- OWNER: publish the reviewed Zenodo draft at https://zenodo.org/deposit/24093100; publishing mints both the version DOI and the concept DOI
- OWNER: run record-doi --doi <version DOI>, then manifest --concept-doi <concept DOI>
