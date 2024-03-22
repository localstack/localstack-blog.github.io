---
title: Exploring S3 Mocking Tools — A Comparative Analysis of S3Mock, MinIO, and LocalStack
description: Exploring S3 Mocking Tools — A Comparative Analysis of S3Mock, MinIO, and LocalStack
lead: Exploring S3 Mocking Tools — A Comparative Analysis of S3Mock, MinIO, and LocalStack
date: 2024-03-15T2:34:20+05:30
lastmod: 2024-03-15T2:34:20+05:30
images: []
contributors: []
tags: ['showcase']
contributors: ["Cristopher Pinzon", "Benjamin Simon", "Stefanie Plieschnegger"]
---

## Introduction

Amazon S3 (Simple Storage Service) is a scalable and durable cloud object storage service offered by AWS. 
It plays a crucial role in cloud computing by allowing developers to store and retrieve data from anywhere on the web. 
For testing and development purposes, local alternatives to S3 are convenient to avoid the time and cost of developing and debugging applications directly in the cloud. 
This blog post delves into a comprehensive comparison of some popular tools that emulate S3 locally, exploring their strengths, weaknesses, and performance aspects.

For the comparison we selected the following tools:

1. S3Mock: It provides a server that emulates the behavior of the real Amazon S3 service, allowing developers to write and test code that interacts with S3 without actually using the live AWS service. 
2. MinIO: An open-source object storage server designed to be cloud-native and highly scalable. It is a lightweight and high-performance alternative to proprietary object storage solutions like Amazon S3.
3. LocalStack: A versatile local development environment that emulates various AWS services, including S3, allowing developers to simulate an entire AWS cloud infrastructure
locally.

