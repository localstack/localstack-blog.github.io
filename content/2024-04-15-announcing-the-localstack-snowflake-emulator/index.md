---
title: Announcing the LocalStack Snowflake emulator!
description: We are excited to announce the LocalStack Snowflake emulator which enables a high-fidelity, fully local Snowflake experience to develop & test your data pipelines.
lead: We are excited to announce the LocalStack Snowflake emulator which enables a high-fidelity, fully local Snowflake experience to develop & test your data pipelines.
date: 2024-04-15T5:41:46+05:30
lastmod: 2024-04-15T5:41:46+05:30
images: []
contributors: ["LocalStack Team"]
tags: ['news']
---

## Introduction

We’re excited to announce that we have released a first version of the LocalStack Snowflake emulator. The LocalStack Snowflake emulator allows you to develop and test your data pipelines locally using a Docker container which can be plugged into integrations, such as [Snowpark](https://docs.snowflake.com/en/developer-guide/snowpark/index), [client libraries](https://developers.snowflake.com/drivers-and-libraries/), and various other toolings in the [Snowflake ecosystem](https://docs.snowflake.com/en/user-guide/ecosystem). The Snowflake emulator supports various key features, such as:

-   Basic operations on warehouses, databases, schemas, and tables.
-   Storing files in [user/data/named stages](https://docs.snowflake.com/en/user-guide/data-load-local-file-system-create-stage).
-   [Tasks](https://docs.snowflake.com/en/user-guide/tasks-intro) for scheduled execution.
-   Snowpipe streaming with [Kafka connector](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-streaming-kafka).
-   JavaScript and Python [UDFs](https://docs.snowflake.com/en/developer-guide/udf/udf-overview).
-   [Table streams](https://docs.snowflake.com/en/user-guide/streams-intro) for change data capture (CDC) and audit logs.
-   Cross-Database [Resource Sharing](https://docs.snowflake.com/en/user-guide/data-sharing-intro).
-   Testing infrastructure-as-code with [Terraform](https://snowflake.localstack.cloud/user-guide/integrations/terraform/) & [Pulumi](https://snowflake.localstack.cloud/user-guide/integrations/pulumi/).
-   [Continuous integration](https://snowflake.localstack.cloud/user-guide/continuous-integration/) with GitHub Actions, CircleCI & GitLab CI.

This allows you to bypass the need to rely on the live version for local development & testing while enabling a high velocity, high quality, agile test-driven development for your data applications. With this release, we’re underscoring our commitment to go multi-cloud and build a complete suite of emulators that allow you to achieve efficiency and cost savings by putting development and testing closer together.

The Snowflake emulator is currently in public preview, and you can reach out to us to get access! This post explores how we reached this point, what it means for our users, and provides a quickstart to help you get started.

TL;DR — Navigate to our [LocalStack Snowflake emulator documentation](https://snowflake.localstack.cloud/introduction/) to get started!

## How did we get here?

Software development organizations need a fast build lifecycle, quick continuous integration workflows, and a smoother developer experience — all this while optimizing cost, putting security in the forefront, and testing all application logic at the same time. However, adopting modern cloud-based solutions has been challenging due to a slow dev&test loop that massively undercuts the promise given to us by proprietary providers. The centralized, remote execution model of cloud providers is a limitation & liability, as developers continue to juggle around live development environments, long build times, inefficient CI pipelines, and cumbersome developer experience.

The LocalStack platform addresses these challenges head-on! LocalStack’s core cloud emulator supports over 100 AWS services, such as Lambda, S3, EKS, ECS, RDS, and more. This enables a high-fidelity local developer environment ​​that allows you to run integration tests of cloud solutions locally and in CI environments. While expanding to 100K users worldwide and over 220M+ Docker pulls, we gathered a lot of learnings that further expanded our vision of providing a comprehensive developer platform that facilitates local multi-cloud development across different providers and services.

The demand for a local version of the Snowflake has been on the rise, across [Stack Overflow](https://stackoverflow.com/questions/75820814/is-it-possible-to-run-a-local-snowflake-instance-using-docker), the [Snowflake community](https://community.snowflake.com/s/question/0D50Z00008Li3DTSAZ/has-anyone-come-up-with-a-local-version-of-snowflake-that-allows-for-development-testing-locally), and [Twitter/X](https://twitter.com/criccomini/status/1618782276267163652). A [local testing framework](https://docs.snowflake.com/en/developer-guide/snowpark/python/testing-locally) from Snowflake is available, which provides mock-only support for running integration tests - but that isn’t enough for more expansive use cases. The existing tooling available in the LocalStack core cloud emulator, which included our RDS Postgres utilities, snapshot testing library, analytics service client, and more allowed us to build an initial experimental preview [which was announced on our Discuss forum](https://discuss.localstack.cloud/t/introducing-the-localstack-snowflake-extension-experimental/665/7), gaining significant community traction.

## How did we build this?

At its core, we utilize PostgreSQL as the database engine to store the user data and execute queries. The SQL syntax of Snowflake queries is overall fairly similar to PostgreSQL, but there are several more or less subtle differences. The figure below outlines some of the main components used in our implementation:

// picture

Query Processors are the main building blocks that collectively comprise the core engine that processes incoming user queries. We distinguish 3 main types of query processors:

-   DB initializers are pieces of logic that are applied only once upon creation of a database (e.g., creating custom SQL functions).
-   Query pre-processors operate on the abstract syntax tree (AST) of SQL queries and transform incoming queries in Snowflake format to target queries that can be executed in the DB engine (Postgres).
-   Result post-processors take care of applying additional custom logic and converting the DB engine query results to Snowflake API-compatible result sets — either as JSON blobs or in Apache Arrow table format.

Auxiliary Services encompass additional pieces of logic to handle file stages, session states, table streams, tasks, as well as other integrations and functions.