# Introduction to Delta Lake

**Delta Lake** is an open-source storage framework that provides ACID properties to cloud (object) storage, enabling:
- fine-grained updates (`INSERT`, `UPDATE`, `MERGE`, or `DELETE`)
- schema enforcement (i.e., data type validation)
- time travel (i.e. versioning)
- audit history (i.e., transaction log)

Delta Lake is an open-source project that is part of the Linux Foundation.  The **Delta Lake** provider works with AWS S3, ADLS in Azure, GCS in Google, and more.

<details>
<summary>Contents</summary>
<ol>
    <li>
        <a href="#delta-lake-versioning-and-history">Delta Lake Versioning and History</a>
        <ol>
            <li><a href="#create-populate-table">Create and Populate a Delta Lake Table</a></li>
            <li><a href="#show-metadata-extended">Show Table Metadata Using `DESCRIBE EXTENDED`</a></li>
            <li><a href="#show-metadata-detail">Show Table Metadata Using `DESCRIBE DETAIL`</a></li>
            <li><a href="#show-table-history">Show Table History Using `DESCRIBE HISTORY`</a></li>
        </ol>
    </li>
    <li>
        <a href="#time-travel">Time Travel</a>
        <ol>
            <li><a href="#version-as-of">Show Table as it was at Version 1</a></li>
            <li><a href="#timestamp-as-of">Show Table as it was at a Point in Time</a></li>
            <li><a href="#create-view">Create a View of the Table as it was at Version 1</a></li>
            <li><a href="#restore-table">Restore the Table to a Previous Version</a></li>
        </ol>
    </li>
    <li>
        <a href="#schema-enforcement-and-constraints">Schema Enforcement and Constraints</a>
        <ol>
            <li><a href="#insert-incorrect-data">Try to Insert a Row with an Incorrect Data Type</a></li>
            <li><a href="#create-constraint">Create a Constraint to Enforce a Valid Range for the `gpa` Column</a></li>
            <li><a href="#insert-outside-range">Attempt to Insert a Row with a `gpa` Value Outside the Valid Range</a></li>
        </ol>
    </li>
</ol>
</details>

## Delta Lake Versioning and History

The following example demonstrates how Delta Lake works.

### <a id="create-populate-table"></a>1.  Create and populate a Delta Lake table

> Let's create a Delta Lake table and run some data manipulation language (DML) statements against it...

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
WHERE value > 6;
```

### <a id="show-metadata-extended"></a>2.  Show table metadata using `DESCRIBE EXTENDED`

> **`DESCRIBE EXTENDED`** allows us to see important metadata about our table; let's try it...

```sql
DESCRIBE EXTENDED students;
```

### <a id="show-metadata-detail"></a>3.  Show table metadata using `DESCRIBE DETAIL`

> **`DESCRIBE DETAIL`** is another command that allows us to explore table metadata.

```sql
DESCRIBE DETAIL students;
```

### <a id="show-table-history"></a>4.  Show table history using `DESCRIBE HISTORY`

> As every change to a Delta Lake table is stored in the tables transaction log, we can use the **`DESCRIBE HISTORY`** command to review changes; let's check it out...

```sql
DESCRIBE HISTORY students
```

> Note the correlation between the version number in the block of Data Manipulation Language (DML) statements and the version number in the Delta Lake transaction log

## Time Travel

As Delta Lake keeps a transaction log of all changes to a table, we can use the time travel feature to view or restore data to any previous point in time.

### <a id="version-as-of"></a>5.  Show the table as it was at version 1

> You can traverse to any version using the **`VERSION AS OF`** command.

```sql  
SELECT * FROM students VERSION AS OF 1;
```

### <a id="timestamp-as-of"></a>6.  Show the table as it was at a point in time

> You can also traverse to any version using the **`TIMESTAMP AS OF`** command.

```sql
SELECT * FROM students TIMESTAMP AS OF "paste_the_timestamp_for_version_1_here";
```

### <a id="create-view"></a>7.  Create a view of the table as it was at version 1

> You can use the time travel feature to create views representing data at a specific point in time.

```sql
CREATE OR REPLACE VIEW students_v1 AS
SELECT * FROM students VERSION AS OF 1;
``` 

### <a id="restore-table"></a>8.  Restore the table to a previous version

> You can restore a table to a previous version using the **`RESTORE`** command.

```sql
RESTORE TABLE students TO VERSION AS OF 1;
```

## Schema Enforcement and Constraints

Delta Lake provides schema enforcement, meaning you can't insert data that doesn't match the table schema.

### <a id="insert-incorrect-data"></a>9.  Try to insert a row with an incorrect data type 

> Let's try to insert a row with a string value for the `gpa` column; this should fail...

```sql
INSERT INTO students VALUES (7, "John", "3.0");
```

### <a id="create-constraint"></a>10.  Create a constraint to enforce a valid range for the `gpa` column

> You can also enforce constraints with the **`ALTER TABLE`** command.

```sql
ALTER TABLE students
ADD CONSTRAINT valid_gpa_constraint CHECK (gpa >= 0 AND gpa <= 4);
```

### 11.  Attempt to insert a row with a `gpa` value outside the valid range

> This should fail now:

```sql
INSERT INTO students VALUES (7, "John", "5.0");
```