n addition to its other features, LocalStack presents a unique offering in the form of a [dedicated S3 container image tag](https://hub.docker.com/r/localstack/localstack/tags?page=1&name=s3-latest). 

This specialized feature provides a more streamlined, lightweight option that is exclusively focused on emulating the services of S3. 
It simplifies the process by eliminating unnecessary components and honing in on the essential elements of S3. 
This makes it an ideal solution for users who require a lightweight, efficient, and effective S3 service emulation.

## How to start mocking S3

### [MinIO](https://github.com/minio/minio)

1. Pull the MinIO [Docker image](https://hub.docker.com/r/minio/minio): `docker pull minio/minio`
2. Start the MinIO container `docker run -p 9000:9000 -p 9001:9001 --name minio -d minio/minio server /data --console-address ":9001"`
3. Create a new access key and secret key:
    - Open the MinIO web UI: `http://localhost:9001`
    - Connect to the MinIO console with the following default credentials: `minioadmin` and `minioadmin`.
    - Go to the "Access Keys" section and create a new access key and secret key.
4. Configure your AWS client to use the MinIO server:
    - Set the endpoint URL: `http://localhost:9000`
    - Set the access key and secret key you created in step 3.

### [S3Mock](https://github.com/adobe/S3Mock)

1. Pull the S3Mock Docker image: `docker pull adobe/s3mock`
2. Start the S3Mock container: `docker run -p 9090:9090 --name s3mock -d adobe/s3mock`
3. Configure your AWS client to use the S3Mock server:
    - Set the endpoint URL: `http://localhost:9090`
    - Set the access key and secret key: S3Mock accepts any access key id and secret key. Their example uses `foo` as the access key and `bar` as secret

### [LocalStack](https://github.com/localstack/localstack)

1. Pull the LocalStack Docker image: `docker pull localstack/localstack:s3-latest`
2. Start the LocalStack container: `docker run -p 4566:4566 --name localstack -d localstack/localstack:s3-latest`
3. Configure your AWS client to use the LocalStack server:
    - Set the endpoint URL: `http://localhost:4566`
    - Set the access key and secret key (LocalStack uses a default access key of `test` and a default secret key of `test`

You can also use `localstack/localstack:latest` in case you need other services or use integrations with S3 like SNS, or SQS.

## API compliance with S3

This section presents a comprehensive comparison of the S3 operations supported by the three selected local S3 emulation tools: LocalStack, MinIO, and S3Mock. 

The table is sorted by functionality and reveals which operation is fully supported and also in compliance with AWS S3. 
By understanding the maturity of supported operations for each emulator, developers can choose the tool that best meets their needs for local development and testing.

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

In order to have a comparable set of tests, we went for the S3 integration tests from the LocalStack repository. 
Those tests are verified and validated against AWS S3 and therefore form a good set of representative use cases.

The tests were first targeting MinIO and then S3Mock, to evaluate how well these S3-compatible tools could handle the tests designed for the S3 service. 
It's important to note that we purposely excluded any tests that were designed to validate the integration with other services like Lambda or SQS as we can assume that those integrations are not (fully) working.

In this testing phase, we encountered a significant number of tests that had failed. 
Consequently, we took it upon ourselves to thoroughly analyze the results of these tests. 

We established a system to organize and categorize the results into four distinct classes for a more streamlined analysis. 
These classes included 

- **Passed**: the test completed successfully
- **Missing Exception**: the test failed due to an unexpected lack of exception
- **Imparity**: the test failed due to a response lacking specific attributes
- **Not Supported**: the test could not be carried out due to the lack of necessary support or resources

It's important to note that for MinIO, we had to make several modifications to the tests. 
These included forcing the signature version of the AWS SDK Python client and setting up an access key and secret key that were previously created in the tool console.

{{< img-simple src="localstack-tests-s3mock-MinIO.png" alt="LocalStack S3 Tests executed against S3Mock and MinIO" width="800">}}

### LocalStack and AWS parity

LocalStack strives for continuous improvement to align its services closely with AWS. 
It maintains an extensive set of integration tests to thoroughly assess the features and capabilities of its simulated services, ensuring they mirror their AWS counterparts as accurately as possible. 
Given this dedication, it's understandable that other tools may not match LocalStack's parity with AWS services.

Here are some illustrative examples where LocalStack, in its ongoing effort to achieve parity, manages to outperform other comparable tools in the market. 
The goal of these examples is to demonstrate the relative superiority of LocalStack in terms of the breadth and depth of AWS services it emulates, compared to the other tools:

- `test_s3_list_objects_timestamp_precision`: The reason for this failure is the lack of parity in the timestamp. 
The timestamp is a crucial aspect of data handling and, in this case, it's not returned in the correct format, which should be in ISO 8061.

- `test_put_get_object_special_character`: This parametrized test is designed to explore the potential for uploading objects that contain special characters within their keys. 
This is a critical feature to test as it ensures the systems' ability to handle a variety of object key inputs. Some examples of the keys rejected by MinIO and S3Mock:
    - "file%2Fname"
    - "test key//"
    - "a/%F0%9F%98%80/"
    - "test+key"

- `test_multipart_and_list_parts`: This is a unique circumstance where other tools give the impression of supporting the feature. 
However, when the integrity of the object and its parts is verified, these tools fail.

## Warp Benchmarking

In addition to our usual set of tools, we also employed the use of a benchmarking open-source software developed by the MinIO team called [Warp](https://github.com/minio/warp). 
The primary reason for this was to facilitate a comparative analysis of the performance metrics associated with each tool and operation that we utilized in our process. 

Effectively, this allowed us to measure and understand the impact of each tool and operation on our overall workflow. Following this analysis, we have compiled the results to provide a concise summary.

{{< img-simple src="performance-comparison.png" alt="" width="800">}}

For the comparison we split the tests into two categories: 

- small objects (e.g. 1000 bytes): where we assume that there may be some optimization in place as smaller objects can be kept in-memory
- and medium-sized objects (e.g. 1 MiB).

We compared the [PutObject](https://github.com/minio/warp?tab=readme-ov-file#put) and [GetObject](https://github.com/minio/warp?tab=readme-ov-file#get) requests to get a bigger picture.

The benchmarks were executed on a MacBook Pro M3 Max. 

For small objects, we can see a similar throughput for PUT and GET for the LocalStack-S3 image. It’s also the winner for PUT requests, while GETs are faster on MinIO. 

For the medium-sized benchmark test we can observe a larger difference between the PUT and GET operations in general. 
While LocalStack-S3 outperforms for PUT operations, MinIO and S3Mock are slightly faster for GETs.

During our benchmarking we also ran into some unknown issues with S3Mock, where we had to restart the server in between test runs.

## Conclusion

In conclusion, while S3Mock emerges as a robust and performant option suitable for testing environments, MinIO positions itself as a direct competitor to AWS S3, offering a production-grade object storage solution. 

LocalStack stands out as the optimal choice for testing applications before deploying to AWS. 
Its dedication to closely mirroring AWS services, coupled with an extensive integration testing suite, enables developers to accurately simulate the AWS environment locally.

If your goal is to validate your application's behavior and compatibility with AWS prior to deployment, LocalStack is an ideal testing companion due to its dedicated emulation of AWS services. 
Utilizing LocalStack during development and testing allows you to confidently identify and resolve potential issues, ensuring a smooth transition to the AWS production environment.