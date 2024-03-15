---
title: Exploring S3 Mocking Tools — A Comparative Analysis of S3Mock, MinIO, and LocalStack
description: Exploring S3 Mocking Tools — A Comparative Analysis of S3Mock, MinIO, and LocalStack
lead: Exploring S3 Mocking Tools — A Comparative Analysis of S3Mock, MinIO, and LocalStack
date: 2024-03-15T2:34:20+05:30
lastmod: 2024-03-15T2:34:20+05:30
images: []
contributors: []
tags: ['showcase']
contributors: ["Cristopher Pinzon", "Benjamin Simon"]
---

## Introduction

Amazon S3 (Simple Storage Service) is a scalable and durable cloud object storage service offered by AWS. It plays a crucial role in cloud computing by allowing developers to store and retrieve data from anywhere on the web. For testing and development purposes, local alternatives to S3 are often needed to avoid the time and cost of developing and debugging applications directly in the cloud.This blog post delves into a comprehensive comparison of these tools, exploring their strengths, weaknesses, and performance aspects.

Three popular tools for emulating S3 locally are:

1. S3Mock: A lightweight, open-source tool for quickly emulating S3 locally. 
2. MinIO: A scalable, standalone object storage server that is API-compatible with S3, suitable for local development and production deployments.
3. LocalStack: A versatile local development environment that emulates various AWS services, including S3, allowing developers to simulate an entire AWS cloud infrastructure
locally.

Setting up each of the three tools - S3Mock, MinIO, and LocalStack - for local S3 emulation is a straightforward process. It simply involves pulling the respective image for the tool and starting the container. This ease of setup allows developers to quickly establish a local development environment for testing and development purposes without the need for direct interaction with AWS cloud infrastructures. 

In addition to its other features, LocalStack presents a unique offering in the form of a dedicated S3 container image tag. This specialized feature provides a more streamlined, lightweight option that is exclusively focused on emulating the services of S3. It simplifies the process by eliminating unnecessary components and honing in on the essential elements of S3. This makes it an ideal solution for users who require a lightweight, efficient, and effective S3 service emulation.

## API compliance with S3

This document presents a comprehensive comparison of the S3 operations supported by three popular local S3 emulation tools: LocalStack, MinIO, and S3Mock. By understanding the specific S3 operations that each tool supports, developers can choose the tool that best meets their needs for local development and testing.

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

## Tests results

From the LocalStack repository, we ran the S3 integration tests, first targeting MinIO and then S3Mock, to evaluate how well these S3-compatible tools could handle the tests designed for the LocalStack S3 service. It's important to note that we purposely excluded any tests that were designed to validate the integration with other services like Lambda or SQS.

In this testing phase, we encountered a significant number of tests that had failed. Consequently, we took it upon ourselves to thoroughly analyze the results of these tests. We established a system to organize and categorize the results into four distinct classes for a more streamlined analysis. These classes included 'Passed', where the test completed successfully; 'Missing Exception', where the test failed due to an unexpected lack of exception; 'Imparity', where the test failed due to a response lacking specific attributes; and finally, 'Not Supported', where the test could not be carried out due to the lack of necessary support or resources.

It's important to note that for MinIO, we had to make several modifications to the tests. These included forcing the signature_version of the boto client and setting up an access key and secret key that were previously created in the tool console.

{{< img-simple src="localstack-tests-s3mock-MinIO.png" alt="LocalStack S3 Tests executed against S3Mock and MinIO" width="800">}}

LocalStack strives for continuous improvement to align its services closely with AWS. It maintains an extensive set of integration tests to thoroughly assess the features and capabilities of its simulated services, ensuring they mirror their AWS counterparts as accurately as possible. Given this dedication, it's understandable that other tools may not match LocalStack's parity with AWS services.

Here are some illustrative examples where LocalStack, in its ongoing effort to achieve parity, manages to outperform other comparable tools in the market. The goal of these examples is to demonstrate the relative superiority of LocalStack in terms of the breadth and depth of AWS services it emulates, compared to the other tools:

- test_s3_list_objects_timestamp_precision: The reason for this failure is the lack of parity in the timestamp. The timestamp is a crucial aspect of data handling and, in this case, it's not returned in the correct format, which should be in ISO 8061.
- test_put_get_object_special_character: This parametrized test is designed to explore the potential for uploading objects that contain special characters within their keys. This is a critical feature to test as it ensures the systems' ability to handle a variety of object key inputs. Some examples of the keys rejected by MinIO and S3Mock:
    - "file%2Fname"
    - "test key//"
    - "a/%F0%9F%98%80/"
    - "test+key"
- test_multipart_and_list_parts: This is a unique circumstance where other tools give the impression of supporting the feature. However, when the integrity of the object and its parts is verified, these tools fail.

## Warp Benchmarking

In addition to our usual set of tools, we also employed the use of a benchmarking open-source software developed by the MinIO team called Warp. The primary reason for this was to facilitate a comparative analysis of the performance metrics associated with each tool and operation that we utilized in our process. Effectively, this allowed us to measure and understand the impact of each tool and operation on our overall workflow. Following this analysis, we have compiled the results to provide a concise summary.

{{< img-simple src="performance-comparison.png" alt="" width="800">}}

In this comparison, MinIO and S3 mock consistently outperform LocalStack in terms of speed. This superior performance is evident across various tasks and workloads, demonstrating the efficiency of these tools. The improved performance could be due to their programming language, making them a more suitable choice for those seeking faster data processing and lower latency.

## LocalStack's Performance Enhancement

LocalStack has recently merged a pull request that replaces the previous Hypercorn web server with Twisted, a framework supporting HTTP/2 and WebSocket without relying on asyncio. This change addresses performance bottlenecks, keep-alive issues slowing down clients, and potential blocking behavior experienced with the prior implementation. 

By adopting Twisted, LocalStack eliminates the overhead from the async/sync bridging code, enabling more efficient request handling. Early benchmarks show an average 2x throughput improvement, sometimes even higher. With this enhancement, LocalStack reinforces its commitment to delivering a high-performance local testing environment that closely mirrors AWS.

To begin utilizing this powerful feature, all you need to do is take the simple step of enabling it. You can do this by setting the GATEWAY_SERVER variable to `twisted`.

{{< img-simple src="performance-increase-twisted.png" alt="" width="800">}}

## Conclusion

In conclusion, while S3Mock emerges as a robust and performant option suitable for testing environments, MinIO positions itself as a direct competitor to AWS S3, offering a production-grade object storage solution. LocalStack stands out as the optimal choice for testing applications before deploying to AWS. Its dedication to closely mirroring AWS services, coupled with an extensive integration testing suite, enables developers to accurately simulate the AWS environment locally.

If your goal is to validate your application's behavior and compatibility with AWS prior to deployment, LocalStack is an ideal testing companion due to its faithful emulation of AWS services. Utilizing LocalStack during development and testing allows you to confidently identify and resolve potential issues, ensuring a smooth transition to the AWS production environment.
