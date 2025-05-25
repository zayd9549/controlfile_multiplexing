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
* Oracle **recommends at least 3 control files** on different physical disks

---

## 🧠 **Current Setup**

Check existing control files:

```sql
sql> show parameter control_files;

sql> select name from v$controlfile;
```

**Sample Output:**

```
/u01/oradata/ORADB/control01.ctl  
/u01/oradata/ORADB/control02.ctl
```

We will now add a **third control file** at `/u02/oradata/ORADB/control03.ctl`.

---

### 📌 **Before Starting: Check if SPFILE or PFILE is in Use**

```sql
sql> show parameter spfile;
```

**Output Examples:**

* If using **SPFILE**:
  `/u01/app/oracle/product/19.0.0/dbhome_1/dbs/spfileORADB.ora`

* If using **PFILE**:
  The result will be **blank** (no spfile in use).

💡 **Explanation**:
Because the process to multiplex control files differs based on whether you're using SPFILE or PFILE.

---

## 🔀 **Multiplexing Methods**

1. **SPFILE**
2. **PFILE**

---

## ⚙️ 1) Multiplexing Using **SPFILE**

### 🔹 Step 1: Update Control Files Parameter in SPFILE

```sql
sql> alter system set control_files=
'/u01/oradata/ORADB/control01.ctl',
'/u01/oradata/ORADB/control02.ctl',
'/u02/oradata/ORADB/control03.ctl'
scope=spfile;
```

💡 **Explanation**:
This updates the SPFILE with a new control file list. The change will apply on the next startup.

---

### 🔹 Step 2: Shut Down the Database

```sql
sql> shut immediate;
```

---

### 🔹 Step 3: Copy Control File to New Location

```bash
$ mkdir -p /u02/oradata/ORADB
$ cp /u01/oradata/ORADB/control01.ctl /u02/oradata/ORADB/control03.ctl
```

💡 **Explanation**:
All control files must have identical content. We're copying a valid control file to a new location.

---

### 🔹 Step 4: Start the Database

```bash
$ sqlplus / as sysdba
sql> startup;
```

---

### 🔹 Step 5: Confirm All 3 Control Files Are Active

```sql
sql> show parameter control_files;
sql> select name from v$controlfile;
```

Expected Output:

```
/u01/oradata/ORADB/control01.ctl  
/u01/oradata/ORADB/control02.ctl  
/u02/oradata/ORADB/control03.ctl
```

---

## ❌ Revert SPFILE Before PFILE Practice

If you want to test **pfile-based multiplexing**, first **remove** the 3rd control file from the spfile setup.

### 🔹 Step 1: Shut Down Database

```sql
sql> shut immediate;
```

---

### 🔹 Step 2: Revert Control Files List

```sql
sql> alter system set control_files=
'/u01/oradata/ORADB/control01.ctl',
'/u01/oradata/ORADB/control02.ctl'
scope=spfile;
```

---

### 🔹 Step 3: Delete 3rd Control File

```bash
$ rm -f /u02/oradata/ORADB/control03.ctl
```

---

### 🔹 Step 4: Start Database

```sql
sql> startup;
```

---

## 📄 2) Multiplexing Using **PFILE**

### ⚠️ Before You Begin

👉 If your database is running with **SPFILE**, you need to **create a PFILE** from it and move the SPFILE out of the way.

---

### 🔹 Step 1: Create PFILE from SPFILE

```sql
sql> create pfile from spfile;
```

💡 **Explanation**:
This generates a text-based init file (`initORADB.ora`) that captures the current configuration of the database.

---

### 🔹 Step 2: Rename or Move SPFILE

```bash
$ cd $ORACLE_HOME/dbs
$ mv spfileORADB.ora spfileORADB.ora_bkp
```

💡 **Explanation**:
Oracle will use the PFILE on next startup only if the SPFILE is not found.

---

### 🔹 Step 3: Shut Down the Database (If Not Already)

```sql
sql> shut immediate;
```

---

### 🔹 Step 4: Edit PFILE (`initORADB.ora`)

```bash
$ vi $ORACLE_HOME/dbs/initORADB.ora
```

Update the control\_files parameter:

```ini
*.control_files='/u01/oradata/ORADB/control01.ctl','/u01/oradata/ORADB/control02.ctl','/u02/oradata/ORADB/control03.ctl'
```

💡 **Explanation**:
You're manually specifying the third control file location in the parameter file.

---

### 🔹 Step 5: Copy Control File to New Location

```bash
$ mkdir -p /u02/oradata/ORADB
$ cp /u01/oradata/ORADB/control01.ctl /u02/oradata/ORADB/control03.ctl
```

💡 **Explanation**:
New file is created by cloning an existing valid control file.

---

### 🔹 Step 6: Start Database with PFILE

```bash
$ sqlplus / as sysdba
sql> startup pfile='$ORACLE_HOME/dbs/initORADB.ora';
```

---

### 🔹 Step 7: Confirm All 3 Control Files Are Active

```sql
sql> show parameter control_files;
sql> select name from v$controlfile;
```

Expected Output:

```
/u01/oradata/ORADB/control01.ctl  
/u01/oradata/ORADB/control02.ctl  
/u02/oradata/ORADB/control03.ctl
```

---

## 📖 Reference Views

| View            | Description                                     |
| --------------- | ----------------------------------------------- |
| `v$controlfile` | Lists all control files currently in use        |
| `v$parameter`   | Shows parameters currently active (from memory) |
| `v$spparameter` | Shows parameters stored in the SPFILE           |

---
