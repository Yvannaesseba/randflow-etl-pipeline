
# RandFlow: Google Drive to PostgreSQL ETL Pipeline

## Project Overview

This project was completed as part of my Amdari Data & Cloud Engineering internship.

The task was to build an ETL pipeline for Sablewave Financial, a fictional South African bank. The pipeline automates the extraction of CSV files from Google Drive, cleans the data using Python and pandas, and loads it into PostgreSQL for reporting and analysis.

## Data Sources

Three datasets were provided:

| Dataset          | Records | Description                      |
| ---------------- | ------: | -------------------------------- |
| customers.csv    |     152 | Customer details and segments    |
| accounts.csv     |     221 | Account information and balances |
| transactions.csv |   2,025 | Financial transactions           |

The datasets are linked through customer and account IDs. Accounts are linked to customers, while transactions are linked to accounts.

## Tools and Project Structure

**Technologies:** Python, pandas, Google Drive API, PostgreSQL, SQLAlchemy, psycopg2 and VS Code.

The pipeline consists of five Python modules:

- `config.py` — Loads configuration settings.
- `extract.py` — Downloads files from Google Drive.
- `transform.py` — Cleans and transforms the data.
- `load.py` — Loads the data into PostgreSQL.
- `main.py` — Runs the ETL pipeline.

## ETL Process

The pipeline downloads the three CSV files using the Google Drive API and a service account.

During transformation, it removes duplicate records, standardises formatting, converts data types and handles missing or invalid values.

For transactions, a `signed_amount` column is created based on the transaction type. Purchases, withdrawals, transfers and fees are treated as outgoing amounts (negative), while deposits and refunds are treated as incoming amounts (positive).

The processed data is loaded into PostgreSQL under the `staging` schema. Records previously loaded from the same source file are deleted before reloading to prevent duplicates from accumulating.

## Setup and Execution

The project requires Python, PostgreSQL and a Google Cloud service account with access to the source files.

1. Create a virtual environment and install the dependencies:

   ```bash
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ```
2. Create a PostgreSQL database named `finance_db` and a user with the required permissions.
3. Enable the Google Drive API, save the service account key in `credentials/credentials_service_account.json`, and share the source folder with the service account.
4. Create a `.env` file containing `DRIVE_FOLDER_ID`, `SERVICE_ACCOUNT_FILE`, `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER` and `DB_PASSWORD`.
5. Run the pipeline:

   ```bash
   python src/main.py
   ```

The `.env` file and service account credentials should not be committed to GitHub.

## Results and Verification

| Dataset      | Original rows | Loaded rows |
| ------------ | ------------: | ----------: |
| Customers    |           152 |         150 |
| Accounts     |           221 |         220 |
| Transactions |         2,025 |       1,930 |

The pipeline removed 2 duplicate customer records and 1 duplicate account record. For transactions, 25 duplicate records were removed, followed by 70 records with missing or invalid dates or amounts.

The results were verified using SQL queries in PostgreSQL, including transaction counts, net amounts by category and transactions by customer segment.

The pipeline was also executed a second time with unchanged source files. All three row counts remained the same, confirming that the successful rerun did not create additional records.

## Limitations and Improvements

The current pipeline runs manually and processes all source files again on each execution.

Possible improvements include incremental loading, automated scheduling, additional data validation and better handling of failures during loading.
