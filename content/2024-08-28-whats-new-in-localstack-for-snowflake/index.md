---
title: What’s new in LocalStack for Snowflake?
description: What’s new in LocalStack for Snowflake?
lead: What’s new in LocalStack for Snowflake?
date: 2024-08-28T11:57:42+05:30
lastmod: 2024-08-28T11:57:42+05:30
images: []
contributors: []
tags: ['news']
---

## Introduction

## How to upgrade?

## New features

- [Support for new functions](#support-for-new-functions)
- [New Web User Interface](#new-web-user-interface) (**Preview**)
- [Support for LocalStack Ephemeral Instances](#support-for-localstack-ephemeral-instances) (**Preview**)
- [Improved parity with Snowflake](#improved-parity-with-snowflake)
- [Support for Snowpipe](#support-for-snowpipe)
- [Support for Iceberg tables](#support-for-iceberg-tables)
- [Support for Hybrid tables](#support-for-hybrid-tables)
- [Support for Dynamic Tables](#support-for-dynamic-tables)

### Support for new functions

We have added support for several new functions in LocalStack for Snowflake. The new functions include:

| Function | Notes |
|----------|-------|
| [DATEDIFF](https://docs.snowflake.com/en/sql-reference/functions/datediff.html) | Returns the difference between two dates or timestamps. |
| [TIMEDIFF](https://docs.snowflake.com/en/sql-reference/functions/timediff.html) | Returns the difference between two time values. |
| [LEAD](https://docs.snowflake.com/en/sql-reference/functions/lead.html) | Accesses the value of a subsequent row in a result set without the use of a self-join. |
| [LAG](https://docs.snowflake.com/en/sql-reference/functions/lag.html) | Accesses the value of a previous row in a result set without the use of a self-join. |
| [INITCAP](https://docs.snowflake.com/en/sql-reference/functions/initcap.html) | Converts the first letter of each word in a string to uppercase and the rest to lowercase. |
| [LAST_QUERY_ID](https://docs.snowflake.com/en/sql-reference/functions/last_query_id.html) | Returns the query ID of the last executed query in the session. |
| [DESCRIBE FUNCTION](https://docs.snowflake.com/en/sql-reference/sql/desc-function) | Describes a user-defined function. |
| [SPLIT](https://docs.snowflake.com/en/sql-reference/functions/split.html) | Splits a string into an array based on a specified delimiter. |
| [DATE_FROM_PARTS](https://docs.snowflake.com/en/sql-reference/functions/date_from_parts.html) | Constructs a date from individual year, month, and day parts. |
| [FLOOR](https://docs.snowflake.com/en/sql-reference/functions/floor.html) | Rounds down a numeric value to the nearest integer. |
| [MIN](https://docs.snowflake.com/en/sql-reference/functions/min.html) | Returns the smallest value in a set of values. |
| [MIN_BY](https://docs.snowflake.com/en/sql-reference/functions/min_by.html) | Returns the minimum value of a specified column, grouped by another column. |
| [MAX](https://docs.snowflake.com/en/sql-reference/functions/max.html) | Returns the largest value in a set of values. |
| [MAX_BY](https://docs.snowflake.com/en/sql-reference/functions/max_by.html) | Returns the maximum value of a specified column, grouped by another column. |
| [COALESCE](https://docs.snowflake.com/en/sql-reference/functions/coalesce.html) | Returns the first non-null value in a list of arguments. |
| [DIV0](https://docs.snowflake.com/en/sql-reference/functions/div0.html) | Returns zero if the divisor is zero; otherwise, it performs division. |
| [LEAST](https://docs.snowflake.com/en/sql-reference/functions/least.html) | Returns the smallest value from a list of expressions. |
| [GREATEST](https://docs.snowflake.com/en/sql-reference/functions/greatest.html) | Returns the largest value from a list of expressions. |
| [NVL2](https://docs.snowflake.com/en/sql-reference/functions/nvl2.html) | Returns one value if a specified expression is not null, otherwise returns another value. |
| [IS_NULL_VALUE](https://docs.snowflake.com/en/sql-reference/functions/is_null_value.html) | Checks if a value is null and returns a boolean result. |
| [PARSE_IP](https://docs.snowflake.com/en/sql-reference/functions/parse_ip.html) | Parses an IP address and returns its components. |
| [OBJECT_KEYS](https://docs.snowflake.com/en/sql-reference/functions/object_keys.html) | Returns the keys of an object as an array. |
| [OBJECT_CONSTRUCT_KEEP_NULL](https://docs.snowflake.com/en/sql-reference/functions/object_construct_keep_null.html) | Constructs an object and keeps null values. |
| [DATE_TRUNC](https://docs.snowflake.com/en/sql-reference/functions/date_trunc.html) | Truncates a date or timestamp to a specified precision. |
| [TIMESTAMP_LTZ](https://docs.snowflake.com/en/sql-reference/data-types-datetime#timestamp-ltz-timestamp-ntz-timestamp-tz) | Returns the current timestamp in local time zone. |
| [AS_DOUBLE](https://docs.snowflake.com/en/sql-reference/functions/as_double-real) | Converts a value to double precision. |
| [AS_INTEGER](https://docs.snowflake.com/en/sql-reference/functions/as_integer.html) | Converts a value to an integer. |
| [AS_NUMBER](https://docs.snowflake.com/en/sql-reference/functions/as_decimal-number) | Converts a value to a numeric type. |
| [AS_CHAR](https://docs.snowflake.com/en/sql-reference/functions/as_char-varchar) | Converts a value to a character string. |

### New Web User Interface

We have released a new Web User Interface for LocalStack for Snowflake. This interface is accessible via a web browser and provides a user-friendly experience for managing your local Snowflake resources. The interface includes a dashboard that allows you to:

-   Run Snowflake SQL queries and view the results using a Query Editor.
-   View detailed request/response traces of API calls made to Snowflake.
-   Forward queries from the Snowflake emulator to a real Snowflake instance using a proxy.

You can access the Web User Interface by navigating to [https://snowflake.localhost.localstack.cloud](https://snowflake.localhost.localstack.cloud) in your web browser.

// picture of the new web UI

The Web User Interface is still in **preview**, and we are actively working on adding new features and improving the user experience. Learn more about the new Web User Interface in our [documentation](https://snowflake.localstack.cloud/user-guide/web-user-interface/).

### Support for LocalStack Ephemeral Instances

We have launched Ephemeral Instances, enabling you to run a LocalStack for Snowflake sandbox in the cloud rather than on your local machine. This ephemeral environment is a short-lived, encapsulated deployment of the Snowflake emulator in the cloud. With these sandboxes, you can run tests, preview features in your Snowflake applications, and collaborate asynchronously within and across your team!

// picture

After launching an ephemeral instance, you can switch the Snowflake host in your application to the ephemeral instance's hostname. This change enables you to interact with the Snowflake emulator running in the cloud as if it were on your local machine. Additionally, Ephemeral Instances allow you to generate a preview environment from GitHub Pull Request (PR) builds.

The feature is in **preview**, and you can learn more about it in our [documentation](https://snowflake.localstack.cloud/user-guide/ephemeral-instances/).

### Improved parity with Snowflake

We have made several enhancements to LocalStack for Snowflake to improve compatibility with the Snowflake service. These improvements include:

-   Metadata queries and schema lookups can now utilize fully qualified table names.
-   Support for ID placeholders in JDBC prepared statements to incorporate query parameters.
-   Functions returning VARIANT values, including those that encode dictionary and list values, are now supported.
-   Enhanced parsing of parameters in `CREATE STAGE` statements now supports alternative formats such as `s3compat://...`.

### Support for Snowpipe

Snowpipe enables data loading into Snowflake tables from files in an external stage, continuously processing files as they become available. It uses a queue to manage this near real-time data loading.

LocalStack for Snowflake now includes support for Snowpipe, allowing you to create and manage Snowpipe objects within the emulator. This functionality lets you load data into Snowflake tables from files stored either in a local directory or an S3 bucket, both locally and remotely. The supported SQL statements for managing Snowpipe include:

-   [`CREATE PIPE`](https://docs.snowflake.com/en/sql-reference/sql/create-pipe.html)
-   [`DESCRIBE PIPE`](https://docs.snowflake.com/en/sql-reference/sql/describe-pipe.html)
-   [`DROP PIPE`](https://docs.snowflake.com/en/sql-reference/sql/drop-pipe.html)
-   [`SHOW PIPES`](https://docs.snowflake.com/en/sql-reference/sql/show-pipes.html)

Learn more about Snowpipe support in our [documentation](https://snowflake.localstack.cloud/user-guide/snowpipe/).

### Support for Iceberg tables

Iceberg tables utilize the Apache Iceberg open table format specification, providing an abstraction layer over data files stored in open formats. In Snowflake, Iceberg tables enable schema evolution, partitioning, and snapshot isolation for efficient table data management.

LocalStack for Snowflake now includes support for Iceberg tables, enabling you to create and manage these tables locally. You can use Iceberg tables to query data in Snowflake using the Iceberg format, with data stored in external volumes or local/remote S3 buckets.

For more detailed information on using Iceberg tables, refer to our [documentation](https://snowflake.localstack.cloud/user-guide/iceberg-tables/).

### Support for Hybrid tables
Snowflake Hybrid tables, also known as Unistore hybrid tables, facilitate fast, single-row operations by enforcing unique constraints on primary keys and incorporating indexes to expedite data retrieval. These tables are tailored to support both analytical and transactional workloads concurrently, forming the backbone of Snowflake's Unistore architecture.

LocalStack for Snowflake now includes support for Hybrid tables, enabling the creation and management of these tables locally. The supported SQL statements for managing Hybrid tables include:

-   [`CREATE HYBRID TABLE`](https://docs.snowflake.com/en/sql-reference/sql/create-hybrid-table.html)
-   [`DROP HYBRID TABLE`](https://docs.snowflake.com/en/sql-reference/sql/drop-hybrid-table.html)
-   [`SHOW HYBRID TABLES`](https://docs.snowflake.com/en/sql-reference/sql/show-hybrid-tables.html)

For more detailed information on using Hybrid tables, refer to our [documentation](https://snowflake.localstack.cloud/user-guide/hybird-tables/).

### Support for Dynamic Tables

Snowflake Dynamic Tables allow a background process to continuously load new data from sources into the table, accommodating both delta and full load operations. These tables automatically update to reflect query results, eliminating the need for a separate target table and custom data transformation code. They are regularly updated through scheduled refreshes by an automated process.

LocalStack for Snowflake now supports Dynamic tables, enabling you to create and manage them locally. The supported SQL statements for managing Dynamic tables include:

* [`CREATE DYNAMIC TABLE`](https://docs.snowflake.com/en/sql-reference/sql/create-dynamic-table.html)
* [`DROP DYNAMIC TABLE`](https://docs.snowflake.com/en/sql-reference/sql/drop-dynamic-table.html)

For more detailed information on using Dynamic tables, refer to our [documentation](https://snowflake.localstack.cloud/user-guide/dynamic-tables/).

## Conclusion
