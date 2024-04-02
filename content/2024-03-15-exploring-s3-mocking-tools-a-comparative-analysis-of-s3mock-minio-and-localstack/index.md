---
title: Exploring S3 Mocking Tools — A Comparative Analysis of S3Mock, MinIO, and LocalStack
description: In this blog post, we conduct a comparison of S3Mock, MinIO, and LocalStack for local development and testing. We evaluate how these tools stack up in terms of API parity with AWS S3 standards, aiming to identify the optimal choice for local S3 development & testing.
lead: In this blog post, we conduct a comparison of S3Mock, MinIO, and LocalStack for local development and testing. We evaluate how these tools stack up in terms of API parity with AWS S3 standards, aiming to identify the optimal choice for local S3 development & testing.
date: 2024-03-15T2:34:20+05:30
lastmod: 2024-03-15T2:34:20+05:30
images: ["explore-s3-mocking-tools-banner.png"]
leadimage: "explore-s3-mocking-tools-banner.png"
tags: ['showcase']
contributors: ["Cristopher Pinzon", "Benjamin Simon", "Stefanie Plieschnegger"]
---

{{< img-simple src="explore-s3-mocking-tools-banner.png" width=300 alt="Banner image for the blog: Exploring S3 Mocking Tools — A Comparative Analysis of S3Mock, MinIO, and LocalStack">}}

## Introduction

