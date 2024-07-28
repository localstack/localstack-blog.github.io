---
title: Simulating outages for local cloud apps with LocalStack Chaos API
description:  
lead:  
date: 2024-07-25T9:17:34+05:30
lastmod: 2024-07-25T9:17:34+05:30
images: []
contributors: ["Harsh Mishra", "Anca Ghenade", "Viren Nadkarni"]
tags: ['tutorial']
---

## Introduction

LocalStack's core cloud emulator provides the capability to emulate various AWS services, including Lambda, DynamoDB, ECS, and more, directly on your local machine. 
One notable feature of LocalStack is its support for advanced disaster recovery testing, including:

* Region-wide outages
* DNS failovers
* Service failures
* Network faults

All these testing scenarios can be efficiently executed within LocalStack, providing thorough coverage for critical situations in a matter of minutes rather than hours or days. 
To simulate outages in LocalStack, you can use the [LocalStack Chaos API](https://docs.localstack.cloud/user-guide/chaos-engineering/chaos-api/) that enables you to run various chaos experiments on your local cloud application to monitor the system's response in situations where the infrastructure is compromised.

This allows you to quickly experiment with different failure scenarios, allowing you to perform chaos testing at an early stage by introducing errors at the infrastructure level. 
This is valuable as it enables you to replicate conditions that might not be feasible to mimic unless deployed to a production environment.

This blog will walk you through the process of setting up a cloud application on your local machine and leveraging the Chaos API to perform service failures in a local environment while using robust error handling to address and mitigate such issues. 
Furthermore, we will explore how to shift-left your chaos testing by integrating automated testing directly into your continuous integration workflow.

## Prerequisites

* [LocalStack Docker image](https://docs.localstack.cloud/references/docker-images/#localstack-pro-image) & [`LOCALSTACK_AUTH_TOKEN`](https://docs.localstack.cloud/getting-started/auth-token/)
* [Docker Compose](https://docs.docker.com/compose/install/)
* [AWS CLI](https://docs.aws.amazon.com/cli/v1/userguide/cli-chap-install.html) & [`awslocal`  wrapper](https://docs.localstack.cloud/user-guide/integrations/aws-cli/#localstack-aws-cli-awslocal)
* [Maven 3.8.5](https://maven.apache.org/install.html) & [Java 17](https://www.java.com/en/download/help/download_options.html)
* [Python](https://www.python.org/downloads/) & [`pytest`  framework](https://docs.pytest.org/en/8.0.x/)
* [`cURL`](https://curl.se/docs/install.html)

## Product Management System with Lambda, API Gateway, and DynamoDB

This demo sets up an HTTP CRUD API functioning as a Product Management System. The components deployed include:

* A DynamoDB table named  `Products`.
* Three Lambda functions:
  * `add-product`  for product addition.
  * `get-product`  for retrieving a product.
  * `process-product-events`  for event processing and DynamoDB writes.
* A locally hosted REST API named  `quote-api-gateway`.
* SNS topic named  `ProductEventsTopic`  and SQS queue named  `ProductEventsQueue`.  
* API Gateway resource named  `productApi`  with additional  `GET`  and  `POST`  methods.

Additionally, the applications set up a subscription between the SQS queue and the SNS topic, along with an event source mapping between the SQS queue and the  `process-product-events`  Lambda function.

All resources can be deployed using a [LocalStack Init Hook](https://docs.localstack.cloud/references/init-hooks/) via the [`init-resources.sh`](https://github.com/localstack-samples/sample-chaos-api-serverless/blob/main/init-resources.sh)  script in the repository. 
To begin, clone the repository on your local machine:

```bash
git clone https://github.com/localstack-samples/sample-chaos-api-serverless.git
cd sample-chaos-api-serverless
```

Let's create a Docker Compose configuration for simulating a local outage in the running Product Management System.

### Set Up the Docker Compose

To start LocalStack and use the Chaos API, create a new Docker Compose configuration. 
You can find the official Docker Compose file for starting the LocalStack container in [our documentation](https://docs.localstack.cloud/getting-started/installation/#docker-compose).

For an extended setup, include the following in your Docker Compose file:

* Include the  `LOCALSTACK_HOST=localstack`  environment variable to ensure LocalStack services are accessible from other containers.
* Create the  `ls_network`  network to use LocalStack as its DNS server and enable the resolution of the domain name to the LocalStack container (also specify it via  `LAMBDA_DOCKER_NETWORK`  environment variable).
* Add a new volume attached to the LocalStack container. This volume holds the  `init-resources.sh`  file, which is copied to the LocalStack container and executed when the container is ready.
* Add another volume to copy the built Lambda functions specified as ZIP files during Lambda function creation.
* Optionally, add the  `LAMBDA_RUNTIME_ENVIRONMENT_TIMEOUT`  to wait for the runtime environment to start up, which may vary in speed based on your local machine.

The final Docker Compose configuration is as follows (also [provided in the repository](https://github.com/localstack-samples/sample-chaos-api-serverless/blob/main/docker-compose.yml)):

```yaml
version: "3.9"

services:
  localstack:
    networks:
      - ls_network
    container_name: localstack
    image: localstack/localstack-pro:latest
    ports:
      - "127.0.0.1:4566:4566"            
      - "127.0.0.1:4510-4559:4510-4559"  
      - "127.0.0.1:443:443"
    environment:
      - DOCKER_HOST=unix:///var/run/docker.sock
      - LOCALSTACK_HOST=localstack
      - LAMBDA_DOCKER_NETWORK=ls_network
      - LOCALSTACK_AUTH_TOKEN=${LOCALSTACK_AUTH_TOKEN:?}
      - LAMBDA_RUNTIME_ENVIRONMENT_TIMEOUT=600
    volumes:
      - "./volume:/var/lib/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"
      - "./lambda-functions/target/product-lambda.jar:/etc/localstack/init/ready.d/target/product-lambda.jar"
      - "./init-resources.sh:/etc/localstack/init/ready.d/init-resources.sh"

networks:
  ls_network:
    name: ls_network
```

### Deploy the local AWS infrastructure

Before deploying the demo application locally, build the Lambda functions to ensure they can be copied over during Docker Compose startup. Execute the following command:

```bash
cd lambda-functions && mvn clean package shade:shade
```

The built Lambda function is now available at  `lambda-functions/target/product-lambda.jar`. 
Start the Docker Compose configuration, which automatically creates the local deployment using AWS CLI and the  `awslocal`  script inside the LocalStack container:

```bash
export LOCALSTACK_AUTH_TOKEN=<your-auth-token>
docker-compose up
```

Check the Docker Compose logs to verify that LocalStack is running and your local AWS infrastructure is set up correctly. 
You should see the following output:

```bash
localstack  | 
localstack  | LocalStack version: 3.6.1.dev20240725091954
localstack  | LocalStack build date: 2024-07-26
localstack  | LocalStack build git hash: d536652
localstack  | 
localstack  | 2024-07-26T08:55:12.280  INFO --- [  MainThread] l.p.c.b.licensingv2        : Successfully activated cached license a61ce6c3-2f7f-4a18-bc50-b69184a29a4a:enterprise from /var/lib/localstack/cache/license.json 🔑✅
localstack  | 2024-07-26T08:55:13.680  INFO --- [  MainThread] l.p.c.extensions.platform  : loaded 0 extensions
...
localstack  | 2024-07-26T08:55:21.526  INFO --- [et.reactor-2] localstack.request.aws     : AWS sns.CreateTopic => 200
localstack  | {
localstack  |     "TopicArn": "arn:aws:sns:us-east-1:000000000000:ProductEventsTopic"
localstack  | }
localstack  | 2024-07-26T08:55:21.827  INFO --- [et.reactor-3] localstack.request.aws     : AWS sqs.CreateQueue => 200
localstack  | {
localstack  |     "QueueUrl": "http://sqs.us-east-1.localstack:4566/000000000000/ProductEventsQueue"
localstack  | }
```

After deployment, use  `cURL`  to create a product entity. 
Execute the following command:

```bash
curl --location 'http://12345.execute-api.localhost.localstack.cloud:4566/dev/productApi' \
--header 'Content-Type: application/json' \
--data '{
  "id": "prod-2004",
  "name": "Ultimate Gadget",
  "price": "49.99",
  "description": "The Ultimate Gadget is the perfect tool for tech enthusiasts looking for the next level in gadgetry. Compact, powerful, and loaded with features."
}'
```

The output should be:

```bash
Product added/updated successfully.
```

You can verify the successful addition by scanning the DynamoDB table:

```bash
awslocal dynamodb scan \
    --table-name Products
```

The output should be:

```json
{
    "Items": [
        {
            "name": {
                "S": "Ultimate Gadget"
            },
            "description": {
                "S": "The Ultimate Gadget is the perfect tool for tech enthusiasts looking for the next level in gadgetry. Compact, powerful, and loaded with features."
            },
            "id": {
                "S": "prod-2004"
            },
            "price": {
                "N": "49.99"
            }
        }
    ],
    "Count": 1,
    "ScannedCount": 1,
    "ConsumedCapacity": null
}
```

### Injecting Chaos in the local infrastructure

You can now use the Chaos API for chaos testing of your locally deployed infrastructure. You can access the Chaos API through the REST API at [`http://localhost.localstack.cloud:4566/_localstack/chaos/faults`](http://localhost.localstack.cloud:4566/_localstack/chaos/faults), accepting standard HTTP requests.

To create an outage, taking down the DynamoDB table in the `us-east-1` region, execute the following command:

```bash
curl --location --request POST 'http://localhost.localstack.cloud:4566/_localstack/chaos/faults' \
  --header 'Content-Type: application/json' \
  --data '
  [
    {
      "service": "dynamodb",
      "region": "us-east-1"
    }
  ]'
```

The output should be:

```json
[{"service": "dynamodb", "region": "us-east-1"}]
```

This command creates an outage in the locally mocked  `us-east-1`  DynamoDB tables. Verify by scanning the  `Products`  table:

```bash
awslocal dynamodb scan \
    --table-name Products
```

The output should be:

```bash
An error occurred (ServiceUnavailable) when calling the Scan operation (reached max retries: 2): Operation failed due to a simulated fault
```

You can verify it in the LocalStack logs:

```bash
localstack  | 2024-07-26T09:14:31.250  INFO --- [et.reactor-2] localstack.request.aws     : AWS dynamodb.Scan => 503 (ServiceUnavailable)
localstack  | 2024-07-26T09:14:32.255  INFO --- [et.reactor-0] localstack.request.aws     : AWS dynamodb.Scan => 503 (ServiceUnavailable)
localstack  | 2024-07-26T09:14:34.225  INFO --- [et.reactor-2] localstack.request.aws     : AWS dynamodb.Scan => 503 (ServiceUnavailable)
```

You can retrieve the current outage configuration using the following  `GET`  request:

```bash
curl --location --request GET 'http://localhost.localstack.cloud:4566/_localstack/chaos/faults'
```

The output should be:

```bash
[{"service": "dynamodb", "region": "us-east-1"}]
```

### Error handling for the local outage

Now that the experiment is started, the DynamoDB table is inaccessible, resulting in the user being unable to retrieve or create products. 
The API Gateway will return an `Internal Server Error`. To prevent this, include proper error handling and a mechanism to prevent data loss during a database outage.

The solution includes an SNS topic, an SQS queue, and a Lambda function that picks up queued elements and retries the `PutItem` operation on the DynamoDB table. 
If DynamoDB is still unavailable, the item will be re-queued.

Test this by executing the following command:

```bash
curl --location 'http://12345.execute-api.localhost.localstack.cloud:4566/dev/productApi' \
     --header 'Content-Type: application/json' \
     --data '{
       "id": "prod-1003",
       "name": "Super Widget",
       "price": "29.99",
       "description": "A versatile widget that can be used for a variety of purposes. Durable, reliable, and affordable."
     }'
```

The output should be:

```bash
A DynamoDB error occurred. Message sent to queue.
```

To stop the outage, send a  `POST`  request by using an empty list in the configuration. 
The following request will clear the current configuration:

```bash
curl --location --request POST 'http://localhost.localstack.cloud:4566/_localstack/chaos/faults' \
--header 'Content-Type: application/json' \
--data '[]'
```

Now, scan the DynamoDB table and verify that the `Super Widget` item has been inserted:

```bash
awslocal dynamodb scan \
    --table-name Products
```

The output should be:

```bash
awslocal dynamodb scan --table-name Products
{
    "Items": [
        {
            "name": {
                "S": "Super Widget"
            },
            ...
            "price": {
                "N": "29.99"
            }
        },
        {
            "name": {
                "S": "Ultimate Gadget"
            },
            ...
            "price": {
                "N": "49.99"
            }
        }
    ],
    "Count": 2,
    "ScannedCount": 2,
    "ConsumedCapacity": null
}
```

### Automating Chaos Experiments using Pytest

You can now implement a straightforward chaos test using `pytest` to start an outage. 
The test will:

* Validate the availability of Lambda functions and the DynamoDB table.
* Start a local outage and verify if DynamoDB API calls throw an error.
* Validate the ongoing outage and its appropriate cessation.
* Query the DynamoDB table for new items and assert their presence.

For integration testing, you can use the [boto3](https://aws.amazon.com/sdk-for-python/) and the [pytest](https://pytest.org/) framework. 
In a new directory named `tests`, create a file named `test_chaos.py`. 
Add the necessary imports and `pytest` fixtures:

```python
import pytest
import time
import boto3
import requests

# Replace with your LocalStack endpoint
LOCALSTACK_ENDPOINT = "http://localhost.localstack.cloud:4566"
CHAOS_ENDPOINT = f"{LOCALSTACK_ENDPOINT}/_localstack/chaos/faults"

# Replace with your LocalStack DynamoDB table name
DYNAMODB_TABLE_NAME = "Products"

# Replace with your Lambda function names
LAMBDA_FUNCTIONS = ["add-product", "get-product", "process-product-events"]

@pytest.fixture(scope="module")
def dynamodb_resource():
    return boto3.resource("dynamodb", endpoint_url=LOCALSTACK_ENDPOINT)

@pytest.fixture(scope="module")
def lambda_client():
    return boto3.client("lambda", endpoint_url=LOCALSTACK_ENDPOINT)
```

Add the following code to perform a simple smoke test ensuring the availability of Lambda functions and the DynamoDB table:

```python
def test_dynamodb_table_exists(dynamodb_resource):
    tables = dynamodb_resource.tables.all()
    table_names = [table.name for table in tables]
    assert DYNAMODB_TABLE_NAME in table_names

def test_lambda_functions_exist(lambda_client):
    functions = lambda_client.list_functions()["Functions"]
    function_names = [func["FunctionName"] for func in functions]
    assert all(func_name in function_names for func_name in LAMBDA_FUNCTIONS)
```

Next, add the following helper functions to start, check, and stop the DynamoDB outage:

```python
def initiate_dynamodb_outage():
    outage_payload = [{"service": "dynamodb", "region": "us-east-1"}]
    requests.post(CHAOS_ENDPOINT, json=outage_payload)
    return outage_payload

def check_outage_status(expected_status):
    outage_status = requests.get(CHAOS_ENDPOINT).json()
    assert outage_status == expected_status

def stop_dynamodb_outage():
    requests.post(CHAOS_ENDPOINT, json=[])
    check_outage_status([])
```

Now, add the following code to chaos test the locally deployed DynamoDB table:

```python
def test_dynamodb_outage(dynamodb_resource):
    # Initiate DynamoDB outage
    outage_payload = initiate_dynamodb_outage()

    # Make a request to DynamoDB and assert an error
    url = "http://12345.execute-api.localhost.localstack.cloud:4566/dev/productApi"
    headers = {"Content-Type": "application/json"}
    data = {
        "id": "prod-1002",
        "name": "Super Widget",
        "price": "29.99",
        "description": "A versatile widget that can be used for a variety of purposes. Durable, reliable, and affordable.",
    }

    response = requests.post(url, headers=headers, json=data)
    assert "error" in response.text

    # Check if outage is running
    check_outage_status(outage_payload)

    # Stop the outage
    stop_dynamodb_outage()

    # Wait for a few seconds
    time.sleep(60)

    # Query if there are items in DynamoDB table
    table = dynamodb_resource.Table(DYNAMODB_TABLE_NAME)
    response = table.scan()
    items = response["Items"]
    print(items)
    assert any(item["name"] == "Super Widget" for item in items)
```

Run the test locally using the following command:

```bash
pytest -v
```

The output should be:

```bash
collected 3 items                                                                               

tests/test_outage.py::test_dynamodb_table_exists PASSED                                   [ 33%]
tests/test_outage.py::test_lambda_functions_exist PASSED                                  [ 66%]
tests/test_outage.py::test_dynamodb_outage PASSED                                         [100%]

================================= 3 passed in 71.20s (0:01:11) ==================================
```

## Key Benefits

WIP

## Conclusion

LocalStack Chaos API allows you to further chaos test your other resources, such as Lambda functions, S3 buckets, and more to ascertain service continuity, user experience, and the system's resilience to the failures introduced, and how far you can go on to fix them. 
An ideal strategy is to design the experiments and group them in the categories of  **knowns**  and  **unknowns**, while analyzing whatever chaos your system might end up encountering.

In the upcoming blog posts, we'll demonstrate how to perform more complex chaos testing scenarios, such as RDS & Route53 failovers, inject network latency to every API call, and use AWS Resilience Testing Tools such as [AWS Fault Injection Service (FIS)](https://aws.amazon.com/fis/)  locally. 
Stay tuned for more blogs on how LocalStack is enhancing your chaos engineering experience!

You can find the code in this [GitHub repository](https://github.com/localstack-samples/sample-chaos-api-serverless).
