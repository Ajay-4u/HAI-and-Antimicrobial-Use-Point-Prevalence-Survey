# HAI point prevalence survey: data quality and survey report (R)

An R pipeline for a healthcare-associated infection (HAI) and antimicrobial use point prevalence survey collected in REDCap. It validates the raw export, raises a query log for hospitals to resolve, classifies HAIs, and renders a ten-page HTML report.

> **All data in this repository is synthetic.** `R/00_generate_synthetic_data.R` builds a REDCap export of 540 patients in 10 invented hospitals from scratch, in the exact 562-column layout of the survey instrument. No real patient, facility or programme data was used, and no real records were used as a template for the values.

**[View the rendered report](reports/hai_pps_report.html)** (download and open in a browser, or see it on my portfolio).

## The report

| Page | Contents |
|---|---|
| 1. Data quality | Patient age and length of stay by ward with outliers labelled by record ID; patients on the same ward with different survey dates; every other issue by rule, hospital and data collector; the query log |
| 2. General survey report | Patients meeting the non-surgical and surgical HAI chart review criteria, and the total for review, by ward and hospital |
| 3. Microbiology | Culture positivity and organisms for blood, urine and respiratory cultures |
| 4. Devices | Devices on the survey date among patients on antimicrobials; devices in the 14 days before the survey; mean device duration |
| 5. Antimicrobial and diagnostic metrics | Prescribing among reviewed patients, and use of blood, urine and respiratory cultures and chest imaging |
| 6 to 9. UTI, BSI, pneumonia, SSI | Patients reviewed, cases meeting the definition, infections present on admission, signs and symptoms, device-associated cases, organisms, by ward |
| 10. HAI summary | Prevalence with 95% confidence intervals, by type, ward and hospital; device-associated HAIs; causative organisms; agreement with clinician documentation |

## How it works

| Script | What it does |
|---|---|
| `R/00_generate_synthetic_data.R` | Builds each patient to be clinically consistent (ward drives age, devices and surgery; a latent condition drives cultures, imaging, symptoms and classification forms), then plants 64 known data errors and writes an answer key |
| `R/01_validation_functions.R` | General-purpose helpers: standardise and log formatting, parse dates, separate exact duplicates from conflicting ones, build rules, run them and produce a query log |
| `R/02_pps_rules.R` | 122 rules for this instrument. Generic checks (missing values, codes, ranges, date formats) are generated from `instrument/codebook.csv`; logic checks cover date order, ward survey dates, age against ward, antimicrobial and device consistency, duplicate patients, and classification forms that should have been completed |
| `R/03_derive_indicators.R` | Reshapes the repeating REDCap blocks (10 antimicrobials, 7 microbiology tests, 7 imaging studies, 3 of each device) into long tables, applies the review criteria, and classifies HAIs |
| `R/04_run_pipeline.R` | Runs validation and derivation, and writes the query logs and analysis dataset |
| `reports/hai_pps_report.Rmd` | The ten-page report |

### Survey definitions are settings, not buried logic

The review criteria and HAI windows sit at the top of `R/03_derive_indicators.R` (`SURVEY_PARAMS`), so they can be aligned with the protocol in use. The HAI classification is a **simplified** version of standard surveillance definitions written for demonstration; it is not a full implementation of NHSN or ECDC criteria.

### Design principles

- **Only unambiguous formatting is corrected in code, and every change is logged.** Anything that needs a judgement is raised as a query for the hospital to resolve from the patient folder.
- **Records with a critical query are held back from analysis** until resolved; records with major or minor queries stay in, flagged.
- **One problem, one query.** Derived checks are skipped when an input is already impossible, so hospitals are not sent duplicate queries for the same mistake.

## Tested

`tests/testthat` checks that:

- the generated export matches the REDCap template column for column;
- the rules raise **no** queries on the error-free version of the data;
- every one of the 64 planted errors is caught by the rule expected to catch it;
- the HAI classification recovers every patient's true condition on clean data, and does not count infections present on admission or colonisation as HAIs.

## Running it

Requires R 4.2 or later:

```r
install.packages(c("dplyr", "tidyr", "readr", "purrr", "stringr", "tibble", "ggplot2",
                   "ggrepel", "forcats", "knitr", "kableExtra", "rmarkdown", "testthat"))
```

From the repository root:

```sh
Rscript run_all.R
```

This generates the data, runs validation, runs the tests and renders `reports/hai_pps_report.html`.

## Outputs

```
outputs/query_log.csv                    every open query, with a plain-language request to the hospital
outputs/query_logs_by_site/              one query log per hospital, ready to send
outputs/rule_catalogue.csv               every rule applied, with severity
data/clean/pps_analysis_ready.csv        one row per patient, with review status and HAI flags
reports/hai_pps_report.html              the report
```

## Author

Ajuzieogu Egbulefu, public health data and MEL specialist. [GitHub](https://github.com/Ajay-4u)
