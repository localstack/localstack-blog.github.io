---
title: Announcing the LocalStack Snowflake emulator!
description: We are excited to announce the LocalStack Snowflake emulator which enables a high-fidelity, fully local Snowflake experience to develop & test your data pipelines.
lead: We are excited to announce the LocalStack Snowflake emulator, which provides a high-fidelity, completely local Snowflake experience for developing and testing your data pipelines.
date: 2024-04-15T5:41:46+05:30
lastmod: 2024-04-15T5:41:46+05:30
images: []
contributors: ["LocalStack Team"]
tags: ['news']
show_cta_1: true
---

## Introduction

We are excited to announce that we have released a first version of the LocalStack Snowflake emulator. The LocalStack Snowflake emulator allows you to develop and test your data pipelines locally and CI pipelines, using a Docker container which can be easily plugged into your Snowflake integrations, such as [Snowpark](https://docs.snowflake.com/en/developer-guide/snowpark/index), [client libraries](https://developers.snowflake.com/drivers-and-libraries/), among others. The Snowflake emulator supports various key features, such as:

-   DDL/DML/DQL operations on **warehouses**, **databases**, **schemas**, and **tables**;
-   Storing files in [Snowflake **stages**](https://docs.snowflake.com/en/user-guide/data-load-local-file-system-create-stage);
-   [**Tasks**](https://docs.snowflake.com/en/user-guide/tasks-intro) for scheduled execution;
-   Snowpipe **streaming** with [Kafka connector](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-streaming-kafka);
-   [User-defined **functions**](https://docs.snowflake.com/en/developer-guide/udf/udf-overview) (UDFs) in JavaScript and Python;
-   [Table **streams**](https://docs.snowflake.com/en/user-guide/streams-intro) for change data capture (CDC) and audit logs;
-   Cross-database [**resource sharing**](https://docs.snowflake.com/en/user-guide/data-sharing-intro);
-   Basic support to run [**Streamlit**](https://docs.snowflake.com/en/developer-guide/streamlit/about-streamlit) applications locally;
-   Infrastructure-as-code with [**Terraform**](https://snowflake.localstack.cloud/user-guide/integrations/terraform/) & [**Pulumi**](https://snowflake.localstack.cloud/user-guide/integrations/pulumi/);
-   Test your data applications in [**Continuous Integration**](https://snowflake.localstack.cloud/user-guide/continuous-integration/) (CI) pipelines with GitHub Actions, CircleCI & GitLab CI (among others);
- Enterprise-tested features like [**Cloud Pods** and **Persistence**](https://snowflake.localstack.cloud/user-guide/state-management/) for state snapshots.

This allows you to bypass the need to rely on the live version for local development and testing, while also enabling high-velocity, high-quality, agile test-driven development for your data applications. With this release, we are demonstrating our commitment to go multi-cloud and build a complete suite of developer tools that will allow you to achieve efficiency and cost savings by bringing development and testing closer together.

The Snowflake emulator is currently in **public preview**, and [you can reach out to us](https://localstack.cloud/contact) to get access! This blog explores how we've reached this important milestone, outlines what it means for our users, and provides a quick introduction to help you get started.

For guidance, please navigate to our [**LocalStack Snowflake emulator documentation**](https://snowflake.localstack.cloud/introduction/).

## How did we get here?

Software development organizations need a fast build lifecycle, quick continuous integration workflows, and a smoother developer experience — all this while optimizing cost, putting security in the forefront, and testing all application logic at the same time. However, adopting modern cloud-based solutions has been challenging due to a slow dev&test loop that massively undercuts the promise given to us by proprietary providers. The centralized, remote execution model of cloud providers is a limitation & liability, as developers continue to juggle around live development environments, long build times, inefficient CI pipelines, and cumbersome developer experience.

LocalStack has responded to these challenges by providing a robust cloud emulator that supports over 100 AWS services, allowing for integration tests to be run locally and in CI environments. This capability has positioned LocalStack as a critical tool for developers seeking to improve efficiency and reduce dependencies on remote cloud environments.

As LocalStack's user base expanded, so did the demand for similar capabilities with Snowflake. Although a [local testing framework](https://docs.snowflake.com/en/developer-guide/snowpark/python/testing-locally) from Snowflake is available, it only provides mock support for running integration tests, which falls short for more complex use cases. Leveraging the existing toolset in the LocalStack core cloud emulator — including our RDS Postgres utilities, snapshot testing library, analytics service client, and more — allowed us to build an initial experimental preview. This new extension was [announced on our Discuss forum](https://discuss.localstack.cloud/t/introducing-the-localstack-snowflake-extension-experimental/665/7), where it quickly gained significant traction within the community.

## How did we build this?

At its core, we utilize PostgreSQL as the database engine to store the user data and execute queries. The SQL syntax of Snowflake queries is overall fairly similar to PostgreSQL, but there are several more or less subtle differences. The figure below outlines some of the main components used in our implementation:

// picture

Query Processors are the main building blocks that collectively comprise the core engine that processes incoming user queries. We distinguish 3 main types of query processors:

-   DB initializers are pieces of logic that are applied only once upon creation of a database (e.g., creating custom SQL functions).
-   Query pre-processors operate on the abstract syntax tree (AST) of SQL queries and transform incoming queries in Snowflake format to target queries that can be executed in the DB engine (Postgres).
-   Result post-processors take care of applying additional custom logic and converting the DB engine query results to Snowflake API-compatible result sets — either as JSON blobs or in Apache Arrow table format.

Auxiliary Services encompass additional pieces of logic to handle file stages, session states, table streams, tasks, as well as other integrations and functions.

## How do I start?

To get started with the LocalStack Snowflake emulator, pull our Docker image from DockerHub:

```bash
docker pull localstack/snowflake:latest
```

You can start the emulator using the `localstack` CLI after exporting your [LocalStack Auth Token](https://docs.localstack.cloud/getting-started/auth-token/) (`LOCALSTACK_AUTH_TOKEN`) in your terminal session:

```bash
export LOCALSTACK_AUTH_TOKEN=<your-auth-token>
IMAGE_NAME=localstack/snowflake localstack start
```

This command starts the emulator on `snowflake.localhost.localstack.cloud`, a DNS name that resolves to the local IP address 127.0.0.1. This setup ensures that the connector interacts seamlessly with the local APIs.

If you’re using [Snowflake Drivers](https://docs.snowflake.com/en/developer-guide/drivers), such as the [Snowflake Connector for Python](https://docs.snowflake.com/en/developer-guide/python-connector/python-connector), you can use the following code to connect to the local Snowflake instance:

```python
import snowflake.connector as sf

conn = sf.connect(
    user="test",
    password="test",
    account="test",
    database="test",
    host="snowflake.localhost.localstack.cloud",
)
```

Similarly, you can utilize the JDBC driver to connect to the Snowflake emulator from your preferred database visualization tool.

Once connected, you can set up your development environment by executing commands to establish core components. Create a warehouse named `test_warehouse`, a database named `testdb`, and a schema named `testschema` with the following commands:

```python
conn.cursor().execute("CREATE WAREHOUSE IF NOT EXISTS test_warehouse")
conn.cursor().execute("CREATE DATABASE IF NOT EXISTS testdb") 
conn.cursor().execute("USE DATABASE testdb")
conn.cursor().execute("CREATE SCHEMA IF NOT EXISTS testschema")
```

You can navigate to the LocalStack logs via `localstack logs` to see the Snowflake emulator in action. To connect your existing Snowflake app to the emulator, all you need to do is add the Snowflake Host name as `snowflake.localhost.localstack.cloud` while specifying mock credentials for your Snowflake user, password, and account.

Note that LocalStack at no point talks to the real Snowflake instance — Everything runs locally, giving you the full power and flexibility to develop and test your data applications without relying on real cloud resources.

For more detailed, real-world examples check out our [**Snowflake samples repository**](https://github.com/localstack-samples/localstack-snowflake-samples) on GitHub to explore a variety of use cases that you can run locally.

## Next steps

The foundation for our next steps in the development of the Snowflake emulator lies in **parity**, **performance**, and **developer experience**, and our promise to provide the best tooling to empower data engineers across the entire software development lifecycle (SDLC).

Join our preview program today - we're working very closely with our community of early adopters to understand your use cases, and we can prioritize feature development during this current stage of development, to ensure our implementation properly supports your requirements. Here are some features you can expect in the upcoming months:

-   Full emulation of database roles, role-based access control, and row-level security policies
-   Enhanced support for table streams and CDC use cases.
-   Advanced integration with other storage/streaming cloud services in LocalStack (AWS Glue, Kinesis Firehose, S3, AppFlow, etc).
-   Tooling for test data management and preseeding the emulator with data from a real Snowflake instance.
-   A Web user experience to inspect the state of your local Snowflake resources and help with common day-to-day tasks.
-   A connection proxy that allows mirroring data from real Snowflake cloud into the local emulator, to easily flip the switch between local and remote query execution.

We are excited to have the privilege of working with our community to accelerate cloud and data development processes. LocalStack is poised to change the cloud development landscape, and we are looking forward to your continued support and feedback!

## Learn more

-   Watch the [webinar recording](https://youtu.be/fWYRfuNMxuU) on testing data pipelines with the Snowflake emulator.
-   Navigate to our [documentation](https://snowflake.localstack.cloud/introduction/) & [tutorials](https://snowflake.localstack.cloud/tutorials/) to try out the various features.
-   Have questions? Join the [LocalStack Slack Community](https://localstack.cloud/slack) to get help.

New to LocalStack? Create a [free account today](https://app.localstack.cloud/sign-up) and [get started](https://snowflake.localstack.cloud/getting-started/installation/)!
