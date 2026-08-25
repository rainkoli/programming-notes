# 12.3.4 Table Character Set and Collation

## Official Documentation Path

MySQL 8.4 Reference Manual

    Character Sets, Collations, Unicode
        >
    Specifying Character Sets and Collations
        >
    Table Character Set and Collation

------------------------------------------------------------------------

# MySQL Character Set and Collation Priority

## 1. Character Set and Collation Relationship

### Character Set

Character Set defines how characters are encoded and stored.

Examples:

-   `utf8mb4`
-   `latin1`
-   `gbk`

Its responsibility:

    Character <----> Bytes

It decides how characters are converted into binary data.

------------------------------------------------------------------------

### Collation

Collation defines how characters are compared and sorted.

Examples:

-   `utf8mb4_bin`
-   `utf8mb4_general_ci`
-   `utf8mb4_0900_ai_ci`

For example:

`utf8mb4_bin`

    A != a

because it compares binary values.

`utf8mb4_general_ci`

    A = a

because `ci` means case insensitive.

Relationship:

    Character Set
          |
          |
          v
     Collation

One character set can have multiple collations.

------------------------------------------------------------------------

# 2. Table Character Set and Collation Selection Rules

MySQL chooses table character set and collation according to the
following rules.

------------------------------------------------------------------------

## Case 1: Both CHARACTER SET and COLLATE are specified

Example:

``` sql
CREATE TABLE user(
    name VARCHAR(50)
)
CHARACTER SET utf8mb4
COLLATE utf8mb4_bin;
```

Result:

    Table Character Set:

    utf8mb4

    Table Collation:

    utf8mb4_bin

MySQL uses exactly the values provided.

------------------------------------------------------------------------

## Case 2: Only CHARACTER SET is specified

Example:

``` sql
CREATE TABLE user(
    name VARCHAR(50)
)
CHARACTER SET utf8mb4;
```

MySQL uses:

    Character Set:

    utf8mb4


    Collation:

    utf8mb4 default collation

For MySQL 8.4:

    utf8mb4_0900_ai_ci

is the default collation.

------------------------------------------------------------------------

## Case 3: Only COLLATE is specified

Example:

``` sql
CREATE TABLE user(
    name VARCHAR(50)
)
COLLATE utf8mb4_bin;
```

MySQL finds the character set associated with the collation.

Because:

    utf8mb4_bin belongs to utf8mb4

The result:

    Character Set:

    utf8mb4


    Collation:

    utf8mb4_bin

------------------------------------------------------------------------

## Case 4: Neither CHARACTER SET nor COLLATE is specified

Example:

``` sql
CREATE TABLE user(
    name VARCHAR(50)
);
```

MySQL uses the database character set and collation.

Hierarchy:

    Server
       |
    Database
       |
    Table
       |
    Column

------------------------------------------------------------------------

# 3. Character Set Priority

The priority order is:

    Column
      |
    Table
      |
    Database
      |
    Server

The closer the configuration is to the column, the higher the priority.

------------------------------------------------------------------------

## Example

Server:

    latin1

Database:

    utf8mb4

Table:

    gbk

Column:

    latin1

Final result:

    Column uses latin1

The column setting overrides table, database, and server settings.

------------------------------------------------------------------------

# 4. Table Character Set as Column Default

Official meaning:

> The table character set and collation are used as default values for
> column definitions if the column character set and collation are not
> specified.

Example:

``` sql
CREATE TABLE user(
    name VARCHAR(50)
)
CHARACTER SET utf8mb4;
```

Equivalent to:

``` sql
CREATE TABLE user(
    name VARCHAR(50)
        CHARACTER SET utf8mb4
);
```

because the column inherits the table character set.

------------------------------------------------------------------------

## Column Override Example

``` sql
CREATE TABLE user(
    name VARCHAR(50)
        CHARACTER SET latin1
)
CHARACTER SET utf8mb4;
```

Result:

    Table:

    utf8mb4


    Column name:

    latin1

Column configuration has higher priority.

------------------------------------------------------------------------

# 5. DEFAULT CHARACTER SET vs CHARACTER SET

MySQL syntax:

