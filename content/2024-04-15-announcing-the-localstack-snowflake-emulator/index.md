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
