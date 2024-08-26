---
title: LocalStack Release v3.7.0
description: LocalStack 3.7.0 is now available! This minor release introduces several new features, enhancements, and bug fixes, with a focus on improved parity with AWS, new DevX features, and enhanced local developer tooling.
lead: LocalStack 3.7.0 is now available! This minor release introduces several new features, enhancements, and bug fixes, with a focus on improved parity with AWS, new DevX features, and enhanced local developer tooling.
date: 2024-08-23T3:59:43+05:30
lastmod: 2024-08-23T3:59:43+05:30
images: []
contributors: ['Harsh Mishra']
tags: ['news']
---

## Introduction

LocalStack 3.7.0 is now available! This minor release introduces several new features, enhancements, and bug fixes. The highlights include a new Lambda Debug Mode, state merging for Cloud Pods, support for DMS Serverless, and a new Lambda Event Source Mapping implementation. 

Additionally, the Chaos Engineering dashboard now uses the Chaos API, and LocalStack now supports fetching logs for Ephemeral Instances. The Extensions interface is now embedded in the Web Application, and the CloudFormation provider has received new enhancements. The EC2 Libvirt VM manager and SES provider have also been updated with new features.

## How to upgrade?

To upgrade to LocalStack 3.7.0 using the LocalStack CLI, run the following command to update both the LocalStack Docker image and CLI to the latest version:

```bash
$ localstack update all
```

If using LocalStack with Docker CLI or Docker Compose, update the Docker image by running:

```bash
$ docker pull localstack/localstack:3.7.0 # Community Edition
$ docker pull localstack/localstack-pro:3.7.0 # Pro Edition
```

Pin the LocalStack version in your `docker run` command or Docker Compose file to `3.7.0`.

## What’s new in LocalStack 3.7.0?