[Amazon S3 (Simple Storage Service)](https://aws.amazon.com/s3/) is a scalable and durable cloud object storage service offered by [AWS](https://aws.amazon.com). 
It plays a crucial role in cloud computing by allowing developers to store and retrieve data anywhere on the web. 
For development and testing purposes, local alternatives to S3 are convenient to avoid the time and cost of developing and debugging applications directly on the cloud. 
This blog post delves into a comprehensive comparison of popular tools that emulate S3 locally, exploring their strengths, weaknesses, and performance aspects.

For the comparison, we selected the following tools:

1. [S3Mock](https://github.com/adobe/S3Mock): It provides a server that emulates the behavior of the S3 service, allowing developers to write and test code that interacts with S3 without actually using the live AWS service. 
2. [MinIO](https://github.com/minio/minio): An open-source object storage server designed to be cloud-native and highly scalable. It is a lightweight and high-performance alternative to proprietary object storage solutions like Amazon S3.
3. [LocalStack](http://github.com/localstack/localstack): A versatile local development environment that emulates various AWS services, including S3, allowing developers to simulate an entire AWS cloud infrastructure
locally.

In addition to its other features, LocalStack presents a unique offering in the form of a [dedicated S3 container image tag](https://docs.localstack.cloud/user-guide/aws/s3/#s3-docker-image). 

This Docker image offers a simple & lightweight choice designed for emulating the S3 service. 
It makes things easier by removing extra functionality and concentrating on the key aspects of S3. 
This makes it perfect for users needing a lightweight, efficient, and successful S3 service emulation.

## How to start mocking S3

### MinIO

1. Pull the MinIO [Docker image](https://hub.docker.com/r/minio/minio): `docker pull minio/minio`
2. Start the MinIO container `docker run -p 9000:9000 -p 9001:9001 --name minio -d minio/minio server /data --console-address ":9001"`
3. Create a new access key and secret key:
    - Access the MinIO web UI: `http://localhost:9001`
    - Log in to the MinIO console using default credentials: `minioadmin` and `minioadmin`.
    - Navigate to the **Access Keys** section and create a new access key and secret key.
4. Configure your AWS client to use the MinIO server:
    - Set the endpoint URL: `http://localhost:9000`
    - Set the access key and secret key you created in step 3.

### S3Mock

1. Pull the S3Mock Docker image: `docker pull adobe/s3mock`
2. Start the S3Mock container: `docker run -p 9090:9090 --name s3mock -d adobe/s3mock`
3. Configure your AWS client to use the S3Mock server:
    - Set the endpoint URL: `http://localhost:9090`
    - 1.  -   Provide any access key and secret key. As per their example, you can use `foo` as the access key and `bar` as the secret key.

### LocalStack

1. Pull the LocalStack Docker image: `docker pull localstack/localstack:s3-latest`
2. Start the LocalStack container: `docker run -p 4566:4566 --name localstack -d localstack/localstack:s3-latest`
3. Configure your AWS client to use the LocalStack server:
    - Set the endpoint URL: `http://localhost:4566`
    - Set the access key and secret key. LocalStack uses a default access key `test` and a default secret key `test`.

You can also use `localstack/localstack:latest` if you need other services or use integrations with S3 like SNS or SQS.

## API compliance with S3

This section presents a comparison of the S3 operations supported by the three selected local S3 mocking tools: LocalStack, MinIO, and S3Mock.

The table is sorted by functionality and reveals which operations are fully supported and also compliant with AWS S3 standards.

By understanding the maturity of supported operations for each emulator, developers can choose the tool that best meets the need for local development and testing.


| Operation | LocalStack | MinIO | S3Mock |
| --- | --- | --- | --- |
| **Core operations** |  |  |  |
| [`CreateBucket`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CreateBucket.html) | ✅ | ✅ | ✅ |
| [`DeleteBucket`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_DeleteBucket.html) | ✅ | ✅ | ✅ |
| [`HeadBucket`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_HeadBucket.html) | ✅ | ✅ | ✅ |
| [`ListBuckets`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListBuckets.html) | ✅ | ✅ | ✅ |
| [`GetBucketLocation`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetBucketLocation.html) | ✅ | ✅ | ✅ |
| [`PutObject`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObject.html) | ✅ | ✅ | ✅ |
| [`GetObject`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetObject.html) | ✅ | ✅ | ✅ |
| [`HeadObject`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_HeadObject.html) | ✅ | ✅ | ✅ |
| [`CopyObject`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CopyObject.html) | ✅ | ✅ | ✅ |
| [`DeleteObject`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_DeleteObject.html) | ✅ | ✅ | ✅ |
| [`DeleteObjects`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_DeleteObjects.html) | ✅ | ❌ | ✅ |
| [`GetObjectAttributes`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetObjectAttributes.html) | ✅ | ✅ | ✅ |
| [`RestoreObject`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_RestoreObject.html) | ✅ | ❌ | ❌ |
| [`ListObjects`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListObjects.html) | ✅ | ✅ | ✅ |
| [`ListObjectsV2`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListObjectsV2.html) | ✅ | ✅ | ✅ |
| [`ListObjectVersions`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListObjectVersions.html) | ✅ | ✅ | ✅ |
| [`CreateMultipartUpload`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CreateMultipartUpload.html) | ✅ | ❌ | ✅ |
| [`AbortMultipartUpload`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_AbortMultipartUpload.html) | ✅ | ✅ | ✅ |
| [`CompleteMultipartUpload`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CompleteMultipartUpload.html) | ✅ | ✅ | ✅ |
| [`ListMultipartUploads`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListMultipartUploads.html) | ✅ | ✅ | ✅ |
| [`UploadPart`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_UploadPart.html) | ✅ | ❌ | ✅ |
| [`UploadPartCopy`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_UploadPartCopy.html) | ✅ | ❌ | ✅ |
| [`ListParts`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListParts.html) | ✅ | ❌ | ✅ |
| [`SelectObjectContent`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_SelectObjectContent.html) | ✅ | ✅ | ❌ |
| [`WriteGetObjectResponse`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_WriteGetObjectResponse.html) | ❌ | ❌ | ❌ |
| **Bucket features** |  |  |  |
| [`PutBucketAccelerateConfiguration`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketAccelerateConfiguration.html) | ✅ | ❌ | ❌ |
| [`PutBucketAcl`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketAcl.html) | ✅ | ✅ | ❌ |
| [`PutBucketAnalyticsConfiguration`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketAnalyticsConfiguration.html) | ✅ | ❌ | ❌ |
| [`PutBucketCors`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketCors.html) | ✅ | ❌ | ❌ |
| [`PutBucketEncryption`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketEncryption.html) | ✅ | ✅ | ❌ |
| [`PutBucketIntelligentTieringConfiguration`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketIntelligentTieringConfiguration.html) | ✅ | ❌ | ❌ |
| [`PutBucketInventoryConfiguration`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketInventoryConfiguration.html) | ✅ | ❌ | ❌ |
| [`PutBucketLifecycleConfiguration`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketLifecycleConfiguration.html) | ✅ | ✅ | ✅ |
| [`PutBucketLogging`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketLogging.html) | ✅ | ❌ | ❌ |
| [`PutBucketMetricsConfiguration`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketMetricsConfiguration.html) | ❌ | ❌ | ❌ |
| [`PutBucketNotificationConfiguration`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketNotificationConfiguration.html) | ✅ | ✅ | ❌ |
| [`PutBucketOwnershipControls`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketOwnershipControls.html) | ✅ | ❌ | ❌ |
| [`PutBucketPolicy`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketPolicy.html) | ✅ | ❌ | ❌ |
| [`PutBucketReplication`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketReplication.html) | ✅ | ✅ | ❌ |
| [`PutBucketRequestPayment`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketRequestPayment.html) | ✅ | ❌ | ❌ |
| [`PutBucketTagging`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketTagging.html) | ✅ | ✅ | ❌ |
| [`PutBucketVersioning`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketVersioning.html) | ✅ | ✅ | ❌ |
| [`PutBucketWebsite`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketWebsite.html) | ✅ | ❌ | ❌ |
| [`PutPublicAccessBlock`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutPublicAccessBlock.html) | ✅ | ❌ | ❌ |
| [`PutObjectLockConfiguration`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObjectLockConfiguration.html) | ✅ | ✅ | ✅ |
| **Object features** |  |  |  |
| [`PutObjectAcl`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObjectAcl.html) | ✅ | ✅ | ✅ |
| [`PutObjectLegalHold`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObjectLegalHold.html) | ✅ | ✅ | ✅ |
| [`PutObjectRetention`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObjectRetention.html) | ✅ | ✅ | ✅ |
| [`PutObjectTagging`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObjectTagging.html) | ✅ | ✅ | ✅ |
| [`GetObjectTorrent`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetObjectTorrent.html) | ❌ | ❌ | ❌ |

## Test results

For comparable tests we went for the [S3 integration test suite from the LocalStack repository](https://github.com/localstack/localstack/tree/s3-benchmarking/tests/aws/services/s3). Those tests are verified and validated against AWS S3, allowing for a good set of representative use cases.

The tests first targeted MinIO and then S3Mock to evaluate how well these S3-compatible tools could handle the tests designed for the S3 service.

It’s important to note that we purposely excluded any tests that validate the integration with other services (like Lambda or SQS). We can assume that those integrations are not (fully) working with the other tools.

In this testing phase, we encountered a significant number of tests that had failed. Consequently, we took it upon ourselves to thoroughly analyze the results of these tests.

We established a system to organize and categorize the results into four distinct classes for a more streamlined analysis. These classes included:

-   **Passed**: The test was successful.
-   **Missing Exception**: The test failed because an expected exception was not encountered.
-   **Imparity**: The test failed because the response did not contain specific attributes.
-   **Not Supported**: The test could not be performed because required support or resources were not available.

It’s important to note that for MinIO, we had to make several modifications to the tests. These included forcing the signature version of the AWS SDK Python client and setting up an access key and secret key previously created in the tool console.

{{< img-simple src="localstack-tests-s3mock-MinIO.png" alt="LocalStack S3 Tests executed against S3Mock and MinIO" width="800">}}

### LocalStack and AWS parity

LocalStack strives for continuous improvement to align its services closely with AWS. 
It maintains an extensive set of integration tests to thoroughly assess the features and capabilities of its simulated services, ensuring they mirror their AWS counterparts as accurately as possible. 
Given this dedication, it's understandable that other tools may not match LocalStack's parity with AWS services.

Here are some illustrative examples where LocalStack outperforms other comparable tools.
The goal of these examples is to demonstrate the relative superiority of LocalStack in terms of the breadth and depth of AWS services it emulates, compared to the other tools:

- `test_s3_list_objects_timestamp_precision`: The reason for this failure is the lack of parity in the timestamp.
The timestamp is a crucial aspect of data handling and, in this case, it's not returned in the correct format, which should be in ISO 8061.

- `test_put_get_object_special_character`: This parametrized test explores the potential for uploading objects that contain special characters within their keys. 
It is a critical feature to test as it ensures the systems' ability to handle a variety of object key inputs. Some examples of the keys rejected by MinIO and S3Mock:
    - S3mock:
        - `file%2Fname`
        - `a/%F0%9F%98%80/`
        - `test+key`
        - `test key//`
    - MinIO:
        - `test key//`

- `test_multipart_and_list_parts`: This is a unique circumstance where other tools give the impression of supporting the feature. 
However, when the integrity of the object and its parts is verified, these tools fail.

## Warp Benchmarking

In addition to our usual set of tools, we used benchmarking open-source software developed by the MinIO team called [Warp](https://github.com/minio/warp). 
Warp facilitated a comparative analysis of the performance metrics associated with each tool and operation we utilized.

Effectively, this allowed us to measure and understand the impact of each tool and operation on our overall workflow. Following this analysis, we have compiled the results to provide a concise summary.

{{< img-simple src="performance-comparison.png" alt="" width="800">}}

> The benchmarks were executed on a MacBook Pro M3 Max.

For the comparison, we divided the tests into two categories:

-   Small objects (e.g., 1000 bytes): Here, we presume there might be optimizations in place since smaller objects can be stored in memory.
-   Medium-sized objects (e.g., 1 MiB).

We compared the [PutObject](https://github.com/minio/warp?tab=readme-ov-file#put) and [GetObject](https://github.com/minio/warp?tab=readme-ov-file#get) requests to get a bigger picture.

For small objects, we can see a similar throughput for PUT and GET for the LocalStack-S3 image. It’s also the winner for PUT requests, while GETs are faster on MinIO.

For the medium-sized benchmark test, we can observe a significant difference between the PUT and GET operations in general. While LocalStack-S3 outperforms for PUT operations, MinIO and S3Mock are slightly faster for GETs.

During our benchmarking, we ran into unknown issues with S3Mock, where we had to restart the server between test runs.

## Conclusion

In conclusion, while S3Mock is a robust and performant option suitable for testing environments, MinIO is a direct competitor to AWS S3, offering a production-grade object storage solution.

LocalStack is the optimal choice for testing applications before deploying to AWS. Its dedication to closely mirroring AWS services, validated with an extensive integration testing suite, enables developers to accurately simulate the AWS environment locally.

If your goal is to validate your application’s behavior and compatibility with AWS before deployment, LocalStack is an ideal testing companion due to its dedicated emulation of AWS services. Utilizing LocalStack during development and testing allows you to confidently identify and resolve potential issues, ensuring a smooth transition to the AWS production environment.
