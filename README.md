# Chinook Sales Simulator

A synthetic sales data generator for the Chinook database, designed to simulate realistic daily transactions for data engineering practice and portfolio projects.

This project is fully cross-platform and will run on Windows, macOS, and Linux.

## What is This Project?

This project simulates a **real-world online sales system** that generates synthetic transaction data for practice and learning.

### Use Case

Perfect for **building data engineering portfolios** without needing access to real production databases. Great for:
- Learning data pipeline development (ETL/ELT)
- Testing data warehouse architectures
- Practicing with realistic transactional data
- Building end-to-end data engineering projects

### Key Concept: D-1 Batch Processing

**D-1** means "Day minus 1" — processing yesterday's data today.

This is a common pattern in production data engineering:
- **Today is November 1st** → Script generates sales for **October 31st**
- **Today is November 2nd** → Script generates sales for **November 1st**

**Why D-1?** In real companies, sales data is typically processed in overnight batches. This simulator replicates that realistic workflow, making your portfolio projects more authentic.

### How It Fits Into a Data Pipeline

```
┌─────────────────┐         ┌──────────────┐         ┌─────────────────┐
│  This Simulator │  D-1    │  PostgreSQL  │  Batch  │   Data Pipeline │
│    (Python)     │─Sales──>│    (Neon)    │─Extract>│   (Databricks,  │
│                 │         │              │         │   Airflow, etc) │
└─────────────────┘         └──────────────┘         └─────────────────┘
                                                               │
                                                               ▼
                                                      ┌─────────────────┐
                                                      │   Data Lakehouse│
                                                      │  (Bronze/Silver/│
                                                      │       Gold)     │
                                                      └─────────────────┘
```

This project is the **data source** — the starting point for building complete data engineering solutions.

## Technical Features

### Data Generation

- **Configurable batch operations**: Generates user-specified volumes of inserts, updates, and deletes
- R**ealistic transaction simulation**:

    - **Inserts**: Multi-item sales (1-5 tracks) with random customer and product selection, timestamped in D-1
    - **Updates**: Adds additional items to random invoices from last 90 days while recalculating 
    - **Deletes**: Complete invoice cancellations from last 90 days with cascading line item removal

- **90-day modification window**: Updates and deletes operate on recent invoices only, simulating real-world return and cancellation policies
- **Temporal distribution**: Random timestamp generation across 24-hour period for new sales maintains realistic data patterns
- **Late-arriving changes**: Historical modifications within 90-day window create realistic data pipeline challenges

### Database Architecture

- **ACID compliance**: All operations execute within a single atomic transaction (commit or rollback)
- **Concurrent-safe ID generation**: PostgreSQL SEQUENCE objects prevent ID collisions in multi-process scenarios
- **Referential integrity**: Foreign key constraints enforced across Invoice, InvoiceLine, Customer, and Track tables
- **Idempotent setup**: Database initialization scripts safely re-runnable without data corruption

### Engineering Practices

- **Secure credential management**: Database connections via neonctl CLI, zero hardcoded passwords
- **Structured logging**: Configurable log levels (DEBUG/INFO/WARNING/ERROR) with ISO timestamps
- **Database state validation**: Pre-execution checks verify existence of required functions and sequences
Performance optimization: Batch processing with progress tracking and summary statistics
- **Cross-platform compatibility**: Runs on Windows, macOS, and Linux without modifications

## Prerequisites - First Time Setup

**If this is your first time**, you'll need to install some tools and set up your Neon database. Follow the sections below before running the simulator.

### Required Tools

#### 1. Python 3.11+

**What it is**: The programming language this project is written in.

**Check if installed**:

```bash
python --version  # or python3 --version
```

