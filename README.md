# SynthAVE

> A synthetic multilingual benchmark for attribute-value verification in e-commerce, validated with a diverse LLM arena.

[![Paper](https://img.shields.io/badge/EMNLP-Industry%20Track%202026-b31b1b)](#citation)
[![arXiv](https://img.shields.io/badge/arXiv-2607.07469-b31b1b)](https://arxiv.org/abs/2607.07469)
[![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey)](#license)
[![Languages](https://img.shields.io/badge/languages-DE%20%7C%20EN%20%7C%20ES%20%7C%20FR%20%7C%20IT-blue)](#languages)
[![Rows](https://img.shields.io/badge/rows-15%2C906-informational)](#dataset-statistics)

SynthAVE is a benchmark of 15,906 synthetic e-commerce products across five languages, released with per-row labels for the task of attribute-value verification. Each product is paired with an `(attribute, value)` and a `label` in `{Correct, Incorrect, Unknown}`. Labels are produced by a 21-judge LLM arena (7 model families × 3 prompts). The core 12,726-row benchmark across DE / ES / FR / IT follows the paper's full protocol — arena majority plus expert adjudication on the disagreement stratum, with a stratified audit on the agreement stratum. An additional 3,180 English rows are released with arena-majority labels only (see [note](#english-note)*).

> **Release note.** This dataset and its accompanying code are released solely for academic and scientific reproducibility purposes, in support of the methods and findings described in the associated publication. Pull requests are not accepted, in order to maintain the artifact exactly as it was used in the paper. If you want to build on this work as an ongoing project, please contact the authors and start a new release.

## Table of Contents

- [Dataset Description](#dataset-description)
- [Languages](#languages)
- [Dataset Structure](#dataset-structure)
- [Dataset Statistics](#dataset-statistics)
- [Label Provenance](#label-provenance)
- [How to Use](#how-to-use)
- [Dataset Creation](#dataset-creation)
- [Considerations for Using the Data](#considerations-for-using-the-data)
- [Citation](#citation)
- [License](#license)
- [Contact](#contact)

## Dataset Description

**Task.** Attribute-value verification: given a product (title, bullet points, description) and a candidate `(attribute, value)` pair, decide whether the value is `Correct`, `Incorrect`, or `Unknown` (not determinable from the product text).

**Origin.** Products are generated with the attribute-aware controlled-generation pipeline of Negri et al. 2025. Attribute values are deliberately manipulated so that all three label types occur in usable proportions. Brand names, model identifiers, and other potentially identifying signals are anonymized during generation.

**Validation.** Every `(product, attribute, value)` triple is judged by 21 LLM configurations (7 model families × 3 prompt versions, 267,246 judgments in total). Final labels are:
- **Expert adjudication** when the arena majority disagrees with the synthetic-pipeline label, and on additional adjudication rounds for DE and IT.
- **Arena majority** when the arena and synthetic pipeline agree, validated on a 400-sample stratified human audit (Cohen's κ = 0.92 vs. expert; 3.0% overturn rate).

Estimated label accuracy on the released set is 97.9%.

## Languages

Spanish (`es`), French (`fr`), Italian (`it`), German (`de`), English (`en`*).

## Dataset Structure

Files are organised per language:

```
SynthAVE/
├── german/dataset.json        # DE — 3,059 records
├── italian/dataset.json       # IT — 3,007 records
├── french/dataset.json        # FR — 3,501 records
├── spanish/dataset.json       # ES — 3,159 records
├── english/dataset.json       # EN — 3,180 records*
└── release_summary.json       # aggregate counts and label provenance
```

Each `dataset.json` is a JSON array of records. Example (Italian):

```json
{
  "id": "it_0",
  "language": "it",
  "category": "NOTEBOOK_COMPUTER",
  "title": "NexusPro Elite 15 R75800H 16GB/750GB W11H",
  "bullet_points": [],
  "description": "NexusPro Elite 15 R75800H 16GB/750GB W11H",
  "attribute": {
    "name": "hard_disk.size",
    "value": "750.0 GB"
  },
  "label": "Correct",
  "original_label": "Correct",
  "label_source": "arena_majority"
}
```

### Fields

| Field | Type | Description |
|---|---|---|
| `id` | string | Anonymised product identifier, `<lang>_<n>` (e.g. `it_0`) |
| `language` | string | Locale code: `de`, `en`, `es`, `fr`, or `it` |
| `category` | string | Product category (one of 229) |
| `title` | string | Product title |
| `bullet_points` | list[string] | Bullet-point features |
| `description` | string | Long-form product description |
| `attribute.name` | string | Attribute being verified (one of 792) |
| `attribute.value` | string | Candidate value for that attribute |
| `label` | string | Final released label: `Correct`, `Incorrect`, `Unknown`, or `null` on 3 undecisive triage rows |
| `original_label` | string | Label before this cleaning pass (audit trail) |
| `label_source` | string | Provenance tag; see [Label Provenance](#label-provenance) |
| `triage_note` | string, optional | Free-text auditor note when present |

## Dataset Statistics

| Locale | Products | Correct | Incorrect | Unknown |
|---|---:|---:|---:|---:|
| ES | 3,159 | 1,483 | 501 | 1,175 |
| FR | 3,501 | 1,670 | 550 | 1,281 |
| IT | 3,007 | 1,452 | 442 | 1,113 |
| DE | 3,059 | 1,461 | 512 | 1,086 |
| EN* | 3,180 | 1,320 | 351 | 1,509 |
| **Total** | **15,906** | **7,386** (46.4%) | **2,356** (14.8%) | **6,164** (38.8%) |

- **Product categories**: 229 (4-locale set) / 274 (English)
- **Unique attributes**: 792 (4-locale set) / 727 (English)
- **Distinct product–attribute combinations**: 2,607 (4-locale set)
- **Minimum products per category**: 30† (4-locale set)
- **Judge configurations behind each label**: 21

<a id="english-note"></a>

<sub>* English is released as a fifth-locale extension of the paper's four-locale benchmark. Two important differences apply to English rows only:  (1) labels come from the arena majority vote alone — no expert adjudication or human triage audit was performed on English, whereas DE / ES / FR / IT went through the paper's full protocol;  (2) the arena's Claude family judge is Claude 4 Sonnet, since the paper's Claude 3.5 Sonnet inference profile was retired before English could be run. Estimated label quality for English is therefore lower than the 97.9% figure that applies to the paper's four locales, and should be treated accordingly.</sub>

<sub>† Target minimum on the 4-locale set. A small number of items were removed at release time because their category could not be reliably determined, leaving 14 of the 229 categories slightly below the target with 26–29 products each. The largest category contains 209 products.</sub>

Full per-category and per-attribute breakdowns are in the paper appendix.

## Label Provenance

Every row carries a `label_source` tag so the origin of its label is auditable.

| `label_source` | Count | Share | Meaning |
|---|---:|---:|---|
| `arena_majority` | 14,639 | 92.0% | Label taken from the 21-judge arena majority (agreement stratum for DE / ES / FR / IT; all 3,180 English rows) |
| `expert_adjudication` | 1,258 | 7.9% | Human reviewer adjudicated the row (arena–pipeline disagreement stratum on DE / ES / FR / IT only) |
| `arena_majority_triage_unsure` | 6 | <0.1% | Arena majority kept; auditor 1 marked the row "unsure" during the 400-row audit |
| `triage_undecisive` | 3 | <0.1% | Arena majority kept; auditor 1 disagreed but did not commit to a replacement label |

The 3 `triage_undecisive` rows have `label` equal to the arena majority and the auditor's note preserved in `triage_note` so downstream users can filter or hand-correct them. `expert_adjudication` never appears on English rows — all English labels are arena-only, per the [English note](#english-note).

## How to Use

### Python (standard library)

```python
import json
from pathlib import Path

root = Path("SynthAVE")
data = {}
for lang, locale in [("german", "de"), ("italian", "it"),
                     ("french", "fr"), ("spanish", "es"),
                     ("english", "en")]:
    with open(root / lang / "dataset.json", encoding="utf-8") as fh:
        data[locale] = json.load(fh)

print(f"IT rows: {len(data['it'])}")
print(f"First IT record: {data['it'][0]}")
```

### pandas

```python
import pandas as pd

df = pd.concat([
    pd.read_json("SynthAVE/german/dataset.json"),
    pd.read_json("SynthAVE/italian/dataset.json"),
    pd.read_json("SynthAVE/french/dataset.json"),
    pd.read_json("SynthAVE/spanish/dataset.json"),
    pd.read_json("SynthAVE/english/dataset.json"),
], ignore_index=True)

df["attribute_name"]  = df["attribute"].str["name"]
df["attribute_value"] = df["attribute"].str["value"]

print(df.groupby("language")["label"].value_counts())
```

### Hugging Face `datasets`

```python
from datasets import load_dataset

ds = load_dataset(
    "json",
    data_files={
        "de": "SynthAVE/german/dataset.json",
        "it": "SynthAVE/italian/dataset.json",
        "fr": "SynthAVE/french/dataset.json",
        "es": "SynthAVE/spanish/dataset.json",
        "en": "SynthAVE/english/dataset.json",
    },
)
```

## Dataset Creation

**Source data.** Product descriptions are synthetically generated from seed products drawn from commercial e-commerce catalogs. Brand names, model identifiers, and potentially identifying signals are anonymised during generation, and this release applies a second-pass sweep that replaces any real brand that slipped through with a deterministic synthetic name. Whitespace and case variants of the same brand share one synthetic replacement so label semantics (Correct / Incorrect / Unknown for `brand`-attribute rows) are preserved. A per-locale audit trail is available in `brand_replacements.json`.

**Attribute-value pairs.** Values are deliberately manipulated to produce a usable mix of `Correct`, `Incorrect`, and `Unknown` labels (Section 3 of the paper).

**Annotation.** Each row is judged by 21 LLM configurations (7 model families × 3 prompt versions). The 21 votes are aggregated by simple majority. When the arena majority disagrees with the synthetic-pipeline label, an expert adjudicates. Agreement cases are validated on a stratified sample of 400 rows (100 per locale, 100 unanimous / 300 mixed-agreement), audited independently by two experts (Cohen's κ = 0.98 between auditors). Full details in Section 4 and Appendix A of the paper.

**This release.** The released `label` follows the paper's disagreement-based annotation strategy: `expert_adjudication` when a human reviewer examined the row (arena–pipeline disagreement stratum on the 4-locale set, 1,258 rows, ~10 % of the 4-locale benchmark) and `arena_majority` otherwise. On top of this, this release applies auditor 1's 9 note-mapped overturns from the 400-row triage sample as `expert_adjudication`; the 3 auditor-1 `disagree` rows whose notes did not commit to a specific alternative label are flagged `triage_undecisive` rather than defaulted to `Unknown`, since `Unknown` is a substantive verification outcome, not a stand-in for "we don't know".

**English addition.** The English locale (3,180 rows) uses the same 21-configuration arena as the paper's four locales, with one adjustment: the Claude judge is Claude 4 Sonnet, because the paper's Claude 3.5 Sonnet inference profile was retired mid-way through the pipeline. Every English label comes from the arena majority vote; no human adjudication or triage audit was performed. English rows are therefore included in the release as a supplementary track, and their labels should be regarded as silver-standard-with-a-lower-floor relative to the paper's four core locales.

## Considerations for Using the Data

**Intended uses.**
- Benchmarking attribute-value verification models across languages.
- Evaluating LLM-as-judge and multi-model ensemble methodologies.
- Studying label noise, adjudication protocols, and audit-based silver labels.

**Limitations.**
- SynthAVE is a **test set**, not a training corpus. Category coverage is prioritised over volume.
- Labels for agreement cases are not independent of the arena's own judgments; residual error is estimated at 2.1% (Appendix on quality).
- Verifiability of an attribute varies by attribute type; `UNKNOWN` reflects insufficient product text rather than a truly unknown ground truth.

**Bias and privacy.** All products are synthetic; brands and identifiers are anonymised. Content nonetheless inherits the biases of the LLMs used to generate and judge it.

**Out-of-scope uses.** SynthAVE is licensed under CC BY-NC 4.0 and may not be used for commercial purposes.

## Citation

If you use SynthAVE in your work, please cite the paper. The arXiv preprint is available now; the camera-ready version will appear in the EMNLP 2026 Industry Track proceedings.

```bibtex
@misc{scarinci2026synthavescalablesyntheticlabeling,
  title         = {SynthAVE: Scalable Synthetic Labeling for E-Commerce with LLM-Arena Validation},
  author        = {Andrea Scarinci and Virginia Negri and Brayan Impata and Suleiman Khan and Victor Martinez and Marcello Federico},
  year          = {2026},
  eprint        = {2607.07469},
  archivePrefix = {arXiv},
  primaryClass  = {cs.CL},
  url           = {https://arxiv.org/abs/2607.07469}
}
```

Please also cite the underlying attribute-aware product-generation pipeline:

```bibtex
@misc{negri2025attributeawarecontrolledproductgeneration,
  title         = {Attribute-Aware Controlled Product Generation with LLMs for E-commerce},
  author        = {Virginia Negri and Víctor Martínez Gómez and Sergio A. Balanya and Subburam Rajaram},
  year          = {2025},
  eprint        = {2601.04200},
  archivePrefix = {arXiv},
  primaryClass  = {cs.CL},
  url           = {https://arxiv.org/abs/2601.04200}
}
```

## License

Released under **Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)**. You are free to share and adapt the material for non-commercial purposes, provided appropriate credit is given.

- Full license text: [`LICENSE.txt`](./LICENSE.txt)
- Attribution and third-party notices: [`NOTICE.txt`](./NOTICE.txt)
- Human-readable summary: <https://creativecommons.org/licenses/by-nc/4.0/>

## Contact

For research inquiries, please contact the paper authors. Issues and questions are welcome. Pull requests are not accepted: this repository is maintained as a frozen research artifact matching the published paper.
