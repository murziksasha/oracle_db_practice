# Local Oracle Database for SQL Practice

This repository provides a local Oracle Database (Oracle Database Free 23ai) running in Docker for the Udemy Oracle SQL course ([Oracle Database SQL Certified Associate 1Z0-071].

The database runs in a lightweight Docker container, and desktop tools such as Oracle SQL Developer connect to it from Windows.

---

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running on Windows.
- At least 4 GB of RAM allocated to Docker.
- [Oracle SQL Developer](https://www.oracle.com/database/sqldeveloper/technologies/download/) installed on Windows (or another database client such as DBeaver / VS Code Oracle extension).

---

## Quick Start

### 1. Start the Database

Open a terminal (PowerShell or Command Prompt) in this directory and run:

```bash
docker compose up -d
```

> **Note:** The initial image download (~1.5–2 GB) and first startup take a few minutes. Oracle needs time to initialize the container and run the HR schema setup script.

To check container startup logs:

```bash
docker compose logs -f
```

Wait until you see:

```text
DATABASE IS READY TO USE!
```

### 2. Stop the Database

To stop the container without losing your data:

```bash
docker compose down
```

> **Note on Data Persistence:** Data is stored in the Docker named volume `oracle-data`. Running `docker compose down` preserves all your tables and changes. If you want to reset everything back to a clean state, run `docker compose down -v`.

---

## Connecting with Oracle SQL Developer

Open Oracle SQL Developer on Windows, click the green **`+`** (New Connection) button, and enter the following settings:

### Course Practice Connection (`HR` user)

| Setting                 | Value                                                                          |
| ----------------------- | ------------------------------------------------------------------------------ |
| **Connection Name**     | `Oracle-Practice-HR` (or any name)                                             |
| **Connection Type**     | `Basic`                                                                        |
| **Authentication Type** | `Default`                                                                      |
| **Username**            | `HR`                                                                           |
| **Password**            | `hr`                                                                           |
| **Save Password**       | Checked                                                                        |
| **Hostname**            | `localhost`                                                                    |
| **Port**                | `1521`                                                                         |
| **Service Name**        | `FREEPDB1` _(Make sure "Service Name" radio button is selected, NOT "SID" XE)_ |

Click **Test** to verify connection (`Status: Success`), then click **Connect**.

### Admin Connection (`SYSTEM` user)

When a course lesson requires administrator / DBA privileges:

| Setting             | Value                    |
| ------------------- | ------------------------ |
| **Connection Name** | `Oracle-Practice-SYSTEM` |
| **Connection Type** | `Basic`                  |
| **Username**        | `SYSTEM`                 |
| **Password**        | `oracle`                 |
| **Save Password**   | Checked                  |
| **Hostname**        | `localhost`              |
| **Port**            | `1521`                   |
| **Service Name**    | `FREEPDB1`               |

---

## HR Sample Schema Verification

Once connected as `HR`, verify the tables and sample data by executing:

```sql
SELECT table_name FROM user_tables ORDER BY table_name;
```

Expected tables:

- `COUNTRIES`
- `DEPARTMENTS`
- `EMPLOYEES`
- `JOBS`
- `JOB_HISTORY`
- `LOCATIONS`
- `REGIONS`

Check row counts:

```sql
SELECT COUNT(*) FROM employees;   -- Expected: 107
SELECT COUNT(*) FROM departments; -- Expected: 27
```

You can also test queries from the command line inside Docker:

```bash
docker exec -it oracle-practice sqlplus hr/hr@localhost:1521/FREEPDB1
```
