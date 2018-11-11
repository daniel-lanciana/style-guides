# SQL Style Guide

- [Overview](#overview)
- [General](#general)
	- [Do](#do)
	- [Avoid](#avoid)
- [Naming conventions](#naming-conventions)
	- [General](#general-1)
	- [Tables](#tables)
	- [Columns](#columns)
	- [Aliasing or correlations](#aliasing-or-correlations)
	- [Uniform suffixes](#uniform-suffixes)
- [Schema design](#schema-design)
	- [Data types](#data-types)
	- [Default values](#default-values)
	- [UUIDs (Universally Unique Identifiers)](#uuids-universally-unique-identifiers)
	- [Primary keys](#primary-keys)
	- [Constraints](#constraints)
	- [Indexes](#indexes)
	- [Validation](#validation)
	- [Create syntax](#create-syntax)
	- [Designs to avoid](#designs-to-avoid)
- [Query syntax](#query-syntax)
	- [Reserved words](#reserved-words)
	- [Spacing](#spacing)
	- [Joins](#joins)
	- [Parentheses](#parentheses)
	- [Preferred formalisms](#preferred-formalisms)
- [Integration with the logic layer](#integration-with-the-logic-layer)
- [References](#references)

## Overview

Defines the set of standards pertaining to relational database design and
interaction by engineers or data scientists.

Forked from https://github.com/treffynnon/sqlstyle.guide with the most significant
changes being the removal of the river pattern, ORM (Object Relational Mapping) 
considerations, and the inclusion of UUIDs.

## General

### Do

* Use consistent and descriptive identifiers and names.
* Make judicious use of white space and indentation to make code easier to read.
* Use auto-incrementing integers when a surrogate primary key is needed.
* Store [ISO-8601][iso-8601] compliant time and date information
  (`YYYY-MM-DD HH:MM:SS.SSSSS`).
* Try to use only standard SQL functions instead of vendor specific functions for
  reasons of portability.
* Keep code succinct and devoid of redundant SQL—such as unnecessary quoting or
  parentheses or `WHERE` clauses that can otherwise be derived.
* Include comments in SQL code where necessary. Use the C style opening `/*` and
  closing `*/` where possible otherwise precede comments with `--` and finish
  them with a new line.
* Include a standard `created_at` timestamp field in all tables, `updated_at` for
  all mutable tables, and `archived_at` for any tables where rows may be archived
  (as a traceable alternative to deletion, where applicable). This encourages
  traceability and is a standard with frameworks such as Ruby on Rails.

### Avoid

* CamelCase—it is difficult to scan quickly.
* Descriptive prefixes or Hungarian notation such as `sp_` or `tbl`.
* Plurals—use the more natural collective term where possible instead. For example
  `staff` instead of `employees` or `people` instead of `individuals`.
* Quoted identifiers—if you must use them then stick to SQL92 double quotes for
  portability (you may need to configure your SQL server to support this depending
  on vendor).
* Object oriented design principles should not be applied to SQL or database
  structures.
* Stored procedures—logic (e.g. stored procedures, triggers) should be in the application, 
  not data, layer. They make it harder to reason about code execution and state changes.

## Naming conventions

Strive for consistent, idiomatic naming that integrates well with established ORM solutions.

### General

* Ensure the name is unique and does not exist as a reserved keywords.
* Keep the length to a maximum of 30 characters.
* Only use letters, numbers and underscores in names—and begin with a letter and not end with an underscore.
* Avoid the use of multiple consecutive underscores—these can be hard to read.
* Use underscores where you would naturally include a space in the name (first
  name becomes `first_name`).
* Avoid abbreviations and if you have to use them make sure they are commonly
  understood.

### Tables

* Use a collective name or, less ideally, a plural form. For example (in order of
  preference) `staff` and `employees`.
* Do not prefix with `tbl` or any other such descriptive prefix or Hungarian
  notation.
* Never give a table the same name as one of its columns and vice versa.
* Avoid, where possible, concatenating two table names together to create the name
  of a relationship table. Rather than `car_mechanics` prefer `services`. When 
  unavoidable use the Ruby on Rails convention of only using a plural for the last
  word (`car_mechanics` instead of `cars_mechanics`).

### Columns

* Always use the singular name.
* Surrogate primary key fields should be named `id`.  
* Foreign key fields should be `singularized_table_name_id` (such as `item_id`).  
* Do not add a column with the same name as its table and vice versa.

### Aliasing or correlations

* Should relate in some way to the object or expression they are aliasing.
* As a rule of thumb the correlation name should be the first letter of each word
  in the object's name.
* If there is already a correlation with the same name then append a number.
* Always include the `AS` keyword—makes it easier to read as it is explicit.
* For computed data (`SUM()` or `AVG()`) use the name you would give it were it
  a column defined in the schema.

### Uniform suffixes

The following suffixes have a universal meaning ensuring the columns can be read
and understood easily from SQL code. Use the correct suffix where appropriate.

* `_id`—a unique identifier such as a column that is a primary key.
* `_status`—flag value or some other status of any type such as
  `publication_status`.
* `_total`—the total or sum of a collection of values.
* `_num`—denotes the field contains any kind of number.
* `_name`—signifies a name such as `first_name`.
* `_seq`—contains a contiguous sequence of values.
* `_date`—denotes a column that contains the date of something.
* `_tally`—a count.
* `_size`—the size of something such as a file size or clothing.
* `_addr`—an address for the record could be physical or intangible such as
  `ip_addr`.

## Schema design

### Data types

* Where possible do not use vendor specific data types—these are not portable and
  may not be available in older versions of the same vendor's software.
* Use optimaly-sized data types and lengths—it reduces size and increases performance
  as the database grows. Be aware of the range of values each type can accomodate
  (e.g. unsigned numbers, decimal place precision).
* Only use `REAL` or `FLOAT` types where it is strictly necessary for floating
  point mathematics otherwise prefer `NUMERIC` and `DECIMAL` at all times. Floating
  point rounding errors are a nuisance!

### Default values

* The default value must be the same type as the column—if a column is declared
  a `DECIMAL` do not provide an `INTEGER` default value.
* Default values must follow the data type declaration and come before any
  `NOT NULL` statement.

### UUIDs (Universally Unique Identifiers)

128-bit random (using UUIDv4) private keys should only be used with the following
requirements outweigh the performance (more disk space, memory, slower sorting), 
portability (uuid data type) and usability tradeoffs (lack of inbuilt way to generate
UUIDs in database, harder to remember and verbalize for ad-hoc operations):

* hide underlying infromation about the system (e.g. the total number of users) and
  prevent attack vectors (e.g. scraping, brute force) *AND* a unique, public, human-friendly
  alternative does not exist—such as a username or slug
* to save round-trip calls to the database to obtain the primary key in order to perform
  an intended action
* a requirement for unique IDs across databases so they can be merged without key 
  collisions.

UUIDs also have no natural order, so can't be used in clustered databases without using a 
sequential UUID (e.g. with `newsequentialid()`) at the cost reduced randomness.

### Primary keys

Tables must have at least one key to be complete and useful—as it will effect performance 
and referntial integrity. A primary key (or combined composite key) needs to be:

* unique
* immutable (will never change)
* an integer (or a data type that can be validated against a standard ISO format).

Using an auto-incrementing integer is good practice for tables that do not have a
naturally-ocurring primary key.

### Constraints

Constraints help ensure referential integrity and define relationships between tables. 

Use of `FOREIGN KEY` should always be used when referencing values from another table. Use 
of `UNIQUE` should always be used when one-or-more fields are not permitted to be
duplicated within the confines of a table. 

### Indexes

Should be used on frequently queried, non-key fields of data types that lend themselves
to efficient sorting (numericals, short strings) and have a high read-to-write ratio
(to offset the space and reindexing overhead).

### Validation

It's acceptable to perform data validation only at the application layer, but for
complete integrity the same rules can be applied at the data layer also. Be
aware that any changes to validation will require code and schema changes.

* Use `LIKE` and `SIMILAR TO` constraints to ensure the integrity of strings
  where the format is known.
* Where the ultimate range of a numerical value is known it must be written as a
  range `CHECK()` to prevent incorrect values entering the database or the silent
  truncation of data too large to fit the column definition. In the least it
  should check that the value is greater than zero in most cases.
* `CHECK()` constraints should be kept in separate clauses to ease debugging.

### Create syntax

* Specify the primary key first right after the `CREATE TABLE` statement.
* Constraints should be defined directly beneath the column they correspond to.
  Indent the constraint so that it aligns to the right of the column name.
* If it is a multi-column constraint then consider putting it as close to both
  column definitions as possible and where this is difficult as a last resort
  include them at the end of the `CREATE TABLE` definition.
* If it is a table level constraint that applies to the entire table then it
  should also appear at the end.
* Use alphabetical order where `ON DELETE` comes before `ON UPDATE`.
* If it make senses to do so align each aspect of the query on the same character
  position. For example all `NOT NULL` definitions could start at the same
  character position. This is not hard and fast, but it certainly makes the code
  much easier to scan and read.

```sql
CREATE TABLE staff (
    PRIMARY KEY (staff_num),
    staff_num      INT(5)       NOT NULL,
    first_name     VARCHAR(100) NOT NULL,
    pens_in_drawer INT(2)       NOT NULL,
                   CONSTRAINT pens_in_drawer_range
                   CHECK(pens_in_drawer >= 1 AND pens_in_drawer < 100)
)
```

### Designs to avoid

* Splitting up data that should be in one table across many because of arbitrary
  concerns such as time-based archiving or location in a multi-national
  organisation. Later queries must then work across multiple tables with `UNION`
  rather than just simply querying one table.
* Repeating values across multiple tables uses more space and leads to data
  integrity issues and more effort to refactor (e.g. renaming). Instead refactor values 
  into a common table referenced via a foreign key relationship.  

## Query syntax

### Reserved words

Always use uppercase for the reserved keywords like `SELECT` and `WHERE`.

Use of abbreviated keywords is encouraged (`ABS` to `ABSOLUTE`).

Do not use database server specific keywords where an ANSI SQL keyword already
exists performing the same function. This helps to make code more portable.

### Spacing

To make the code easier to read it is important that the correct complement of
spacing is used. Do not crowd code or remove natural language spaces.

Root keywords should all start on the same character boundary. This is counter to 
the common "rivers" pattern described [here](https://www.sqlstyle.guide/#spaces). Less
annoying to write and easy to comment out specific lines for debugging.

It's acceptable to include an argument on the same line as the root keyword, if there is exactly one argument.  

**Good**

```sql
SELECT
  client_id,
  submission_date
FROM
  main_summary
WHERE
  sample_id = '42'
  AND submission_date > '20180101'
LIMIT
  10
```

**Bad**

```sql
SELECT client_id,
       submission_date
  FROM main_summary
 WHERE sample_id = '42'
   AND submission_date > '20180101'
 LIMIT 10
```

Although not exhaustive always include spaces:

* before and after equals (`=`)
* after commas (`,`)
* surrounding apostrophes (`'`) where not within parentheses or with a trailing
  comma or semicolon.
  
Always include newlines/vertical space:

* before `AND` or `OR`
* after semicolons to separate queries for easier reading
* after each keyword definition
* after a comma when separating multiple columns into logical groups
* to separate code into related sections, which helps to ease the readability of
  large chunks of code.

```sql
UPDATE albums -- single argument on same line as root keyword
SET 
  release_date = '1990-01-01 01:01:01.00000'
WHERE 
  title = 'The New Danger'
```

```sql
SELECT 
  SUM(a.copies) AS copies_total,
  a.title,
  a.release_date, a.recording_date, a.production_date -- grouped dates together
FROM 
  albums AS a
WHERE 
  a.title = 'Charcoal Lane'
  OR a.title = 'The New Danger'
```

### Joins

Joins should be indented with left-aligned root keywords.

```sql
SELECT 
  r.last_name
FROM 
  riders AS r
INNER JOIN 
  bikes AS b
ON 
  r.bike_vin_num = b.vin_num
  AND b.engine_tally > 2
INNER JOIN 
  crew AS c
ON 
  r.crew_chief_last_name = c.last_name
  AND c.chief = 'Y'
```

### Parentheses

If parentheses span multiple lines:

* the opening parenthesis should terminate the line
* the closing parenthesis should be lined up under the first character of the line that starts the multi-line construct
* the contents of the parentheses should be indented one level.

Do not use nested queries. Instead, use Common Table Expressions (the `WITH` keyword) to improve readability.

```sql
WITH sample AS (
  SELECT
    client_id,
  FROM
    main_summary
  WHERE
    sample_id = '42'
)
```

### Preferred formalisms

* Make use of `BETWEEN` where possible instead of combining multiple statements
  with `AND`.
* Similarly use `IN()` instead of multiple `OR` clauses.
* Where a value needs to be interpreted before leaving the database use the `CASE`
  expression. `CASE` statements can be nested to form more complex logical structures.
* Avoid the use of `UNION` clauses and temporary tables where possible. If the
  schema can be optimised to remove the reliance on these features then it most
  likely should be.

```sql
SELECT 
  CASE postcode
    WHEN 'BN1' 
      THEN 'Brighton'
    WHEN 'EH1' 
      THEN 'Edinburgh'
  END AS city
FROM 
  office_locations
WHERE 
  country = 'United Kingdom'
  AND opening_time BETWEEN 8 AND 9
  AND postcode IN ('EH1', 'BN1', 'NN1', 'KW1')
```

## Integration with the logic layer

* Avoid writing SQL, use an ORM (faster development, database agnostic, easier 
  to maintain)
* Use migration scripts for schema and seed data changes (versioned, committed 
  to repository, rollbacks)

## References

* [Reserved keywords](https://www.sqlstyle.guide)
