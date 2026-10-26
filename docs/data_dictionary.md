# PaySim — Data Dictionary

> **Project:** Fraud Data Engineering & Analytics  
> **Dataset:** PaySim Synthetic Financial Dataset  
> **Status:** Schema verified against the local CSV using pandas. Full-dataset quality statistics are intentionally reserved for the profiling stage.

## 1. Dataset overview

| Property                | Description                                                                                                                                       |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| Dataset name            | PaySim — Synthetic Financial Datasets for Fraud Detection                                                                                         |
| Source                  | [Kaggle: ealaxi/paysim1](https://www.kaggle.com/datasets/ealaxi/paysim1)                                                                          |
| Domain                  | Simulated mobile-money transactions and fraud detection                                                                                           |
| Data format             | CSV                                                                                                                                               |
| Approximate volume      | 6.36 million transaction records (verify exact count locally)                                                                                     |
| Unit of observation     | One transaction per row                                                                                                                           |
| Target                  | `isFraud` (binary simulated fraud label)                                                                                                          |
| License                 | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) — as displayed on the dataset page; verify the dataset's attribution requirements |
| Local raw-data location | `data/raw/` (excluded from Git)                                                                                                                   |
| Documentation date      | 2026-10-26                                                                                                                                        |

### Intended uses

- Data ingestion, profiling, validation and quarantine of malformed records.
- SQL transaction analysis, aggregation and rule-based fraud monitoring.
- Data warehouse development and Tableau visualizations.
- Fraud detection modeling with documented leakage controls.

## 2. Schema

The **pandas dtypes below were observed directly from the local PaySim CSV** supplied by the project author. They describe pandas inference, not PostgreSQL column definitions. `object` denotes text values in these columns.

| Column           | Observed pandas dtype | Meaning                                            | Validation / interpretation notes                                                                                                                      |
| ---------------- | --------------------- | -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `step`           | `int64`               | Simulation time step; one step represents one hour | Nonnegative; not a real-world calendar timestamp                                                                                                       |
| `type`           | `object`              | Transaction type                                   | Expected categories: `CASH_IN`, `CASH_OUT`, `DEBIT`, `PAYMENT`, `TRANSFER`                                                                             |
| `amount`         | `float64`             | Transaction amount in simulated monetary units     | Check missing, nonnumeric, negative and extreme values; do not assume real currency                                                                    |
| `nameOrig`       | `object`              | Originating account identifier                     | Required for transfer network analysis; treat as opaque ID                                                                                             |
| `oldbalanceOrg`  | `float64`             | Originator's balance before transaction            | Do not treat zero as missing without evidence                                                                                                          |
| `newbalanceOrig` | `float64`             | Originator's balance after transaction             | Compare against expected balance movement only where transaction semantics support it                                                                  |
| `nameDest`       | `object`              | Destination account identifier                     | May represent different account/entity types; inspect prefixes before interpretation                                                                   |
| `oldbalanceDest` | `float64`             | Destination balance before transaction             | Zero may be a recorded value, not necessarily an error                                                                                                 |
| `newbalanceDest` | `float64`             | Destination balance after transaction              | Some balance fields may not behave as expected for all transaction types                                                                               |
| `isFraud`        | `int64`               | Simulated fraud ground-truth label                 | Expected values: 0 or 1; labels reflect simulator assumptions                                                                                          |
| `isFlaggedFraud` | `int64`               | Flag produced by a simulation rule                 | **Not** a second independent ground-truth label; avoid treating as an ordinary predictive feature without explaining leakage / deployment availability |

## 3. Data relationships and grain

- **Grain:** One row corresponds to one simulated transaction.
- **Origin → destination:** `nameOrig` and `nameDest` identify transaction endpoints.
- **Time:** `step` gives relative simulation time, not a date/time with a timezone.
- **No guaranteed transaction ID:** Do not assume an explicit unique transaction key exists. If an ingestion key is needed, create a reproducible row identifier and document how it is generated.
- **No native customer KYC, device ID, IP address or merchant risk table:** Do not invent those fields in the original dataset schema.

## 4. Data-quality checks (planned)

| Check               | Proposed rule                                                                          | Action                                                  |
| ------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| Schema conformity   | Required 11 columns exist; expected types can be parsed                                | Reject/quarantine incompatible input                    |
| Null identifiers    | `nameOrig`, `nameDest` and `type` are populated                                        | Quarantine and report                                   |
| Amount validity     | `amount` is numeric and >= 0                                                           | Quarantine invalid records; investigate zero separately |
| Label domain        | `isFraud`, `isFlaggedFraud` belong to {0,1}                                            | Quarantine invalid labels                               |
| Time validity       | `step` is an integer >= 0                                                              | Quarantine invalid records                              |
| Duplicates          | Report exact duplicate rows; do not automatically delete repeated-looking transactions | Review with business rules                              |
| Balance consistency | Compare balances by transaction type, with documented exceptions                       | Flag for investigation, not automatic deletion          |
| Class imbalance     | Report count and proportion of each fraud label                                        | Use suitable evaluation metrics later                   |

> **Important:** This section specifies _planned validation rules_. It does not claim that any errors have been found yet.

## 5. Known limitations and modeling cautions

1. PaySim is **synthetic**, so patterns and fraud labels should not be presented as measured real-world banking behavior.
2. `step` is an hourly simulation index. Converting it to a real calendar timestamp would invent information.
3. The source does not natively contain all entities needed for a full AML/KYC heterogeneous graph; use a dedicated AML dataset for that project.
4. Beware of **target leakage**: inspect the meaning and availability of balance fields and `isFlaggedFraud` before using them in predictive models.
5. High classification accuracy can be misleading with rare fraud events; report precision, recall, F1, AUPRC and confusion matrices.
6. Dataset license terms do not establish that the dataset is suitable for production financial decision-making.

## 6. Source attribution and reuse

**Source:** PaySim Synthetic Financial Dataset, distributed via [Kaggle](https://www.kaggle.com/datasets/ealaxi/paysim1). PaySim originates from published simulation research; consult the dataset page and original paper for the correct creator/paper citation before publishing your final portfolio.

**License:** [Creative Commons Attribution–ShareAlike 4.0 International (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/).

**Changes:** No changes to the original dataset are claimed in this schema documentation. Future transformations, injected data-quality test cases and derived exports will be documented separately. Redistribution of adapted licensed material must follow applicable attribution and ShareAlike terms. The original large CSV is not committed to this repository.

## 7. Local schema verification and profiling status

| Metric                        | Observed result                         |
| ----------------------------- | --------------------------------------- |
| Actual row count              | Not measured in this documentation step |
| Actual column count           | **11 (verified)**                       |
| File size                     | Not measured                            |
| Missing values by column      | Not measured                            |
| Exact duplicate row count     | Not measured                            |
| Transaction type distribution | Not measured                            |
| Fraud count and rate          | Not measured                            |
| Observed pandas dtypes        | **Verified; see Section 2**             |

## 8. Reproducibility

1. Download the dataset from the source link above.
2. Save the original CSV under `data/raw/`.
3. Do not commit raw data or credentials to Git.
4. Run the profiling notebook/script once available and update Section 7 with measured results.

---

_Documentation version: 1.0 — Day 002. Schema verified; full-dataset statistics will be added after measurement._
