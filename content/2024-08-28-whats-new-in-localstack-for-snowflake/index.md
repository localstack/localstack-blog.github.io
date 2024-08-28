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

- [New Web User Interface](#new-web-user-interface) (**Preview**)
- [Support for LocalStack Ephemeral Instances](#support-for-localstack-ephemeral-instances) (**Preview**)
- [Improved parity with Snowflake](#improved-parity-with-snowflake)
- [Support for Snowpipe](#support-for-snowpipe)
- [Support for Iceberg tables](#support-for-iceberg-tables)

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


## Conclusion