**Install if needed**: [python.org/downloads](https://www.python.org/downloads/)

#### 2. uv (Python Package Manager)

**What it is**: A modern, fast Python package manager (faster and more reliable than pip).

**Why uv instead of pip?**: Better dependency resolution, faster installs, and built-in virtual environment management.

**Install**:

```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**Verify**:

```bash
uv --version
```

**Learn more**: [docs.astral.sh/uv](https://docs.astral.sh/uv/)

#### 3. Node.js 18+ (Required for neonctl)

**What it is**: JavaScript runtime needed to run the Neon CLI tool.

**Check if installed**:

```bash
node --version
```

**Install if needed**: [nodejs.org](https://nodejs.org/en) (Download LTS version)

#### 4. Neon CLI (neonctl)

**What it is**: Command-line tool to interact with your Neon database from the terminal.

**Install via npm**:

```bash
npm i -g neonctl
```

**Verify**:

```bash
neonctl --version
```

**Learn more**: [Neon CLI Docs](https://neon.com/docs/cli)

#### 5. PostgreSQL Client (psql)

**What it is**: Command-line tool to connect to PostgreSQL databases (needed to import Chinook data).

**Check if installed**:

```bash
psql --version
```

**Install if needed**:
- **macOS**: `brew install postgresql` (or download from [postgresql.org](https://www.postgresql.org/download/))
- **Linux**: `sudo apt install postgresql-client` (Debian/Ubuntu) or equivalent
- **Windows**: Download from [postgresql.org](https://www.postgresql.org/download/)

**Note**: You only need the **client** (`psql`), not the full PostgreSQL server.

## Setting Up Your Neon Database

**If you already have a Neon account with the Chinook database**, skip to the [Setup](#setup) section below.

### Step 1: Create a Neon Account and Project

1. Go to [neon.com](https://neon.com/) and click **Sign Up** (free tier available)
2. Sign up with GitHub, Google, or email
3. Once logged in, click **New Project**
4. Configure your project:

    - **Name**: "chinook-simulator" (or any name you prefer)
    - **Region**: Choose the one closest to you
    - **PostgreSQL version**: 16 (recommended, or latest available)

5. Click **Create Project**

Your first project is created with a default database called `neondb`.

### Step 2: Choose Your Database Name

**Important decision**: You need to choose a name for the database where Chinook data will be stored.

**Recommended name**: `chinook_db` (we'll use this throughout the guide)

**Why it matters**: You'll need to use this EXACT name in:
- The database creation command below
- The data import command
- Your `.env` configuration file (later in Setup)

**Note**: You can choose any name you prefer (e.g., `chinook`, `music_store`, `sales_db`), just be consistent everywhere.

### Step 3: Import the Chinook Sample Database

The Chinook database contains realistic data for a digital media store (artists, albums, tracks, customers, invoices).

**Get Your Connection String**
1. In your Neon project dashboard, click the **Connect** button
2. Copy the connection string (it looks like: `postgresql://user:password@host/neondb`)
3. Keep this handy — you'll need it for the next commands

**Understanding Connection Strings (Important!)**

Your Neon connection string has this format:

```
postgresql://username:password@ep-cool-name-123456.us-east-2.aws.neon.tech/neondb
                                                                           ^^^^^^^
                                                                      database name
```

To connect to a different database, **replace the database name** at the end:

```
postgresql://username:password@ep-cool-name-123456.us-east-2.aws.neon.tech/chinook_db
                                                                           ^^^^^^^^^^
```

**Import via Command Line**

```bash
# 1. Create the chinook_db database (using your original connection string with /neondb)
psql "postgresql://username:password@host.neon.tech/neondb" -c "CREATE DATABASE chinook_db;"

# 2. Download the official Chinook SQL file
wget https://raw.githubusercontent.com/neondatabase/postgres-sample-dbs/main/chinook.sql

# 3. Import the data (replace /neondb with /chinook_db in your connection string)
psql "postgresql://username:password@host.neon.tech/chinook_db" -f chinook.sql

# 4. Verify the import (should return 412)
psql "postgresql://username:password@host.neon.tech/chinook_db" -c 'SELECT COUNT(*) FROM public."Invoice";'
```

**Remember**: Replace `username:password@host.neon.tech` with your actual Neon credentials.

**Expected output**: `412` (number of invoices in the sample data)

**Troubleshooting**:
- If `wget` is not installed, download the file manually from the URL in step 2
- On Windows without `wget`, use: `curl -O https://raw.githubusercontent.com/neondatabase/postgres-sample-dbs/main/chinook.sql`

### Step 4: Get Your Neon Credentials

You'll need three values to configure this project:
- NEON_ORG_ID: Your organization ID
- NEON_PROJECT_ID: Your project ID
- NEON_DATABASE: The database name you chose in Step 2 (e.g., `chinook_db`)

**Finding Your Organization ID (NEON_ORG_ID)**

**Via Neon Console**:
1. In the Neon Console, click on your organization name (top-left corner)
2. Go to **Settings** (in the organization menu)
3. Look under **General information**
4. Copy the **Organization ID** (format: `org-word-word-12345678`)

**Via CLI** (alternative method):

```bash
neonctl orgs list
# Your org ID will be shown in the output
```

**Finding Your Project ID (NEON_PROJECT_ID)**

**Via Neon Console**:
1. Open your project in the Neon Console
2. Click **Settings** (in the left sidebar)
3. Under **General**, find and copy the **Project ID**

**Via CLI** (alternative method):

```bash
neonctl projects list
# Your project ID will be shown in the output
```

**Save these IDs** — you'll need them in the next step!

## Setup

**Prerequisites**: Make sure you've completed the [Prerequisites](#prerequisites---first-time-setup) and [Setting Up Your Neon Database](#setting-up-your-neon-database) sections above.

1. **Clone the repository**:

    ```bash
    git clone https://github.com/ppkhai2612/chinook-sales-simulator.git
    cd chinook-sales-simulator
    ```

2. **Create a `.env` file**: Copy the example file and fill in your Neon credentials (from [Step 4](#step-4-get-your-neon-credentials) above):

    ```bash
    cp .env.example .env
    # Windows (CMD) users: use 'copy' instead of 'cp'
    ```
    
    Edit `.env` and add the credentials you found earlier:

    ```bash
    NEON_ORG_ID=org-your-actual-org-id
    NEON_PROJECT_ID=your-actual-project-id
    NEON_DATABASE=chinook_db
    ```

    **Critical**: The `NEON_DATABASE` value MUST match the database name you created in [Step 2](#step-2-choose-your-database-name). If you used a different name than `chinook_db`, use that name here.

    Optional parameters you can configure:

    ```bash
    # Optional: specify database role (default: project default)
    # NEON_ROLE=your_role_name

    # Optional: specify branch (default: main)
    # NEON_BRANCH=main

    # Optional: logging level (default: INFO)
    # LOG_LEVEL=DEBUG
    ```

3. **Install dependencies**:

    ```bash
    uv sync
    ```

4. **Set up the database**: Run the automated setup command. This will:

    - Validate your configuration and test connectivity
    - Update historical invoice data to align with D-1 workflow
    - Create all required simulation functions (`INSERT`, `UPDATE`, `DELETE`) and sequences

    ```bash
    uv run src/main.py setup
    ```

    The setup is **idempotent** and safe to run multiple times.

## Usage

All commands are run via the `src/main.py` script. The simulator supports generating a mix of inserts, updates, and deletes in a single execution.

The script prompts for three space-separated integers:
1. Number of **new sales** (Inserts)
2. Number of **sale updates** (Updates)
3. Number of **sale cancellations** (Deletes)

### Interactive Mode (Recommended)

Run the simulator and wait for it to prompt you for input.

```bash
uv run src/main.py simulate
```

You will see a prompt like this. Enter three numbers and press Enter:

```bash
Enter the number of INSERTS, UPDATES, and DELETES for D-1 (e.g., '100 5 2'): 100 5 2
```

### Non-Interactive Mode

To pass the numbers directly (useful for automation), you can echo a string with the three values and pipe it to the command.

```bash
# Format: "INSERTS UPDATES DELETES"
echo "100 5 2" | uv run src/main.py simulate
```

### Example Output

```bash
Enter the number of INSERTS, UPDATES, and DELETES for D-1 (e.g., '100 5 2'): 100 5 2
2026-10-21 10:40:21 - d1_sales_simulator - INFO - === D-1 Sales Simulator Starting ===
2026-10-21 10:40:22 - d1_sales_simulator - INFO - Database connection established
2026-10-21 10:40:22 - d1_sales_simulator - INFO - Validating database state...
2026-10-21 10:40:22 - d1_sales_simulator - INFO - Database state validation successful
2026-10-21 10:40:22 - d1_sales_simulator - INFO - Starting batch operation in a SINGLE TRANSACTION...
2026-10-21 10:40:22 - d1_sales_simulator - INFO - Starting batch operation for D-1 (2026-10-20)
2026-10-21 10:40:22 - d1_sales_simulator - INFO - Requested: 100 Inserts, 5 Updates, 2 Deletes
2026-10-21 10:40:22 - d1_sales_simulator - INFO - Processing 2 DELETES...
2026-10-21 10:40:22 - d1_sales_simulator - INFO - Progress: 1/2 deletes executed for InvoiceId 394.
2026-10-21 10:40:22 - d1_sales_simulator - INFO - Progress: 2/2 deletes executed for InvoiceId 408.
2026-10-21 10:40:22 - d1_sales_simulator - INFO - Processing 5 UPDATES...
2026-10-21 10:40:22 - d1_sales_simulator - INFO - Progress: 1/5 updates executed for InvoiceId 395.
2026-10-21 10:40:22 - d1_sales_simulator - INFO - Progress: 2/5 updates executed for InvoiceId 412.
2026-10-21 10:40:22 - d1_sales_simulator - INFO - Progress: 3/5 updates executed for InvoiceId 402.
2026-10-21 10:40:22 - d1_sales_simulator - INFO - Progress: 4/5 updates executed for InvoiceId 410.
2026-10-21 10:40:22 - d1_sales_simulator - INFO - Progress: 5/5 updates executed for InvoiceId 401.
2026-10-21 10:40:22 - d1_sales_simulator - INFO - Processing 100 INSERTS...
2026-10-21 10:40:23 - d1_sales_simulator - INFO - Progress: 10/100 (10%) - Pending InvoiceId=422, Total=$1.98
2026-10-21 10:40:23 - d1_sales_simulator - INFO - Progress: 20/100 (20%) - Pending InvoiceId=432, Total=$4.95
2026-10-21 10:40:23 - d1_sales_simulator - INFO - Progress: 30/100 (30%) - Pending InvoiceId=442, Total=$2.98
2026-10-21 10:40:24 - d1_sales_simulator - INFO - Progress: 40/100 (40%) - Pending InvoiceId=452, Total=$2.97
2026-10-21 10:40:24 - d1_sales_simulator - INFO - Progress: 50/100 (50%) - Pending InvoiceId=462, Total=$2.97
2026-10-21 10:40:25 - d1_sales_simulator - INFO - Progress: 60/100 (60%) - Pending InvoiceId=472, Total=$2.97
2026-10-21 10:40:25 - d1_sales_simulator - INFO - Progress: 70/100 (70%) - Pending InvoiceId=482, Total=$4.95
2026-10-21 10:40:26 - d1_sales_simulator - INFO - Progress: 80/100 (80%) - Pending InvoiceId=492, Total=$1.98
2026-10-21 10:40:26 - d1_sales_simulator - INFO - Progress: 90/100 (90%) - Pending InvoiceId=502, Total=$3.96
2026-10-21 10:40:26 - d1_sales_simulator - INFO - Progress: 100/100 (100%) - Pending InvoiceId=512, Total=$2.98
2026-10-21 10:40:26 - d1_sales_simulator - INFO - Batch successfully prepared. Committing transaction...
2026-10-21 10:40:26 - d1_sales_simulator - INFO - SUCCESS: All operations committed to the database.
2026-10-21 10:40:26 - d1_sales_simulator - INFO - Simulation log saved to: /home/khai2612/chinook-sales-simulator/simulation_logs/simulation_2026-10-21_03-40-21.toml
2026-10-21 10:40:26 - d1_sales_simulator - INFO - Summary: 100 new sales inserted, 5 updated, 2 deleted.
2026-10-21 10:40:26 - d1_sales_simulator - INFO - Total new revenue: $311.06
2026-10-21 10:40:26 - d1_sales_simulator - INFO - Average new sale: $3.11
```

## Verifying Simulation Results

After running a simulation, a detailed log file in TOML format is created in the `simulation_logs/` directory. This log contains a full record of every operation performed (inserts, updates, and deletes).

You can verify the integrity of the simulation by running the `verify_simulation.py` script. This script automatically:
1. Finds the most recent simulation log.
2. Reads the summary and every individual operation from the log file.
3. Connects to the database and queries it to ensure that the state of the data perfectly matches what was recorded in the log.
4. Reports on its findings, confirming that every insert, update, and delete was successfully and correctly applied.

```bash
uv run src/verify_simulation.py
```

### Example Verification Output

```bash
2026-10-21 11:34:45 - __main__ - INFO - === Starting Log-Based Simulation Verification ===
2026-10-21 11:34:45 - __main__ - INFO - Found latest log file: simulation_2026-10-21_03-40-21.toml
2026-10-21 11:34:45 - __main__ - INFO - Verifying 107 operations for D-1: 2026-10-20
2026-10-21 11:34:51 - __main__ - INFO - --- Verifying 100 Inserts ---
2026-10-21 11:34:57 - __main__ - INFO - --- Verifying 2 Deletes ---
2026-10-21 11:34:57 - __main__ - INFO - --- Verifying 5 Updates ---
2026-10-21 11:34:57 - __main__ - INFO - --- Verification Summary ---
2026-10-21 11:34:57 - __main__ - INFO - ✅ SUCCESS: All 107 verified operations are consistent with the log.
```

## Architecture & Implementation

### Project Structure

```bash
chinook-sales-simulator
├── .env                              # Environment variables (gitignored)
├── .env.example                      # Configuration template
├── .gitignore                        # .gitignore file
├── pyproject.toml                    # Python dependencies & metadata
├── README.md                         # Documentation
├── simulation_logs/                  # Output directory for TOML log files
├── sql/                              # Database schema layer
│   ├── simulation_functions.sql      # PL/pgSQL stored procedures (all DML)
│   └── update_historical_data.sql    # Historical data alignment
├── src/                              # Python application layer
│   ├── d1_sales_simulator.py         # Core simulation engine
│   ├── main.py                       # CLI interface and orchestration
│   └── verify_simulation.py          # Log-based data validation utility
```

### Design Decisions

#### Separation of Concerns

- **Python layer**: Orchestration, CLI interface, connection management, and transaction control
- **PostgreSQL layer**: Business logic encapsulated in stored procedures for performance and atomicity

#### Why PL/pgSQL Functions?

- **Performance**: Minimizes network round-trips; all data manipulation occurs server-side
- **Atomicity**: Database enforces transactional guarantees at the storage layer
- **Concurrent safety**: SEQUENCE objects provide lock-free ID generation
- **Maintainability**: Business logic versioned alongside schema in SQL files

#### Technologies Choices

- **PostgreSQL 16**: Advanced ACID compliance, mature MVCC, robust constraint enforcement
- **Neon**: Serverless PostgreSQL with connection pooling and auto-scaling capabilities
- **Python 3.11+**: Type hints, improved error messages, performance enhancements
- **uv package manager**: 10-100x faster than pip with deterministic dependency resolution

## Glossary

Quick reference for technical terms used in this project:

| **Term** | **Definition** |
|-|-|
| **D-1** | "Day minus 1" — yesterday. In data engineering, processing yesterday's data today (common batch pattern). |
| **OLTP** | Online Transaction Processing — database optimized for day-to-day operations (inserts, updates). Opposite of OLAP (analytics). |
| **ETL/ELT** | Extract, Transform, Load / Extract, Load, Transform — processes for moving data between systems. |
| **Syntheic Data** | Artificially generated data that mimics real data patterns, used for testing and development. |
| **Idempotent** | Safe to run multiple times without unintended side effects. Running the same operation twice produces the same result. |
| **Batch Processing** | Processing data in groups (batches) rather than one record at a time. Common in overnight data pipelines. |
| **Sequence** | PostgreSQL feature for generating unique, sequential IDs safely in concurrent environments. |
| **Lakehouse** | Modern data architecture combining data lake (storage) and data warehouse (analytics) features. |
| **Medallion Architecture** | Data organization pattern with Bronze (raw), Silver (cleaned), and Gold (aggregated) layers. |

## Troubleshooting

**"Function 'simulate_new_sale' not found" (or update/delete function)**
- Your database setup is likely incomplete or outdated. Run the automated setup again: `uv run src/main.py setup`

**"neonctl not found"**
- Install Neon CLI: `npm i -g neonctl`
- See [Prerequisites](#prerequisites---first-time-setup) section for details

**"NEON_ORG_ID and NEON_PROJECT_ID must be set"**
- Create .env file from .env.example and add your credentials
- See [Setting Up Your Neon Database](#setting-up-your-neon-database) section for how to find these IDs

**"psql: command not found"**
- Install PostgreSQL client (see [Prerequisites](#prerequisites---first-time-setup))
- You only need the client, not the full PostgreSQL server

**Need to reset historical data?**
- The setup command is idempotent. If you need to re-align historical dates, you can run it again: `uv run src/main.py setup`

**Want more detailed logs?**
- Add `LOG_LEVEL=DEBUG` to your `.env` file

**Import failed with "database chinook already exists"**
- That's okay! It means you already imported it. Skip to the verification step.

**Verification shows wrong number of invoices**
- Expected: 412 invoices
- If different, try dropping and recreating: `psql "<connection-string>" -c "DROP DATABASE chinook;"` then repeat [Step 2 of Setting Up Your Neon Database](#step-2-choose-your-database-name)