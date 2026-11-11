# SENTINEL Fraud Analytics -- MongoDB Atlas with PyMongo

A Python + MongoDB implementation of the SENTINEL data-modeling and
querying assignment. The notebook loads customer profiles, embeds device
details inside customer documents, stores transaction records in a
separate collection, demonstrates CRUD operations, creates indexes,
compares query execution plans, and runs aggregation pipelines.

## Project structure

``` text
.
├── crudOperations.ipynb
├── README.md
├── .env                 # Local secrets; do not commit
├── .env.example         # Safe environment-variable template
└── .gitignore
```

The SENTINEL dataset is read from the local data directory configured in
the notebook. The notebook expects a `manifest.json` file and gzipped
CSV shards under the configured `DATA_DIR`.

## Technology stack

-   Python
-   MongoDB Atlas
-   PyMongo
-   Jupyter Notebook
-   SENTINEL Project Work 4 and 5 dataset

## MongoDB Atlas setup

1.  Create a free MongoDB Atlas account at
    https://www.mongodb.com/atlas.
2.  Create a free shared cluster.
3.  Under **Database Access**, create a database user with read/write
    access to the project database.
4.  Under **Network Access**, add the IP address from which the notebook
    will connect. For better security, allow only your current IP
    instead of opening access to everyone.
5.  Select **Connect → Drivers → Python** and copy the connection
    string.
6.  Replace the username and password placeholders with your database
    user's credentials. URL-encode special characters in the password
    when required.

Create a local `.env` file:

``` env
MONGODB_URI=mongodb+srv://<username>:<password>@<cluster-host>/?retryWrites=true&w=majority
MONGODB_DATABASE=sentinel_db
```

Create `.env.example` and commit only this safe template:

``` env
MONGODB_URI=mongodb+srv://<username>:<password>@<cluster-host>/?retryWrites=true&w=majority
MONGODB_DATABASE=sentinel_db
```

**Security:** Never commit the real Atlas URI, password, API keys, or
other credentials. If a real password has ever been committed or shared,
rotate it in Atlas.

## Installation

Create and activate a virtual environment (recommended), then install
the dependencies:

``` bash
python -m venv .venv
```

Windows:

``` bash
.venv\Scripts\activate
```

macOS/Linux:

``` bash
source .venv/bin/activate
```

Install packages:

``` bash
pip install pymongo dnspython python-dotenv pandas jupyter
```

Open the notebook:

``` bash
jupyter notebook crudOperations.ipynb
```

## Configuration

Update the notebook to load credentials from `.env` rather than
hardcoding a connection string:

``` python
import os
from dotenv import load_dotenv
from pymongo import MongoClient

load_dotenv()

mongo_uri = os.getenv("MONGODB_URI")
database_name = os.getenv("MONGODB_DATABASE", "sentinel_db")

if not mongo_uri:
    raise ValueError("MONGODB_URI is missing. Add it to your .env file.")

client = MongoClient(mongo_uri, serverSelectionTimeoutMS=10000)
client.admin.command("ping")

db = client[database_name]
print("Successfully connected to MongoDB Atlas!")
```

Set the dataset directory in the notebook to the location of your
SENTINEL data:

``` python
from pathlib import Path

DATA_DIR = Path(r"<your-local-path-to-supabase_db_kit-data>")
MANIFEST_PATH = DATA_DIR / "manifest.json"
```

The notebook reads the table/shard paths from `manifest.json`. Make sure
the manifest and its referenced `.csv.gz` files are present.

## Data model and design decision

The notebook creates two collections: `customers` and `transactions`.

### 1. `customers` collection

One document represents one customer. Device details are embedded in a
`devices` array.

Example document:

``` json
{
  "customer_id": "C_example",
  "customer_age": 28,
  "home_city": "Indore",
  "region": "central",
  "is_synthetic": true,
  "devices": [
    {
      "device_id": "D_example",
      "os": "Android",
      "device_type": "mobile"
    }
  ]
}
```

### 2. `transactions` collection

One document represents one transaction. Customer, merchant, and device
identifiers are stored with transaction attributes and merchant
information.

Example document:

``` json
{
  "transaction_id": "TX_example",
  "customer_id": "C_example",
  "merchant_id": "M_example",
  "device_id": "D_example",
  "amount": 1500.50,
  "time": "2026-10-31T12:00:00.000+00:00",
  "payment_channel": "UPI",
  "merchant_category": "electronics",
  "merchant_location": "example-location",
  "is_fraud": false
}
```

### Why embed devices but keep transactions separate?

-   **Devices are embedded in customers** because device information is
    useful as part of a customer's profile and can be retrieved with one
    customer-document read. This avoids a separate lookup for the
    customer's device list.
-   **Transactions are stored separately** because a customer can have
    many transactions and transaction records grow continuously. Keeping
    them separate avoids an unbounded transactions array in customer
    documents and supports independent filtering, indexing, and
    aggregation.
-   **Customer, merchant, and device IDs are retained in transaction
    documents** so transactions can be queried and grouped without
    embedding full copies of every related entity.

The notebook also builds an in-memory merchant lookup from the SENTINEL
merchant table to add merchant category and location to transaction
documents.

## Data ingestion

The notebook:

1.  Reads `manifest.json` to identify the SENTINEL table shards.
2.  Loads customer records into `customers`.
3.  Reads device records and embeds device details into the matching
    customer documents.
4.  Loads merchant information into a lookup dictionary.
5.  Loads a sample of **5,000 transactions** into `transactions`,
    including customer, merchant, and device IDs and merchant fields.
