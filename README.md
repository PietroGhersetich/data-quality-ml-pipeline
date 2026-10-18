# Data Quality and Machine Learning Pipeline

This repository contains a data quality pipeline and machine learning project developed for the **Data Information Quality** course at **Politecnico di Milano** (A.Y. 2025/2026).

Using a real-world business registry dataset, the project assesses data quality, applies a structured cleaning pipeline, and evaluates the cleaned data through a downstream multi-label classification task. Further details are available in the [Project Report](report/data_information_quality_report.pdf).

---

## Overview

The original dataset contains **3,909 records** about personal-care businesses. The project covers three stages:

1. **Quality Assessment**: profile missing values, validity, consistency, and redundant fields.
2. **Data Cleaning**: recover address information, consolidate duplicates, handle outliers, and prepare service labels.
3. **Machine Learning**: compare classification results using raw and cleaned data for four service categories: **Tipo A**, **Tipo B**, **Tipo C**, and **Tipo D**.

Since a business may belong to more than one category, the prediction task is **multi-label classification**.

---

## Key Challenges

- **Missing Values**: `Prevalente` has over **92%** missing values; area fields also contain substantial gaps.
- **Address Redundancy**: `Ubicazione` duplicates information found in separate address columns, but can also help recover missing values.
- **Duplicate Records**: entries sharing the same street type, street name, and civic number must be consolidated.
- **Label Imbalance**: the service categories are unevenly represented, with `Tipo B` dominant and `Tipo C` rare.

After cleaning and consolidation, the dataset contains **3,248 records**.

### Service-category distribution

| Target | Positive records | Share |
| --- | ---: | ---: |
| **Tipo A** | 1,197 | 36.85% |
| **Tipo B** | 2,729 | 84.02% |
| **Tipo C** | 139 | 4.28% |
| **Tipo D** | 271 | 8.34% |

*Categories are multi-label, so percentages do not sum to 100%.*

---

## Data Quality Assessment

The dataset was assessed for completeness, accuracy, and consistency. Timeliness could not be measured because no suitable time field was available.

| Dimension | Check | Before | After |
| --- | --- | ---: | ---: |
| **Completeness** | Non-null cell ratio | 79.03% | **100.00%** |
| **Accuracy** | Valid `ZD` values | 99.97% | **100.00%** |
| **Accuracy** | Valid `Codice via` values | 99.87% | **100.00%** |
| **Consistency** | `Civico` format | 99.92% | **99.94%** |

---

## Data Cleaning Pipeline

1. **Label Engineering**: map service descriptions to four Boolean targets using keyword rules; remove records without a usable label.
2. **Address Recovery**: parse `Ubicazione` to fill missing street and civic-number fields, reducing incomplete address records from **77 to 1** before final checks.
3. **Duplicate Aggregation**: group records by normalized address, prioritize more complete entries, and combine area values and labels.
4. **Outlier Treatment**: use an asymmetric IQR rule to flag **59** outliers in `Superficie lavorativa` and **25** in `Superficie altri usi`.
5. **Imputation and Validation**: fill missing area values and re-check retained columns, domain constraints, and consistency. The final cleaned table has no null values in its retained columns.

---

## Machine Learning Approach

A **PyTorch feed-forward neural network** is trained on raw and cleaned data to evaluate the impact of data preparation. It predicts the four service labels from `ZD`, `Tipo via`, `Superficie altri usi`, and `Superficie lavorativa`.

The pipeline includes categorical encoding, numerical scaling, weighted `BCEWithLogitsLoss`, dropout, AdamW weight decay, early stopping, and train/validation/test splits based on target combinations where possible.

---

## Results

### Data quality

Overall completeness increased from **79.03% to 100%**. The checked `ZD` and `Codice via` fields also reached **100% validity** under the notebook's validation rules.

### Cleaned-dataset classification

| Metric | Test result |
| --- | ---: |
| **Exact-match accuracy** | **52.92%** |
| Weighted precision | 71.28% |
| Weighted recall | 78.34% |
| Weighted F1-score | **74.22%** |

Per-class results highlight the impact of imbalance:

| Target | Precision | Recall | F1-score |
| --- | ---: | ---: | ---: |
| **Tipo A** | 0.62 | 0.53 | 0.57 |
| **Tipo B** | 0.84 | 0.98 | 0.91 |
| **Tipo C** | 0.00 | 0.00 | 0.00 |
| **Tipo D** | 0.20 | 0.37 | 0.26 |

The model performs best on `Tipo B` and struggles with the minority categories, especially `Tipo C`. These results show that data cleaning improves data usability, but does not by itself resolve class imbalance or limited feature information.

---

## Limitations

- **Class Imbalance** limits performance on rare service categories.
- **Limited Features**: the selected address and area attributes do not fully describe business services.
- **Rule-Based Labels** may introduce noise when service descriptions are ambiguous.
- **Imputation Assumptions**: replacing missing `Superficie altri usi` values with zero may treat unknown values as genuine zeros.

Potential next steps include improving label rules, adding more informative features, and testing alternative approaches for rare categories.

---

## Tech Stack

- **Python**, **Pandas**, and **NumPy** for data processing
- **Matplotlib** and **Seaborn** for visualization
- **PyTorch** for classification
- **Scikit-learn** for splitting and evaluation
- **TensorBoard** for training monitoring
- **Jupyter Notebook / Google Colab** for experimentation

---

## Repository Structure

```text
.
├── notebooks/
│   └── data_science_pipeline.ipynb          # Profiling, cleaning, and model experiments
├── report/
│   └── data_information_quality_report.pdf  # Project report
└── README.md                                # Project documentation
```

The source CSV is not included. The notebook expects `italian/Comune-di-Milano-Servizi-alla-persona-parrucchieri-estetisti.csv` and contains a Google Drive setup cell; adjust the data path for your environment before running it.

---

## Team

**Alessio Migliore**, **Pietro Ghersetich**, **Filippo Rodella**

*Politecnico di Milano — Data Information Quality, A.Y. 2025/2026*
