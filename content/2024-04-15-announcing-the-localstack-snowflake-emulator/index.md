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

We’re excited to announce that we have released a first version of the LocalStack Snowflake emulator. The LocalStack Snowflake emulator allows you to develop and test your data pipelines locally and test your data pipelines in CI using a Docker container which can be plugged into your Snowflake integrations, such as [Snowpark](https://docs.snowflake.com/en/developer-guide/snowpark/index), [client libraries](https://developers.snowflake.com/drivers-and-libraries/), among others. The Snowflake emulator supports various key features, such as:

-   Operations on warehouses, databases, schemas, and tables.
-   Storing files in [Snowflake stages](https://docs.snowflake.com/en/user-guide/data-load-local-file-system-create-stage).
-   [Tasks](https://docs.snowflake.com/en/user-guide/tasks-intro) for scheduled execution.
-   Snowpipe streaming with [Kafka connector](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-streaming-kafka).
-   JavaScript and Python [UDFs](https://docs.snowflake.com/en/developer-guide/udf/udf-overview).
-   [Table streams](https://docs.snowflake.com/en/user-guide/streams-intro) for change data capture (CDC) and audit logs.
-   Cross-Database [Resource Sharing](https://docs.snowflake.com/en/user-guide/data-sharing-intro).
-   Basic support to run [Streamlit](https://docs.snowflake.com/en/developer-guide/streamlit/about-streamlit) applications locally.
-   Infrastructure-as-code with [Terraform](https://snowflake.localstack.cloud/user-guide/integrations/terraform/) & [Pulumi](https://snowflake.localstack.cloud/user-guide/integrations/pulumi/).
-   [Continuous integration](https://snowflake.localstack.cloud/user-guide/continuous-integration/) with GitHub Actions, CircleCI & GitLab CI.
- Enterprise-tested features like [Cloud Pods and Persistence](https://snowflake.localstack.cloud/user-guide/state-management/) for state snapshots.

This allows you to bypass the need to rely on the live version for local development & testing while enabling a high velocity, high quality, agile test-driven development for your data applications. With this release, we’re underscoring our commitment to go multi-cloud and build a complete suite of emulators that allow you to achieve efficiency and cost savings by putting development and testing closer together.

The Snowflake emulator is currently in public preview, and you can reach out to us to get access! This post explores how we reached this point, what it means for our users, and provides a quickstart to help you get started.

Navigate to our [LocalStack Snowflake emulator documentation](https://snowflake.localstack.cloud/introduction/) to get started!

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

## How do I start?

To get started with the LocalStack Snowflake emulator, pull our Docker image from DockerHub:

```bash
docker pull localstack/snowflake:latest
```

You can start the emulator using the `localstack` CLI after exporting your LocalStack Auth Token (`LOCALSTACK_AUTH_TOKEN`) in your terminal session:

```bash
export LOCALSTACK_AUTH_TOKEN=<your-auth-token>
IMAGE_NAME=localstack/snowflake localstack start
```

It will start the emulator on snowflake.localhost.localstack.cloud, which is a DNS name that resolves to a local IP address (127.0.0.1) to make sure the connector interacts with the local APIs.

If you’re using [Snowflake Drivers](https://docs.snowflake.com/en/developer-guide/drivers), such as the Snowflake Connector for Python, you can use the following code to connect to the local Snowflake instance:

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

Similarly, you can utilize the JDBC driver to connect to the Snowflake emulator from your preferred DB visualization tool (see more details in our [documentation](https://snowflake.localstack.cloud/introduction/)).

You can create a warehouse named `test_warehouse`, a database named `testdb`, and a schema named `testschema`:

```python
conn.cursor().execute("CREATE WAREHOUSE IF NOT EXISTS test_warehouse")
conn.cursor().execute("CREATE DATABASE IF NOT EXISTS testdb") conn.cursor().execute("USE DATABASE testdb")
conn.cursor().execute("CREATE SCHEMA IF NOT EXISTS testschema")
```

You can navigate to the LocalStack logs via localstack logs to see the Snowflake emulator in action. To connect your existing Snowflake app to the emulator, all you need to do is add the Snowflake Host name as `snowflake.localhost.localstack.cloud` while specifying mock credentials for your Snowflake user, password, and account. 

Note that LocalStack at no point talks to the real Snowflake instance — Everything runs locally, giving you the full power and flexibility to develop and test your data applications locally, without depending on real cloud resources.

For a more detailed, real-world example check out our sample application on GitHub (WIP).

## Next steps

As an early adopter of the LocalStack Snowflake emulator, we’ll be banking on essential feedback as we continue to push out new features & enhancements. The foundation for our next steps in the development of the Snowflake emulator lies in **parity**, **performance**, and **developer experience**, and our promise to provide the best tooling to empower data engineers across the entire software development lifecycle (SDLC).

Stay tuned for more news and awesome features in the upcoming months! We're diligently refining and enhancing our current offering, and here are some features you can expect in the upcoming months:

-   Support for data persistence & state snapshots with Cloud Pods
-   Full emulation of DB roles, role-based access control, and row-level security policies
-   Enhanced support for table streams and CDC use cases
-    Advanced integration with other storage/streaming cloud services in LocalStack (AWS Glue, Kinesis Firehose, S3, AppFlow, etc)
-   Tooling for test data management and preseeding the emulator with data from a real Snowflake instance

We are excited to have the privilege of working with our community to accelerate cloud and data development processes. LocalStack is poised to change the cloud development landscape, and we are looking forward to your continued support and feedback!

## Learn more

-   Check out our demo video on running Snowflake applications locally.
-   Watch the webinar on testing data pipelines with the Snowflake emulator.
-   Navigate to our [documentation](https://snowflake.localstack.cloud/introduction/) & [tutorials](https://snowflake.localstack.cloud/tutorials/) to try out various features.
-   Have questions? Join the [LocalStack Slack Community](https://localstack.cloud/slack) to get help.

New to LocalStack? Create a [free account today](https://app.localstack.cloud/sign-up) and [get started](https://snowflake.localstack.cloud/getting-started/installation/)!
