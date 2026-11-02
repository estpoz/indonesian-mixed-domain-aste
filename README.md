# An Indonesian Mixed-Domain Dataset for Aspect Sentiment Triplet Extraction

This dataset contains 7,496 Indonesian user reviews annotated for Aspect Sentiment Triplet Extraction (ASTE) across three domains: e-commerce, mobile applications, and tourism/public places. Each annotation connects an aspect with its corresponding opinion expression and sentiment polarity.

The dataset supports Indonesian aspect-based sentiment analysis research, cross-domain evaluation, and comparisons between supervised and prompting-based approaches.

## Dataset Overview

| Component | Count |
|---|---:|
| Reviews | 7,496 |
| Development reviews | 375 |
| Test reviews | 7,121 |
| ASTE triplets | 22,431 |
| Positive triplets | 11,709 |
| Negative triplets | 10,364 |
| Neutral triplets | 358 |
| Implicit-aspect triplets | 4,455 |

| Domain | Development | Test | Total |
|---|---:|---:|---:|
| E-commerce | 125 | 2,374 | 2,499 |
| Mobile applications | 125 | 2,373 | 2,498 |
| Tourism/public places | 125 | 2,374 | 2,499 |
| Total | 375 | 7,121 | 7,496 |

The dataset provides development and test splits; it does not include a training split.

## Data Sources

Reviews were selected from three existing datasets:

- PRDECT-ID: e-commerce reviews.
- Indonesian Google Play review dataset: mobile-application reviews.
- Natural Tourist Attractions Review Dataset: tourism/public-place reviews.

The initial collection contained 7,500 reviews, with 2,500 from each domain. Domain-stratified sampling allocated 125 reviews per domain to the development set. Four unusable test reviews were excluded during final token conversion.

## Dataset File

The finalized dataset is provided in:

`mixed_domain_aste_final_7496.csv`

The file uses UTF-8 encoding with a byte order mark (BOM) and comma-separated columns.

### Main Columns

| Column | Description |
|---|---|
| global_id | Review identifier. |
| split | Development (`dev`) or test (`test`) assignment. |
| domain | Review domain: `ecommerce`, `mobile_app`, or `tourism_public_place`. |
| text_review | Original review text. |
| final_spans | Final annotations stored as a JSON list of aspect–opinion–sentiment triplets. |
| exclude_row | Exclusion flag. All reviews in this release are retained and marked `NO`. |

Additional columns preserve annotation-review metadata. Use `final_spans` as the reference labels for model evaluation.

## Annotation Format

Each triplet in `final_spans` has the following structure:

```text
[aspect, opinion, sentiment]
```

Example:

```json
[
  ["pengiriman", "cepat", "POS"],
  ["kemasan", "rusak", "NEG"],
  ["NULL", "recommended", "POS"]
]
```

Sentiment labels are:

- `POS`: positive.
- `NEG`: negative.
- `NEU`: neutral.

`NULL` indicates an implicit aspect whose target is not explicitly stated in the review. A review may contain multiple triplets. Explicit aspect and opinion spans are exact, contiguous text spans limited to four whitespace tokens each.

The CSV contains surface-text annotations. In the corresponding Indo LEGO-ABSA token-index representation, spans use zero-based whitespace-token indices, and implicit aspects are represented by `[-1]`.

## Annotation and Validation

Annotations were developed through GPT-assisted annotation, automated structural validation, targeted manual review, and token-span alignment.

Automated checks examined annotation structure, exact span alignment, sentiment labels, and aspect–opinion relationships. A total of 1,060 flagged reviews were manually examined and corrected, followed by targeted repairs.

All 375 development reviews were independently assessed by two validators. Exact triplet-label agreement before adjudication was 87.73%. An adjudicator resolved 46 annotation disagreements, resulting in revisions to 30 development reviews.

Independent double-human validation was limited to the development set. These agreement results should not be interpreted as validation statistics for the entire test set.

## Loading the Dataset

Install pandas:

```bash
pip install pandas
```

Load the CSV and parse its annotations:

```python
import json
import pandas as pd

data = pd.read_csv(
    "mixed_domain_aste_final_7496.csv",
    encoding="utf-8-sig",
    keep_default_na=False,
)

data["triplets"] = data["final_spans"].apply(json.loads)

dev = data[data["split"] == "dev"].copy()
test = data[data["split"] == "test"].copy()

print(f"Total reviews: {len(data)}")
print(f"Development reviews: {len(dev)}")
print(f"Test reviews: {len(test)}")
```

## Supported Tasks

### Aspect Sentiment Triplet Extraction

Use the complete aspect–opinion–sentiment triplets in `final_spans` as reference annotations.

### Aspect–Opinion Pair Extraction

AOPE labels can be derived by removing sentiment polarity and deduplicating identical aspect–opinion pairs within each review:

```python
def to_aope(triplets):
    return sorted({
        (aspect, opinion)
        for aspect, opinion, sentiment in triplets
    })

data["aope_pairs"] = data["triplets"].apply(to_aope)
```

## Evaluation Recommendations

Preserve the original review text and provided split assignments. Use development reviews for prompt design and other development decisions, and reserve test reviews for final evaluation.

Report precision, recall, and F1-score, specifying the matching rules, treatment of implicit aspects, and whether evaluation uses surface expressions or token indices. Report results by domain where relevant.

If converting surface annotations into token indices, use whitespace tokenization and check alignment carefully when an expression occurs more than once in a review.

## License

No license.
