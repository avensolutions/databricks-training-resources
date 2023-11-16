**Delta Lake** is an open source storage framework that provides ACID properties to cloud (object) storage, enabling:
- fine grained updates (`INSERT`, `UPDATE`, `MERGE`, or `DELETE`)
- schema enforcement (i.e. data type validation)
- time travel (i.e. versioning)
- audit history (i.e. transaction log)

Delta Lake is an open source project that is part of the Linux Foundation.  The **Delta Lake** provider works with AWS S3, ADLS in Azure, GCS in Google, and more.

## Delta Lake by Example

Lets create a Delta Lake table and run some data manipulation language (DML) statements against it...

1. Create and populate a Delta Lake table

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

**`DESCRIBE EXTENDED`** allows us to see important metadata about our table.

```sql
DESCRIBE EXTENDED students;
```

**`DESCRIBE DETAIL`** is another command that allows us to explore table metadata.

```sql
DESCRIBE DETAIL students;
```

As every change to a Delta Lake table is stored in the tables transaction log, we can use the **`DESCRIBE HISTORY`** command to review changes.

```sql
DESCRIBE HISTORY students
```

> Note the correlation between the version number in the block of Data Manipulation Language (DML) statements and the version number in the delta lake transaction log

### Time Travel

You can traverse to any version using the **`VERSION AS OF`** command.

```sql  
SELECT * FROM students VERSION AS OF 1;
```

You can also traverse to any version using the **`TIMESTAMP AS OF`** command.

```sql
SELECT * FROM students TIMESTAMP AS OF "paste_the_timestamp_for_version_1_here";
```

You can use the time travel feature to create views which represent data at a specific point in time.

```sql
CREATE OR REPLACE VIEW students_v1 AS
SELECT * FROM students VERSION AS OF 1;
``` 

You can restore a table to a previous version using the **`RESTORE`** command.

```sql
RESTORE TABLE students TO VERSION AS OF 1;
```

### Schema Enforcement

Delta Lake provides schema enforcement, which means that you can't insert data that doesn't match the table schema.

```sql
INSERT INTO students VALUES (7, "John", "3.0");
```

You can also enforce constraints with the **`ALTER TABLE`** command.

```sql
ALTER TABLE students
ADD CONSTRAINT valid_gpa_constraint CHECK (gpa >= 0 AND gpa <= 4);
```

this should fail now:

```sql
INSERT INTO students VALUES (7, "John", "5.0");
```