# Dataset

SG-LegalCite is a principle-augmented benchmark for legal citation retrieval
in Singapore law, comprising 100,890 case–principle pairs extracted from
8,523 Supreme Court judgments spanning 2000–2025.

The dataset files are hosted on HuggingFace:
https://huggingface.co/datasets/ShannonLeeYuehErn/SG-LegalCite

## Statistics

| Attribute | Value |
|---|---|
| Total case–principle pairs | 100,890 |
| Unique citing judgments | 8,523 |
| Unique cited cases | 48,478 |
| Unique issues | 86,519 |
| Unique issue groups | 9,748 |
| Time span | 2000–2025 |
| Courts covered | SGCA, SGCAI, SGHC, SGHCF, SGHCR |

Each judgment is uniquely identified by `URL`, which corresponds
1:1 with the Singapore neutral citation (`Full_Reference`) of the citing
judgment (e.g., `https://www.elitigation.sg/gd/s/2023_SGCA_15` ↔ `[2023] SGCA 15`).

## Files

| File | Size | Description |
|---|---|---|
| `COMBINED_ALL_CASES_FINAL_V2.csv` | 764 MB | Full dataset — 100,890 case–principle pairs |
| `stage2_direct_candidate_pools_v2.json` | 132 MB | 1000-way candidate pools for fact-only retrieval |
| `stage2_single_stage_pools.json` | 144 MB | 1000-way candidate pools for principle-augmented retrieval |
| `stage2_case_lookup.json` | 3.45 MB | Case ID to case text lookup table |

## Fields

The CSV columns, in the order they appear in the file:

| Field | Description |
|---|---|
| `Year` | Year of the citing judgment |
| `Court_Type` | Court type code (SGCA, SGCAI, SGHC, SGHCF, SGHCR) |
| `Case_Number` | Case number of the citing judgment |
| `URL` | URL of the citing judgment on eLitigation |
| `Full_Reference` | Neutral citation of the citing judgment |
| `Case Name` | Full case name of the citing judgment |
| `Current Court Level` | Court level of the citing judgment |
| `Extract of Facts` | LLM-summarised factual background (~45 tokens) |
| `Cited Case` | Name of the cited Singapore case |
| `Paragraph` | Citation paragraph with ±5 surrounding context paragraphs |
| `Key Principles Illustrated` | Legal principle for which the case is cited |
| `Issue` | Specific legal issue addressed |
| `Issue Group` | Fine-grained doctrinal tag (e.g., "Damages", "Contract") |
| `Court Level` | Court level of the **cited** case |
| `Precedential Weight (Binding, Comity, or Persuasive)` | Precedential status of the cited case: Binding, Comity, or Persuasive |

## Loading

```python
import pandas as pd

df = pd.read_csv("COMBINED_ALL_CASES_FINAL_V2.csv", encoding="latin-1")
```

## Splits

The dataset ships as a single file; splits are created at load time. The
8:1:1 split is performed at the **judgment level** (by unique `URL`)
to prevent leakage: all records from one citing judgment fall in the same
split. `random_state=42` is used for reproducibility.

```python
from sklearn.model_selection import train_test_split

unique_urls           = df["URL"].unique()
train_urls, temp_urls = train_test_split(unique_urls, test_size=0.2, random_state=42)
val_urls, test_urls   = train_test_split(temp_urls, test_size=0.5, random_state=42)

train_df = df[df["URL"].isin(train_urls)]
val_df   = df[df["URL"].isin(val_urls)]
test_df  = df[df["URL"].isin(test_urls)]
```
