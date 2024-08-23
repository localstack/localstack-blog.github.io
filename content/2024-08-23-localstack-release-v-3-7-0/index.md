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

### State merging for Cloud Pods

LocalStack Cloud Pods now offer different strategies for state merging into your LocalStack container. The available strategies include:

- `overwrite`: This strategy deletes the existing state and replaces it with the new state from the Cloud Pod, effectively resetting the LocalStack state.
- `account-region-merge` (**default**): This strategy merges services by account and region pairs, combining states from both the existing and the Cloud Pod for the same account and region.
- `service-merge`: This strategy merges services at the account-region level, assuming no resource overlap exists. It gives priority to the resources from the loaded state during the merge.

To activate these strategies, use `--strategy <strategy>` when loading the Cloud Pod, where `<strategy>` is one of the strategies mentioned above. For example, to use the `service-merge` strategy, run:

```bash
localstack pod load my-test-pod --strategy service-merge
```

To learn more about how it works, check out the [Cloud Pods documentation](https://docs.localstack.cloud/user-guide/state-management/cloud-pods/#example-scenario).

### Support for DMS Serverless

LocalStack now supports AWS Database Migration Service (DMS) Serverless. DMS Serverless can be used with sources and targets supported by both LocalStack and AWS. To simulate the different states that a replication config experiences during provisioning, you can set the `DMS_SERVERLESS_STATUS_CHANGE_WAITING_TIME` environment variable. This setting delays each state change for the configured number of seconds until the replication is actively running. Refer to the official documentation for details on the different states.

Note that on AWS, replication table statistics are automatically deleted after the replication finishes and the configuration is deprovisioned. For parity, the same applies in LocalStack. To delay this deprovisioning process, you can set the `DMS_SERVERLESS_DEPROVISIONING_DELAY` environment variable, which defaults to 60 seconds.

Learn more about DMS in the [LocalStack documentation](https://docs.localstack.cloud/user-guide/aws/dms).

