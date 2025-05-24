
## 🧩 **6. Control File Multiplexing**

### 🎯 **Purpose**

Control files store **critical metadata** about the Oracle database such as:

* Database name and creation timestamp
* Datafile and redo log file paths
* Backup information and checkpoints

❗ If even one control file is lost or corrupted, the database **cannot be mounted**. As a DBA, **multiplexing control files** across different disks is essential to ensure database resilience and availability.

---

## 🛡️ **Why Multiplex?**

* Protects against **single point of failure**
* Enables **recovery** in case of disk corruption or accidental deletion
* **Oracle recommends** storing at least **3 control files** on **different physical devices**

---

## 🔀 **Multiplexing Methods**

You can multiplex control files using:

1. **SPFILE**
2. **PFILE**

---

## 🧠 **Current Setup**

Check existing control files:

```sql
sql> show parameter control_files;

sql> select name from v$controlfile;
```

**Output:**

```
/u01/oradata/ORADB/control01.ctl  
/u01/oradata/ORADB/control02.ctl
```

Now we will add a **3rd control file** at `/u02/oradata/ORADB/control03.ctl`.

---

## ⚙️ 1) Multiplexing Using **spfile**

### 🔹 Step 1: Update Control File List in spfile

```sql
sql> alter system set control_files=
'/u01/oradata/ORADB/control01.ctl',
'/u01/oradata/ORADB/control02.ctl',
'/u02/oradata/ORADB/control03.ctl'
scope=spfile;
```

---

### 🔹 Step 2: Shut Down the Database

```sql
sql> shut immediate;
```

---

### 🔹 Step 3: Copy Control File to New Location

> 🔍 **Check if directory doesn't exist, create it:**

```bash
$ mkdir -p /u02/oradata/ORADB
$ cp /u01/oradata/ORADB/control01.ctl /u02/oradata/ORADB/control03.ctl
```

---

### 🔹 Step 4: Start the Database

```bash
$ sqlplus / as sysdba
sql> startup;
```

---

### 🔹 Step 5: Confirm All 3 Control Files Are in Use

```sql
sql> show parameter control_files;

sql> select name from v$controlfile;
```

**Expected Output:**

```
/u01/oradata/ORADB/control01.ctl  
/u01/oradata/ORADB/control02.ctl  
/u02/oradata/ORADB/control03.ctl
```

---

## ❌ Before Practicing with pfile

If you want to test **pfile-based multiplexing**, **remove** the 3rd control file first.

### 🔹 Step 1: Shut Down Database

```sql
sql> shut immediate;
```

### 🔹 Step 2: Edit spfile to Remove Third Entry

```sql
sql> alter system set control_files=
'/u01/oradata/ORADB/control01.ctl',
'/u01/oradata/ORADB/control02.ctl'
scope=spfile;
```

### 🔹 Step 3: Delete 3rd Control File (Optional Cleanup)

```bash
$ rm -f /u02/oradata/ORADB/control03.ctl
```

### 🔹 Step 4: Start Database

```sql
sql> startup;
```

---

## 📄 2) Multiplexing Using **pfile**

### 🔹 Step 1: Shut Down the Database

```sql
sql> shut immediate;
```

---

### 🔹 Step 2: Edit pfile (`initORADB.ora`)

```bash
$ cd $ORACLE_HOME/dbs
$ vi initORADB.ora
```

Update this line:

```ini
*.control_files='/u01/oradata/ORADB/control01.ctl','/u01/oradata/ORADB/control02.ctl','/u02/oradata/ORADB/control03.ctl'
```

Save and exit (`:wq!`)

---

### 🔹 Step 3: Copy Control File

> 🔍 **Check if directory doesn't exist, create it:**

```bash
$ mkdir -p /u02/oradata/ORADB
$ cp /u01/oradata/ORADB/control01.ctl /u02/oradata/ORADB/control03.ctl
```

---

### 🔹 Step 4: Start Database with pfile

```bash
$ sqlplus / as sysdba
sql> startup pfile='$ORACLE_HOME/dbs/initORADB.ora';
```

---

### 🔹 Step 5: Confirm All 3 Control Files Are in Use

```sql
sql> show parameter control_files;

sql> select name from v$controlfile;
```

---

## 🔍 **Reference Views**

* `v$controlfile` – Control file names in use
* `v$parameter` – Parameter values in use
* `v$spparameter` – Parameter values stored in spfile

---
