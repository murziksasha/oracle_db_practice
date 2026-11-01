# Local Oracle Database for SQL Practice

This repository provides a local Oracle Database (Oracle Database Free 23ai) running in Docker for the Udemy Oracle SQL course ([SQL для начинающих: с нуля до сертификата Oracle](https://www.udemy.com/course/sql-oracle-certification/), Oracle Database SQL Certified Associate 1Z0-071).

The database runs in a lightweight Docker container, and desktop tools such as Oracle SQL Developer connect to it from Windows.

---

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running on Windows.
- At least 4 GB of RAM allocated to Docker.
- [Oracle SQL Developer](https://www.oracle.com/database/sqldeveloper/technologies/download/) installed on Windows (or another database client such as DBeaver / VS Code Oracle extension).
- [Node.js](https://nodejs.org/) (optional, if you want to use npm shortcut scripts).

---

## Quick Start (NPM Scripts)

You can manage the Oracle container easily via `npm`:

| Action | NPM Shortcut | Docker Compose Equivalent |
|---|---|---|
| **Start Database** | `npm start` | `docker compose up -d` |
| **View Logs** | `npm run logs` | `docker compose logs -f` |
| **Check Status** | `npm run status` | `docker compose ps` |
| **Stop Database** | `npm stop` | `docker compose down` |
| **Restart Database** | `npm restart` | `docker compose restart` |
| **Connect SQL*Plus (HR)** | `npm run sql` | `docker compose exec -it oracle sqlplus hr/hr@localhost:1521/FREEPDB1` |
| **Connect SQL*Plus (SYS)** | `npm run sql:sys` | `docker compose exec -it oracle sqlplus sys/oracle@localhost:1521/FREEPDB1 as sysdba` |
| **Container Shell** | `npm run bash` | `docker compose exec -it oracle bash` |
| **Full Reset (Clean Data)** | `npm run clean` | `docker compose down -v` |

---

## Quick Start (Docker Compose CLI)

### 1. Start the Database

Open a terminal (PowerShell or Command Prompt) in this directory and run:

```bash
npm start
# or: docker compose up -d
```

> **Note:** The initial image download (~1.5-2 GB) and first startup take a few minutes. Oracle needs time to initialize the container and run the HR schema setup script.

To check container startup logs:

```bash
npm run logs
# or: docker compose logs -f
```

Wait until you see:

```text
DATABASE IS READY TO USE!
```

### 2. Stop the Database

To stop the container without losing your data:

```bash
npm stop
# or: docker compose down
```

> **Note on Data Persistence:** Data is stored in the Docker named volume `oracle-data`. Running `npm stop` preserves all your tables and changes. If you want to reset everything back to a clean state, run `npm run clean` (or `docker compose down -v`).

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
npm run sql
# or: docker exec -it oracle-practice sqlplus hr/hr@localhost:1521/FREEPDB1
```

---

## Credentials Summary

| User     | Password | Service Name | Purpose                                   |
| -------- | -------- | ------------ | ----------------------------------------- |
| `HR`     | `hr`     | `FREEPDB1`   | Course practice (owns `EMPLOYEES` etc.)   |
| `SYSTEM` | `oracle` | `FREEPDB1`   | Admin / DBA tasks                         |
| `SYS`    | `oracle` | `FREEPDB1`   | Superuser, connect `AS SYSDBA` only       |

All values come from `docker-compose.yml` (`ORACLE_PASSWORD`, `APP_USER`, `APP_USER_PASSWORD`). Host `localhost`, port `1521`.

> These are local practice credentials only. Never reuse them on a real server.

> **Tip:** If you connect as `SYSTEM` you will see hundreds of Oracle internal tables (`AQ$_...`, `MVIEW$_...`) and no `EMPLOYEES`. That is expected. The course tables live in the `HR` schema, so use the `HR` connection.

---

## Installing the HR Schema Manually

The scripts in `init/` run **only on the very first startup of an empty `oracle-data` volume**. If the volume already existed (or the container was created another way), the `HR` user exists but has no tables. Check with:

```sql
-- connected as HR
SELECT COUNT(*) FROM user_tables;   -- 0 means the schema is not installed
```

### Option A: Reset and let init run (deletes all your data)

```bash
npm run clean   # docker compose down -v
npm start
npm run logs    # wait for DATABASE IS READY TO USE!
```

### Option B: Install into the running container (keeps other data)

Uses the official [Oracle sample schemas](https://github.com/oracle-samples/db-sample-schemas) (PowerShell):

```powershell
cd $env:TEMP
Invoke-WebRequest https://github.com/oracle-samples/db-sample-schemas/archive/refs/tags/v23.3.zip -OutFile dss.zip
Expand-Archive -Force dss.zip dss
docker cp "dss\db-sample-schemas-23.3\human_resources" oracle-practice:/tmp/hr

docker exec oracle-practice bash -c "cd /tmp/hr && printf 'SET DEFINE OFF\n@hr_create.sql\n@hr_populate.sql\n@hr_code.sql\nEXIT\n' | sqlplus -s hr/hr@localhost/FREEPDB1"
```

Then run the verification queries above (`employees` should return 107 rows).

---

## Troubleshooting

| Problem | Fix |
| ------- | --- |
| `ORA-12514` / listener does not know service | Use **Service Name** `FREEPDB1`, not SID. Wait until logs show `DATABASE IS READY TO USE!` |
| `ORA-00942: table or view does not exist` | You are not connected as `HR`, or the schema is not installed (see above) |
| `ORA-00933` on `LIMIT 5` | Oracle has no `LIMIT`. Use `FETCH FIRST 5 ROWS ONLY` (or `WHERE ROWNUM <= 5`) |
| Port 1521 already in use | Stop the other Oracle container/service or change the port mapping in `docker-compose.yml` |