``` sql
CREATE TABLE tbl_name
(
    column_list
)
[DEFAULT] CHARACTER SET charset_name
[COLLATE collation_name];
```

`DEFAULT` is optional.

The following statements are equivalent:

``` sql
CREATE TABLE user(
    name VARCHAR(50)
)
DEFAULT CHARACTER SET utf8mb4;
```

and:

``` sql
CREATE TABLE user(
    name VARCHAR(50)
)
CHARACTER SET utf8mb4;
```

Both produce:

    Table Character Set:

    utf8mb4

------------------------------------------------------------------------

# 6. Meaning of DEFAULT

`DEFAULT CHARACTER SET` emphasizes:

> This character set is the default value for columns that do not
> specify their own character set.

Example:

``` sql
CREATE TABLE user(
    name VARCHAR(50)
)
DEFAULT CHARACTER SET utf8mb4;
```

Means:

    Table default:

    utf8mb4


    Column without charset:

    inherit utf8mb4

------------------------------------------------------------------------

# 7. Why MySQL Provides DEFAULT Keyword?

Because MySQL has multiple levels of default values:

    Server
     |
    Database
     |
    Table
     |
    Column

The DEFAULT keyword makes the inheritance relationship clearer.

However:

    DEFAULT CHARACTER SET utf8mb4

and:

    CHARACTER SET utf8mb4

have the same execution result in CREATE TABLE.

------------------------------------------------------------------------

# 8. Recommended MySQL Configuration

Database:

``` sql
CREATE DATABASE app
DEFAULT CHARACTER SET utf8mb4
DEFAULT COLLATE utf8mb4_0900_ai_ci;
```

Table:

``` sql
CREATE TABLE user(
    id BIGINT,
    username VARCHAR(50)
)
DEFAULT CHARACTER SET utf8mb4
DEFAULT COLLATE utf8mb4_0900_ai_ci;
```

Recommended:

    Character Set:

    utf8mb4


    Collation:

    utf8mb4_0900_ai_ci

------------------------------------------------------------------------

# 9. Summary

## Character Set

Defines:

    How characters are encoded and stored

## Collation

Defines:

    How characters are compared and sorted

## Priority

    Column
      >
    Table
      >
    Database
      >
    Server

## DEFAULT

`DEFAULT CHARACTER SET`:

-   Optional keyword
-   Same effect as `CHARACTER SET`
-   Emphasizes inheritance behavior

Final rule:

> Column settings override Table settings. Table settings override
> Database settings. Database settings override Server settings.

------------------------------------------------------------------------

# Additional Organized Notes

# 12.3.4 Table Character Set and Collation

## Character Set and Collation Priority

MySQL character set inheritance priority:

    Column
      >
    Table
      >
    Database
      >
    Server

Column settings override table settings. Table settings override
database settings.

## Table Character Set Rules

1.  Both CHARACTER SET and COLLATE specified

``` sql
CREATE TABLE user(
    name VARCHAR(50)
)
CHARACTER SET utf8mb4
COLLATE utf8mb4_bin;
```

MySQL uses the specified values.

2.  Only CHARACTER SET specified

``` sql
CHARACTER SET utf8mb4
```

MySQL uses utf8mb4 and its default collation.

3.  Only COLLATE specified

``` sql
COLLATE utf8mb4_bin
```

MySQL finds the associated character set.

4.  Neither specified

The database character set and collation are used.

## DEFAULT CHARACTER SET

`DEFAULT` is optional.

These are equivalent:

``` sql
DEFAULT CHARACTER SET utf8mb4;
```

and:

``` sql
CHARACTER SET utf8mb4;
```

`DEFAULT` emphasizes that the table character set becomes the default
value for columns.

## Column Inheritance

Example:

``` sql
CREATE TABLE user(
    name VARCHAR(50)
)
DEFAULT CHARACTER SET utf8mb4;
```

The column inherits utf8mb4.

Column override example:

``` sql
CREATE TABLE user(
    name VARCHAR(50)
        CHARACTER SET latin1
)
CHARACTER SET utf8mb4;
```

The column uses latin1.

## Recommended Configuration

``` sql
CREATE DATABASE app
DEFAULT CHARACTER SET utf8mb4
DEFAULT COLLATE utf8mb4_0900_ai_ci;
```