- [New Lambda Debug Mode](#new-lambda-debug-mode)
- [State merging for Cloud Pods](#state-merging-for-cloud-pods-teams--enterprise)
- [Dry run for loading Cloud Pods](#dry-run-for-loading-cloud-pods-teams--enterprise)
- [Support for DMS Serverless](#support-for-dms-serverless-enterprise)
- [Chaos Engineering dashboard now uses Chaos API](#chaos-engineering-dashboard-now-uses-chaos-api-enterprise)
- [Support for fetching LocalStack logs for Ephemeral Instances](#support-for-fetching-localstack-logs-for-ephemeral-instances-pro)
- [Extensions interface is now embedded in the Web Application](#extensions-interface-is-now-embedded-in-the-web-application-pro)
- [New Lambda Event Source Mapping implementation](#new-lambda-event-source-mapping-implementation-preview)
- [Support for SSE-C validation in S3](#support-for-sse-c-validation-in-s3)
- [New `EKS_K8S_PROVIDER` environment variable](#new-eks_k8s_provider-environment-variable-pro)
- [Tagging operations in the EventBridge Pipes provider](#tagging-operations-in-the-eventbridge-pipes-provider-pro)
- [New enhancements in the CloudFormation provider](#new-enhancements-in-the-cloudformation-provider)
- [New template option for the LocalStack Extensions CLI](#new-template-option-for-the-localstack-extensions-cli-pro)
- [New enhancements in the EC2 Libvirt VM manager](#new-enhancements-in-the-ec2-libvirt-vm-manager-pro)
- [New EC2 Kubernetes executor](#new-ec2-kubernetes-executor-enterprise-preview)
- [New enhancements in the SES provider](#new-enhancements-in-the-ses-provider)
- [Miscellaneous](#miscellaneous)

### New Lambda Debug Mode

WIP

### State merging for Cloud Pods (Teams & Enterprise)

LocalStack Cloud Pods now offer different strategies for state merging into your LocalStack container. The available strategies include:

- `overwrite`: This strategy deletes the existing state and replaces it with the new state from the Cloud Pod, effectively resetting the LocalStack state.
- `account-region-merge` (**default**): This strategy merges services by account and region pairs, combining states from both the existing and the Cloud Pod for the same account and region.
- `service-merge`: This strategy merges services at the account-region level, assuming no resource overlap exists. It gives priority to the resources from the loaded state during the merge.

To activate these strategies, use `--strategy <strategy>` when loading the Cloud Pod, where `<strategy>` is one of the strategies mentioned above. For example, to use the `service-merge` strategy, run:

```bash
$ localstack pod load my-test-pod --strategy service-merge
```

To learn more about how it works, check out the [Cloud Pods documentation](https://docs.localstack.cloud/user-guide/state-management/cloud-pods/#example-scenario).

### Dry run for loading Cloud Pods (Teams & Enterprise)

To preview the changes that would occur when loading a Cloud Pod, you can now use the `--dry-run` flag. The result will depend on the selected merge strategy. The result will be displayed in the console, and no changes will be made to the LocalStack state. Here is a quick example:

```bash
$ localstack pod load my-test-pod --dry-run

This load operation modifies one or more resources in the application runtime.
The result will depend on the selected merge strategy. Use the --help option to
read more about it.

This load operation will modify the runtime state as follows:

──────────────────────────── sns ────────────────────────────
+ 2 resources added.
~ 1 resources modified.

──────────────────────── cognito-idp ────────────────────────
+ 1 resources added.
~ 0 resources modified.

──────────────────────────── sqs ────────────────────────────
+ 1 resources added.
~ 1 resources modified.
```

### Support for DMS Serverless (Enterprise)

LocalStack now supports AWS Database Migration Service (DMS) Serverless. DMS serverless can be used for the sources and targets that LocalStack already supports, and that are also supported by AWS. To simulate the different states that a replication config experiences during provisioning, you can set the `DMS_SERVERLESS_STATUS_CHANGE_WAITING_TIME` environment variable. This setting delays each state change for the configured number of seconds until the replication is actively running. Refer to the official documentation for details on the different states.

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

### New Lambda Event Source Mapping implementation (Preview)

LocalStack now supports a Event Source Mapping (ESM) implementation. You can use the `LAMBDA_EVENT_SOURCE_MAPPING=v2` configuration variable to use the new ESM implementation. The ESM v2 implementation is also compatible with the Java-based event pattern rule engine (`EVENT_RULE_ENGINE=java`). However, the new ESM implementation is still in preview and may not support all features. The current limitations include:

-   Lambda Success Destinations are currently not supported.
-   Lambda Failure Destinations have only a basic implementation available, such as FIFO queues may not be supported, and there is no SNS target yet.
-   SQS Dead Letter Queue (DLQ) Messages do not match the AWS structure, using a `context` similar to EventBridge Pipes instead of `requestContext` and `responseContext`.
-   Streaming Pollers for Kinesis & DynamoDB do not implement features like:
    - `BisectBatchOnFunctionError`
    - `MaximumBatchingWindowInSeconds`
    - `ParallelizationFactor`
    - `ScalingConfig`
    - `TumblingWindowInSeconds`.
-   ESM Lifecycle State Updates only provide basic support for state updates, such as no failure states, and `LastProcessingResult` is not consistently updated.
-   Only very basic validations are performed upon creating and updating ESM.
-   Source Managed Streaming for Apache Kafka (MSK) is not yet supported.
-   Persistence is not yet supported.

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

### New `EKS_K8S_PROVIDER` environment variable (Pro)

A new configuration variable `EKS_K8S_PROVIDER` has been introduced with two options:

- `k3s` (**default**, using k3d) 
- `local` (using a mounted kubeconfig). 

The kubeconfig inside the LocalStack container at `/root/.kube/config` no longer influences provider selection, but a warning is issued if it's missing when using the `local` option. Kubeconfig files located on the host at `~/.kube/config` are not automatically mounted; users must manually mount them if necessary.

Learn more about EKS in the [LocalStack documentation](https://docs.localstack.cloud/user-guide/aws/eks).

### Tagging operations in the EventBridge Pipes provider (Pro)

The following tagging operations for EventBridge Pipes have now been implemented:

| API               | Notes                                  |
| :---------------------- | :------------------------------------- |
| [**`TagResource`**](https://docs.aws.amazon.com/eventbridge/latest/pipes-reference/API_TagResource.html)   | Adds or overwrites tags for a resource |
| [**`UntagResource`**](https://docs.aws.amazon.com/eventbridge/latest/pipes-reference/API_UntagResource.html) | Removes tags from a resource           |
| [**`ListTagsForResource`**](https://docs.aws.amazon.com/eventbridge/latest/pipes-reference/API_ListTagsForResource.html) | Lists tags for a resource              |


Learn more about EventBridge Pipes in the [LocalStack documentation](https://docs.localstack.cloud/user-guide/aws/pipes).

### New enhancements in the CloudFormation provider

LocalStack's CloudFormation provider now supports the following enhancements:

- Users can now specify MFA configurations directly in the resource definition for `AWS::Cognito::UserPool`.
- `ArchivePolicy` settings are now supported in `AWS::SNS::Topic` resources.
- Default tags can now be configured for `AWS::EC2::SecurityGroup`.
- Users can now import existing EC2 SSH key pairs using the `AWS::EC2::KeyPair` resource.

Learn more about CloudFormation in the [LocalStack documentation](https://docs.localstack.cloud/user-guide/aws/cloudformation).

### New template option for the LocalStack Extensions CLI (Pro)

Support has been added for the `--template` option in the `localstack extensions dev new` command. This option allows users to specify a template to use from the selection available at [LocalStack Extensions Templates](https://github.com/localstack/localstack-extensions/tree/main/templates). Currently, the available templates include:

- [`basic`](https://github.com/localstack/localstack-extensions/tree/main/templates/basic)
- [`react`](https://github.com/localstack/localstack-extensions/tree/main/templates/react)

To create a new extension using the `react` template, run:

```bash
$ localstack extensions dev new --template=react
```

The generated template will contain a simple Python distribution configuration, and some boilerplate extension code.

### New enhancements in the EC2 Libvirt VM manager (Pro)

The [EC2 Libvirt VM manager](https://docs.localstack.cloud/user-guide/aws/ec2/#libvirt-vm-manager) now supports the following enhancements:

- You can use the `EC2_HYPERVISOR_URI` configuration variable to set the Libvirt connection URI, specifying the hypervisor host for the EC2 Libvirt VM manager. Currently, only QEMU drivers are supported.
- You can set the `UserData` field of the [`RunInstances` API](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_RunInstances.html) to a shell script that will be executed at the time of VM startup in the EC2 Libvirt VM manager.

### New EC2 Kubernetes executor (Enterprise) (Preview)

You can now run EC2 instances on Kubernetes. You can do so by setting the `EC2_VM_MANAGER` environment variable to `kubernetes` in the LocalStack container. Each EC2 instance in the Kubernetes VM manager is backed by a Pod.

The following operations are supported:

| API            | Notes                                  |
| :------------------- | :------------------------------------- |
| [**`DescribeInstances`**](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeInstances.html)  | Returns all EC2 instances              |
| [**`RunInstances`**](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_RunInstances.html)       | Defines and starts an EC2 instance     |
| [**`StartInstances`**](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_StartInstances.html)     | Starts an already defined EC2 instance |
| [**`StopInstances`**](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_StopInstances.html)      | Stops a running EC2 instance           |
| [**`TerminateInstances`**](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_TerminateInstances.html) | Stops and undefines a EC2 instance     |

The current implementation is in preview and does not support volumes, custom AMIs, or networking features available in other [VM managers](https://docs.localstack.cloud/user-guide/aws/ec2/#vm-managers).

### New enhancements in the SES provider

The SES provider now supports the following enhancements:

-   Improved SES provider to match AWS functionality more closely.
-   `DescribeActiveReceiptRuleSet` now handles absent rule sets appropriately.
-   `DeleteReceiptRuleSet` now requires prior deactivation.
-   Enhanced error messaging and input validation for `SetActiveReceiptRuleSet`.

### Miscellaneous

- Users should be able to use LocalStack in any region, encompassing standard AWS regions as well as specialized regions such as those in China, GovCloud, and ISO-partitions. (**Enterprise**)
- Support for custom URL aliases for Lambda Function URLs is now available in LocalStack. This feature allows users to assign custom IDs to either the `$LATEST` version of a function or to an existing version alias.
- LocalStack now supports prebuilding Lambda images before execution with the `LAMBDA_PREBUILD_IMAGES` configuration variable. This approach increases the cold start time but reduces the duration until the Lambda function becomes `ACTIVE`. (**Preview**)
- LocalStack now supports idempotent [`StartExecution` operations](https://docs.aws.amazon.com/step-functions/latest/apireference/API_StartExecution.html) against already running `STANDARD` Step Functions state machines with identical input.

## Conclusion

This minor release underpins our commitment to providing a robust and feature-rich local cloud environment for developers, with a strong focus on improving parity with AWS, introduce new DevX features, and bring new local developer tools to the LocalStack ecosystem. Upgrade to LocalStack 3.7.0 today to take advantage of these new features and enhancements!

To learn more about LocalStack, check out the following:

* [LocalStack Documentation](https://docs.localstack.cloud)
* New to LocalStack? [Create an account](https://app.localstack.cloud/sign-up)!
* Join our [Slack community](https://localstack.cloud/slack) to ask questions and share feedback.
* Navigate to our [Developer Hub](https://docs.localstack.cloud/developer-hub/) and try out sample applications to get started.
* Register for our [upcoming events](https://meetup.com/localstack-community) to learn more about LocalStack.
