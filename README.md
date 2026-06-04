Apex Ledger Ingestion System (ALIS)

1. System Overview

The Apex Ledger Ingestion System (ALIS) is a local Python data pipeline that automates the discovery, extraction, sanitisation, and relational storage of financial transaction records. The system uses a decoupled architecture to separate operating system file I/O from core data parsing logic.

Core Objectives

Automation: Automates directory scanning and file processing.

Data Quality: Normalises strings, dates, and currency values, and handles missing fields without crashing.

Relational Storage: Maps flat text records into structured relational tables inside a local SQLite database.

Observability: Utilises Python's built-in logging module to record system events, tracking errors and successes instead of using standard console prints.

2. System Architecture & Component Boundaries

ALIS separates file handling from parsing logic to adhere to the Single Responsibility Principle (SRP) and improve code testability.

  [ File System (I/O) ]        -->       [ Ingestion Logic ]       -->       [ Storage Layer ]
   (pathlib Operations)                   (Data Streams & Parsing)              (SQLite Database)
                                                    |
                                                    v
                                            [ System Logging ]


Component Responsibilities

main.py (The Driver): Handles physical file system operations. It scans the incoming directory for new files, opens file streams, routes them to the parser, and moves files to an archive directory once processed.

ingestion.py (The Parser): Contains the data transformation logic. It accepts raw text streams, parses lines, handles data formatting anomalies, and structures the records. It has no direct knowledge of the file system.

database.py (The Database Controller): Configures the SQLite database connection, establishes table schemas, validates relational constraints, and commits data.

3. Relational Schema & Data Dictionary

Accounts Table

Tracks individual financial accounts.

Column Name

Data Type

Key Type

Description / Constraints

account_id

Integer

Primary Key

Unique auto-incrementing identifier.

account_name

VarChar



Name of the account (e.g., "Everyday Savings").

account_type

VarChar



Classification constraint (e.g., "Debit", "Credit").

Transactions Table

An immutable ledger tracking individual financial entries.

Column Name

Data Type

Key Type

Description / Constraints

transaction_id

Integer

Primary Key

Unique auto-incrementing identifier.

account_id

Integer

Foreign Key

Links directly to Accounts.account_id.

date

Date/Text



Standardised ISO 8601 temporal string (YYYY-MM-DD).

description

VarChar



Cleansed merchant or transaction text payload.

amount

Float/Real



Signed decimal numeric value representing the transaction value.

category

VarChar



Functional expenditure grouping (e.g., "Groceries").

4. Execution Lifecycle

The system moves linearly through four distinct operational states:

Discovery: The system initialises, scans the data/raw/ directory via pathlib, and identifies target CSV files.

Parsing: The active file stream is processed line-by-line. Formatting anomalies or missing attributes are caught here and routed to the application log.

Persistence: Structured data objects are validated against the database schema constraints and committed to the SQLite database inside a single database transaction.

Archival: The file stream is closed, and the physical CSV file is moved to data/processed/ to prevent duplicate processing loops.

5. References & Standards Applied

Split Phase Pattern: Implemented by separating the code into an I/O driver phase and a clean parsing phase (Refactoring, Martin Fowler).[^1]

Hexagonal Architecture Concepts: Isolating the core logic from direct dependencies on external factors like physical hard drive formats.[^2]

ISO 8601: Enforcing standard temporal markers (YYYY-MM-DD) for relational query optimisation.[^3]

[^1]: Fowler, M. (2018). Refactoring: Improving the Design of Existing Code (2nd ed.). Addison-Wesley.
[^2]: Cockburn, A. (2005). Hexagonal Architecture (Ports and Adapters). Alistair.Cockburn.us.
[^3]: International Organization for Standardization. (2019). ISO 8601-1:2019 Date and time — Representations for information interchange.
