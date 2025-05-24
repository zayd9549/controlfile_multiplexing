## **6. Control File Multiplexing**

### 🔍 **Purpose**

Control files are **critical binary files** in an Oracle database that record the physical structure of the database, such as:

* Database name
* Datafile names and locations
* Redo log files
* Checkpoint information
* Backup and recovery metadata

If a control file is **lost or corrupted**, the database **cannot be mounted or opened**, resulting in possible downtime or data loss.

As a DBA, it is your **responsibility** to **multiplex and back up** control files to:

* Protect against disk failures
* Minimize the risk of database unavailability

---

## ✅ **Best Practice**

Always **multiplex** the control files on **separate physical disks** to avoid a single point of failure.

---

## 🔧 Methods of Control File Multiplexing

There are **two methods** based on the type of parameter file in use:

1. **Using SPFILE (Server Parameter File)**
2. **Using PFILE (Parameter File - init file)**

---

## 1️⃣ Using **SPFILE**

### 🔸 Step 1: Check Current Control File Locations

```sql
SQL> SELECT name FROM v$controlfile;
-- or --
SQL> SHOW PARAMETER control_files;
```

---

### 🔸 Step 2: Update SPFILE with New Control File Paths

```sql
SQL> ALTER SYSTEM SET control_files=
'/u01/oradata/ORADB/control01.ctl',
'/u02/oradata/ORADB/control02.ctl' 
SCOPE=spfile;
```

📌 **Explanation:**

* This sets multiple control file paths.
* `SCOPE=spfile` applies the change only in the **SPFILE**.
* The change takes effect **after database restart**.

---

### 🔸 Step 3: Shutdown the Database

```sql
SQL> SHUTDOWN IMMEDIATE;
```

---

### 🔸 Step 4: Copy Control File to New Location

```bash
$ cd /u01/oradata/ORADB
$ cp control01.ctl /u02/oradata/ORADB/control02.ctl
```

> 🔧 If the `/u02/oradata/ORADB` directory doesn’t exist, create it using `mkdir -p`.

---

### 🔸 Step 5: Start the Database

```bash
$ sqlplus / as sysdba
SQL> STARTUP;
```

---

### 🔸 Step 6: Verify Multiplexed Control Files

```sql
SQL> SELECT name FROM v$controlfile;
```

You should see both `control01.ctl` and `control02.ctl` listed.

---

## 2️⃣ Using **PFILE**

### 🔸 Step 1: Check Existing Control Files

```sql
SQL> SELECT name FROM v$controlfile;
```

---

### 🔸 Step 2: Shutdown the Database

```sql
SQL> SHUTDOWN IMMEDIATE;
```

---

### 🔸 Step 3: Edit the PFILE

Navigate to the `$ORACLE_HOME/dbs` directory and edit the PFILE (init file):

```bash
$ cd $ORACLE_HOME/dbs
$ vi initORADB.ora
```

Update or add the following line:

```text
*.control_files='/u01/oradata/ORADB/control01.ctl','/u02/oradata/ORADB/control02.ctl'
```

Save and exit (`:wq!`)

---

### 🔸 Step 4: Copy Control File to New Location

```bash
$ cp /u01/oradata/ORADB/control01.ctl /u02/oradata/ORADB/control02.ctl
```

---

### 🔸 Step 5: Start the Database

```bash
$ sqlplus / as sysdba
SQL> STARTUP;
```

---

### 🔸 Step 6: Verify Control Files

```sql
SQL> SELECT name FROM v$controlfile;
```

---

## 🔍 Reference Views

* `V$CONTROLFILE` – Lists control files in use.
* `V$PARAMETER` – Shows current parameter settings.
* `V$SPPARAMETER` – Shows SPFILE parameter values.

---

## ⚠️ Important Notes

* Oracle writes to **all control files simultaneously**.
* All copies must be **identical binary replicas**.
* If one is missing or corrupted, the database **fails to mount**.
* Ensure all locations are on **separate physical disks** to avoid a single point of failure.
