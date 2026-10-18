# Relational Databases — Final Project

**Repository:** `goit-rdb-fp`  
**Database:** PostgreSQL  
**Environment:** Google Colab (Python, pandas, SQLAlchemy, `pgserver`)  
**Dataset:** `infectious_cases.csv`

## Project overview

This project analyzes infectious disease statistics across countries, regions, and years. It demonstrates PostgreSQL schema creation and data import, relational normalization to **Third Normal Form (3NF)**, grouped statistical analysis, date calculations, and reusable SQL functions.

The work is organized into five tasks in [`goit_rdb_fp.ipynb`](goit_rdb_fp.ipynb).

## Dataset

The source file, `infectious_cases.csv`, contains **10,521 records** and **12 columns**, covering **245 entities** and the years **1920–2023**. It includes entity names and codes, observation years, and indicators for several infectious diseases (including rabies, malaria, polio, HIV, and tuberculosis).

Some entity codes and disease measurements are missing. These are preserved as SQL `NULL` values rather than replaced with zeros.

**Original dataset:** [infectious_cases (Google Drive)](https://drive.google.com/file/d/1dQf-uVXzYPYYGgXsG1y4Q1qDdhP2miZl/view?usp=drive_link)

## Implementation

### Task 1 — Database setup and import

- Create the PostgreSQL `pandemic` schema and set the SQL search path.
- Read `infectious_cases.csv` with pandas and import it into `pandemic.infectious_cases`.
- Validate the record count, unique entities, year range, and missing rabies values.

**Validation:** 10,521 records, 245 entities, and 3,741 missing `Number_rabies` values.

### Task 2 — Third Normal Form

The original table is split into two related tables:

| Table | Purpose | Key |
| --- | --- | --- |
| `pandemic.entities` | Unique entity name and code combinations | `id` (primary key) |
| `pandemic.infectious_cases_normalized` | Annual disease indicators, linked to an entity | `(entity_id, year)` (composite primary key) |

The normalized fact table references `pandemic.entities(id)` through a foreign key. The join uses `IS NOT DISTINCT FROM` to correctly match entities with missing codes.

**Validation:** 245 entity records, all 10,521 annual records preserved, and **zero orphan foreign keys**.

### Task 3 — Rabies statistics

Calculate `AVG`, `MIN`, `MAX`, and `SUM` of `number_rabies` for each entity, excluding missing rabies observations. Sort by average rabies cases in descending order and return the top 10 entities.

The results include individual countries as well as aggregate entities, such as World and World Bank / WHO regional groups, because both are present in the original dataset.

### Task 4 — Date calculations

Use PostgreSQL built-in functions to derive:

- January 1 of the observation year using `MAKE_DATE(year, 1, 1)`;
- the current date using `CURRENT_DATE`;
- the number of complete years between these dates using `AGE()` and `EXTRACT(YEAR FROM ...)`.

The resulting year difference is evaluated at query execution time.

### Task 5 — User-defined functions

Implement and query two reusable SQL functions:

| Function | Description |
| --- | --- |
| `pandemic.calculate_years_difference(input_year)` | Number of complete calendar years from January 1 of the given year to the current date |
| `pandemic.calculate_period_average(annual_cases, divisor)` | Annual disease value divided by a period count; returns `NULL` for division by zero |

The period-average function is tested with divisors **12** (monthly), **4** (quarterly), and **2** (half-yearly), including use with non-null `number_rabies` values. These results are **arithmetic period averages**, not measured monthly or quarterly disease counts.

## How to run

1. Open [`goit_rdb_fp.ipynb`](goit_rdb_fp.ipynb) in **Google Colab**.
2. Use a Colab Python environment and run the dependency-installation cells. The notebook contains the `pgserver` compatibility installation used with Python 3.13, along with SQLAlchemy, pandas, and Psycopg 3.
3. Make `infectious_cases.csv` available in the Colab working directory **before running the CSV-loading cell**. Upload the file using the Colab Files panel (or uncomment the `files.upload()` call in the notebook).
4. Run the remaining cells in order, or use **Runtime → Run all** after the CSV is available.
5. Review the validation tables and query outputs saved in the notebook.

> **Reproducibility note:** The notebook's CSV-loading cell uses a local filename (`pd.read_csv("infectious_cases.csv")`). A fresh Colab runtime therefore requires the CSV to be uploaded first unless the loading cell is adapted to fetch the file automatically. Database objects are recreated by the notebook's SQL cells.

## Project files

```text
goit-rdb-fp/
├── goit_rdb_fp.ipynb       # SQL implementation and saved query outputs
├── infectious_cases.csv   # Input dataset
├── README.md              # Project documentation
└── screenshots/           # Selected evidence screenshots, if included
```

## Validation and limitations

The project validates that normalization preserves all source records, that foreign-key references are valid, and that the custom SQL functions return the expected results. The notebook was reported to complete **Restart & Run All** successfully in the prepared Colab environment.

Missing disease measurements are not treated as zero. Some entities are regional or global aggregates rather than individual countries, and their totals should not be interpreted as independent country-level observations. Date-based outputs change as `CURRENT_DATE` changes.
