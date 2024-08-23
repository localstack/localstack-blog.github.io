---
title: LocalStack Release v3.7.0
description: LocalStack Release v3.7.0
lead: LocalStack Release v3.7.0
date: 2024-08-23T3:59:43+05:30
lastmod: 2024-08-23T3:59:43+05:30
images: []
contributors: []
tags: ['news']
---

## Introduction

## What’s new in LocalStack 3.7.0?

### State merging for Cloud Pods (**Teams & Enterprise**)

LocalStack Cloud Pods now offer different strategies for state merging into your LocalStack container. The available strategies include:

- `overwrite`: This strategy deletes the existing state and replaces it with the new state from the Cloud Pod, effectively resetting the LocalStack state.
- `account-region-merge` (**default**): This strategy merges services by account and region pairs, combining states from both the existing and the Cloud Pod for the same account and region.
- `service-merge`: This strategy merges services at the account-region level, assuming no resource overlap exists. It gives priority to the resources from the loaded state during the merge.

To activate these strategies, use `--strategy <strategy>` when loading the Cloud Pod, where `<strategy>` is one of the strategies mentioned above. For example, to use the `service-merge` strategy, run:

```bash
localstack pod load my-test-pod --strategy service-merge
```

To learn more about how it works, check out the [Cloud Pods documentation](https://docs.localstack.cloud/user-guide/state-management/cloud-pods/#example-scenario).

### Support for DMS Serverless (Enterprise)

LocalStack now supports AWS Database Migration Service (DMS) Serverless. DMS Serverless can be used with sources and targets supported by both LocalStack and AWS. To simulate the different states that a replication config experiences during provisioning, you can set the `DMS_SERVERLESS_STATUS_CHANGE_WAITING_TIME` environment variable. This setting delays each state change for the configured number of seconds until the replication is actively running. Refer to the official documentation for details on the different states.

Note that on AWS, replication table statistics are automatically deleted after the replication finishes and the configuration is deprovisioned. For parity, the same applies in LocalStack. To delay this deprovisioning process, you can set the `DMS_SERVERLESS_DEPROVISIONING_DELAY` environment variable, which defaults to 60 seconds.

Learn more about DMS in the [LocalStack documentation](https://docs.localstack.cloud/user-guide/aws/dms).

### Chaos Engineering dashboard now uses Chaos API (Enterprise)

The LocalStack Chaos Engineering dashboard now leverages the Chaos API to provide the following features:

-   **DynamoDB Error**: Injects `ProvisionedThroughputExceededException` errors randomly into DynamoDB API responses.
-   **Kinesis Error**: Injects `ProvisionedThroughputExceededException` errors randomly into Kinesis API responses.
-   **500 Internal Error**: Terminates incoming requests randomly, returning an `Internal Server Error` with a response code of 500.
-   **Service Unavailable**: Causes a specific percentage of service API calls to receive a 503 `Service Unavailable` response.
-   **AWS Region Unavailable**: Simulates regional outages and failovers by disabling entire AWS regions.
-   **Latency**: Adds specified latency to every API call, useful for simulating network latency or degraded network performance.

{{< img-simple src="chaos-engineering-dashboard-chaos-api.png" alt="Chaos Engineering dashboard using the Chaos API" width="600">}}

Learn more about the Chaos Engineering Dashboard in the [LocalStack documentation](https://docs.localstack.cloud/user-guide/chaos-engineering/chaos-application-dashboard/).

### Support for fetching LocalStack logs for Ephemeral Instances (Pro)

Ephemeral Instances in LocalStack now support fetching logs from the LocalStack container. You can navigate to the [Ephemeral Instances page](https://app.localstack.cloud/instances/ephemeral) on the [LocalStack Web Application](https://app.localstack.cloud) and click on the **Logs** tab to view the logs for the running instance.

{{< img-simple src="ephemeral-instances-logs.png" alt="Fetching logs for Ephemeral Instances" width="600">}}

The logs are not automatically streamed, but you can manually refresh them by clicking the **Refresh** icon on the **Log output** window. To learn more about Ephemeral Instances, check out the [LocalStack documentation](https://docs.localstack.cloud/user-guide/cloud-sandbox/ephemeral-instance).

### Extensions interface is now embedded in the Web Application (Pro)

The LocalStack Extensions interface is now embedded in the LocalStack Web Application. You can now use extensions, such as AWS Replicator and Mailhog, directly from our Web Application, consolidating all necessary tools in one interface. You can access the Extensions library by navigating to the **Extensions** section on the Web Application.

{{< img-simple src="embedded-extension-web-application-interface.png" alt="LocalStack Extensions interface in the Web Application" width="600">}}

To learn more about LocalStack Extensions, check out the [LocalStack documentation](https://docs.localstack.cloud/user-guide/extensions).

### Support for SSE-C validation in S3

LocalStack's S3 provider now supports SSE-C parameter validation for the following S3 APIs:

-   [**`PutObject`**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObject.html)
-   [**`GetObject`**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetObject.html)
-   [**`HeadObject`**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_HeadObject.html)
-   [**`GetObjectAttributes`**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetObjectAttributes.html)
-   [**`CopyObject`**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CopyObject.html)
-   [**`CreateMultipartUpload`**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CreateMultipartUpload.html)
-   [**`UploadPart`**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_UploadPart.html)

However, it's important to note that LocalStack does not support the actual encryption and decryption of objects using SSE-C. Learn more about S3 in the [LocalStack documentation](https://docs.localstack.cloud/user-guide/aws/s3).

### Tagging operations in the EventBridge Pipes provider (Pro)

The following tagging operations for EventBridge Pipes have now been implemented:

-   [`TagResource`](https://docs.aws.amazon.com/eventbridge/latest/pipes-reference/API_TagResource.html)
-   [`UntagResource`](https://docs.aws.amazon.com/eventbridge/latest/pipes-reference/API_UntagResource.html)
-   [`ListTagsForResource`](https://docs.aws.amazon.com/eventbridge/latest/pipes-reference/API_ListTagsForResource.html)

Learn more about EventBridge in the [LocalStack documentation](https://docs.localstack.cloud/user-guide/aws/pipes).

### New enhancements in the CloudFormation provider

LocalStack's CloudFormation provider now supports the following enhancements:

- Users can now specify MFA configurations directly in the resource definition for `AWS::Cognito::UserPool`.
- `ArchivePolicy` settings are now supported in `AWS::SNS::Topic` resources.
- Default tags can now be configured for `AWS::EC2::SecurityGroup`.
