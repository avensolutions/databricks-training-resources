# Analysts Guide to Delta Lake

**Delta Lake** is an open-source storage framework that provides ACID properties to cloud (object) storage, enabling:
- fine-grained updates (`INSERT`, `UPDATE`, `MERGE`, or `DELETE`)
- schema enforcement (i.e., data type validation)
- time travel (i.e. versioning)
- audit history (i.e., transaction log)

Delta Lake is an open-source project hosted by the Linux Foundation.  The **Delta Lake** provider works with AWS S3, ADLS in Azure, GCS in Google, and more.

<details>
<summary>Contents</summary>
<ol>
<li><a href="#create-populate-table">Create and populate a Delta Lake table</a></li>
<li><a href="#show-metadata">Inspect Table Metadata and History</a></li>
<li><a href="#time-travel">Versioning and Time Travel</a></li>
<li><a href="#schema-enforcement">Schema Enforcement and Constraints</a></li>
</ol>
</details>

## Delta Lake by Example

The following example demonstrates how Delta Lake works. 

> Copy each script into a new SQL worksheet, and update the values for `your_catalog` and `your_schema` to your values.

### <a id="create-populate-table"></a>1. Create and populate a Delta Lake table

Let's create a Delta Lake table and run some data manipulation language (DML) statements against it...

```sql
USE CATALOG your_catalog;
USE SCHEMA your_schema;

-- version 0
CREATE OR REPLACE TABLE students
  (id INT, name STRING, gpa DOUBLE);

-- version 1  
INSERT INTO students VALUES (1, "Yve", 3.0);
-- version 2
INSERT INTO students VALUES (2, "Omar", 3.5);
-- version 3
INSERT INTO students VALUES (3, "Elia", 4.0);

-- version 4
INSERT INTO students
VALUES 
  (4, "Ted", 2.7),
  (5, "Tiffany", 3.5),
  (6, "Vini", 3.3);

-- version 5  
UPDATE students 
SET gpa = 3.1
WHERE id = "4";

-- version 6
DELETE FROM students 
WHERE id >= 6;
```

### <a id="show-metadata"></a>2. Inspect Table Metadata and History

The **`DESCRIBE EXTENDED`** command allows us to inspect the table's metadata.  Delta Lake maintains a transaction log for each table.  We can see summary information about a Delta Lake object using **`DESCRIBE DETAIL`**.  Alternatively, we can use the **`DESCRIBE HISTORY`** command to see the table's change history.

```sql
USE CATALOG your_catalog;
USE SCHEMA your_schema;

/* show columns and table metadata */
DESCRIBE EXTENDED students;

/* show Delta Lake information about the current version of the table */
DESCRIBE DETAIL students;

/* review changes to the table */
DESCRIBE HISTORY students;
```

### <a id="time-travel"></a>3. Versioning and Time Travel

As Delta Lake keeps a transaction log of all changes to a table, we can use the time travel feature to view or restore data to any previous point in time.

```sql
USE CATALOG your_catalog;
USE SCHEMA your_schema;

/* traverse to any version using the `VERSION AS OF` command */
SELECT * FROM students VERSION AS OF 1;

/* show data at a particaulr point in time */
SELECT * FROM students TIMESTAMP AS OF "paste_the_timestamp_for_version_1_here";

/* create a view representing the table as it was at version 1 */
CREATE OR REPLACE VIEW students_v1 AS
SELECT * FROM students VERSION AS OF 1;

/* select from the view */
SELECT * FROM students_v1;

/* restore the table to a previous version */
RESTORE TABLE students TO VERSION AS OF 1;

/* you can see the restore operation in the history */
DESCRIBE HISTORY students;
```

### <a id="schema-enforcement"></a>4. Schema Enforcement and Constraints 

Delta Lake provides schema enforcement, meaning you can't insert data that doesn't match the table schema.

```sql
USE CATALOG your_catalog;
USE SCHEMA your_schema;

/* try to insert a row with a string value for the `gpa` column; this should fail... */
INSERT INTO students VALUES (7, "John", "Smith");

/* you can also enforce constraints on Delta Lake tables */
ALTER TABLE students
ADD CONSTRAINT valid_gpa_constraint CHECK (gpa >= 0 AND gpa <= 4);

/* attempt to insert a row with a `gpa` value outside the valid range; this should fail... */
INSERT INTO students VALUES (7, "John", 5.0);
```