6.  Uses batch inserts (`insert_many`) and bulk updates (`bulk_write`)
    to process records efficiently.

The notebook drops the `customers` and `transactions` collections at the
start of its ingestion section to create a clean run. **This deletes the
existing data in those two collections.** Run the ingestion cells only
when you are comfortable rebuilding them.

## CRUD operations demonstrated

The notebook demonstrates the required PyMongo operations:

  ------------------------------------------------------------------------------
  Operation               Example in notebook            Purpose
  ----------------------- ------------------------------ -----------------------
  Insert one              `customers.insert_one()`       Add one test customer

  Insert many             `transactions.insert_many()`   Add multiple test
                                                         transactions

  Find with filter and    `transactions.find()`          Return transactions
  projection                                             where amount is greater
                                                         than 500, selecting
                                                         only specified fields

  Update many             `transactions.update_many()`   Set `risk_review_flag`
                                                         for matching
                                                         electronics
                                                         transactions below the
                                                         configured amount

  Delete one              `transactions.delete_one()`    Delete the test
                                                         transaction with ID
                                                         `TX_TEST_002`
  ------------------------------------------------------------------------------

The CRUD examples use test records so that the operations can be
demonstrated without modifying the original SENTINEL sample records
intentionally.

## Indexing and query explain

The notebook creates these indexes on `transactions`:

``` python
db.transactions.create_index([("customer_id", 1)])
db.transactions.create_index([("time", -1)])
```

It then compares query execution information for a customer transaction
query:

-   **Before:** uses a `$natural` hint to force a collection scan
    (`COLLSCAN`).
-   **After:** allows MongoDB to select an index for the query and sort.

The notebook prints the number of documents examined, execution time,
and winning execution stage from `explain()`.

The before/after comparison is a demonstration. With only 5,000 sample
records, execution-time differences can be small or inconsistent. The
selected winning plan and documents examined are more useful evidence
than timing alone.

## Aggregation pipelines

### 1. Fraud rate per merchant category

The first pipeline groups transactions by `merchant_category`,
calculates the total transaction count and fraud count, computes the
fraud percentage, and sorts categories by fraud rate in descending
order.

Formula:

``` text
fraud rate (%) = (fraud count / total transactions) * 100
```

**Data-quality note:** In the current ingestion cell, every transaction
is assigned `is_fraud: False` as a default. Therefore, the current
aggregation will report a zero fraud count/rate unless the notebook is
updated to join or map the actual SENTINEL labels to transactions. Do
not interpret those zero rates as a real fraud-analysis result.

### 2. Top 10 customers by amount in their last 24 hours of activity

The second pipeline:

1.  Converts the transaction time string to a MongoDB date.
2.  Finds each customer's latest transaction timestamp.
3.  Uses `$lookup` to retrieve that customer's transactions within the
    24 hours preceding that latest timestamp.
4.  Calculates the total amount and transaction count in that window.
5.  Sorts by total amount and returns the top 10 customers.

**Window definition:** This implementation uses a separate 24-hour
window anchored to each customer's latest transaction. It does not use
the current wall-clock time as a shared cutoff for all customers. The
README and result description should retain this distinction.

## Run order

Run the notebook cells from top to bottom:

1.  Connect to MongoDB Atlas.
2.  Load the manifest and helper functions.
3.  Load customers and embed device records.
4.  Build the merchant lookup.
5.  Load 5,000 transactions.
6.  Run the device embedding/update cells as needed.
7.  Run the CRUD examples.
8.  Create indexes.
9.  Run explain before and after indexing.
10. Run both aggregation pipelines.

If you rerun the notebook, remember that the ingestion section drops the
two collections before reloading them. The later CRUD cells also insert
test records.

## Screenshots to include in the GitHub repository

Create a `screenshots/` folder and add screenshots from your own
successful notebook run. Suggested evidence:

-   `01-atlas-connection.png` -- successful MongoDB Atlas ping.
-   `02-data-loaded.png` -- customer and transaction counts, showing at
    least 5,000 transactions.
-   `03-crud-operations.png` -- insert, find, update, and delete output.
-   `04-indexes.png` -- created indexes.
-   `05-explain-before-after.png` -- explain statistics before and after
    indexing.
-   `06-fraud-rate.png` -- fraud-rate aggregation output after actual
    fraud labels are mapped.
-   `07-top-customers.png` -- top 10 customers aggregation output.

Only include screenshots that reflect actual output from your run. Do
not fabricate counts, query timings, or successful results.

## Important validation before submission

-   Confirm `db.transactions.count_documents({}) >= 5000` after
    ingestion.
-   Confirm that transaction documents contain `customer_id`,
    `merchant_id`, and `device_id`.
-   Map the actual SENTINEL fraud labels to `is_fraud` before claiming
    the fraud-rate pipeline measures fraud.
-   Confirm the `time` values parse correctly with `$toDate`.
-   Check that the test transaction is deleted and that the update
    operation modifies the intended records.
-   Ensure `.env` is excluded from Git.

Suggested `.gitignore`:

``` gitignore
.env
.venv/
__pycache__/
.ipynb_checkpoints/
*.pyc
```

## Submission

Push the notebook, this README, `.env.example`, and your genuine output
screenshots to a **public GitHub repository**. Do not push `.env`,
database passwords, or private dataset files unless you have permission
to share them.

------------------------------------------------------------------------

**Project:** SENTINEL -- MongoDB with Python using PyMongo\
**Database:** `sentinel_db`\
**Collections:** `customers`, `transactions`
