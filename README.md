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

## Setup

## Usage


## Verifying Simulation Results


## Architecture & Implementation


## Glossary

## Troubleshooting