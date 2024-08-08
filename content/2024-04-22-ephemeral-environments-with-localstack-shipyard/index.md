---
title: Ephemeral Environments with LocalStack & Shipyard
description: We're excited to announce our partnership with Shipyard. This integration empowers you to test your cloud applications in a short-lived, encapsulated deployment — allowing you to shift left by fixing things before they break your production.
lead: We're excited to announce our partnership with Shipyard. This integration empowers you to test your cloud applications in a short-lived, encapsulated deployment — allowing you to shift left by fixing things before they break your production.
date: 2024-04-22
lastmod: 2024-04-22
images: ['ephemeral-environments-with-localstack-shipyard-banner.png']
leadimage: 'ephemeral-environments-with-localstack-shipyard-banner.png'
contributors: ["Harsh Mishra", "Natalie Lunbeck"]
tags: ['showcase']
---

{{< img-simple src="ephemeral-environments-with-localstack-shipyard-banner.png" width=300 alt="Banner image for the blog: Ephemeral environments with LocalStack & Shipyard">}}

## Introduction

We’re excited to announce our partnership with [Shipyard](https://shipyard.build), the ephemeral environment self-service platform, that allows you to spin up on-demand preview deployments via Kubernetes clusters to turbocharge your application lifecycle. With Shipyard, you can now deploy your cloud applications in short-lived environments to enable developer teams to run tests, preview features, and get alignment with cross-departmental projects. Shipyard also enables application previews on every pull request and simplifies monitoring, debugging, and deployment all from one dashboard!

LocalStack’s core cloud emulator enables developers to build, test, and deploy cloud & serverless applications locally. Shipped as a Docker image, you can use various integrations such as the `docker` CLI, Docker Compose, or Helm to start LocalStack in a developer environment. Shipyard allows developers to use [Docker Compose configurations](https://docs.docker.com/compose/) which are then automatically transpiled into Kubernetes manifests enabling you a pre-production preview of how your applications work! As such, LocalStack does not require any additional configurations and just enables you to start with a pre-defined Docker Compose setup.

In this blog, we’ll detail why you should be using ephemeral environments and how you can use LocalStack alongside Shipyard. We’ll also go through setting up an AWS-powered application on Shipyard & LocalStack, and how you can pre-seed infrastructure state using Cloud Pods!

## What is Shipyard?

Shipyard is a platform that allows you to create on-demand ephemeral environments to run your tests against a production-like setup. It simplifies pre-production testing, compliance, and collaboration by generating an ephemeral environment preview for your full-stack application on every pull request (PR) to give you a near-production experience while building and testing features. Every environment is a single-tenant cluster, enabling Single Sign-On (SSO) authentication. This ensures that not only developers but also other stakeholders can quickly access and work on developing features with a simple click.

In addition, you can use Shipyard for:

-   Gaining terminal access into your running containers for live debugging.
-   Adding private container registries for pulling images during the build.
-   Integrating with GitHub Actions, CircleCI, Datadog, Slack, and more.
-   Creating snapshots for each named volume in your application setup.
-   Monitoring your deployment while tracking build details & application history.

LocalStack has been in active partnership with Shipyard and both companies are on a mission to streamline the Software Development Life Cycle (SDLC) and provide frictionless developer experience. This partnership is in line with our shared vision of bolstering the DevOps methodologies while providing an improved testing process, quicker release cycles, and optimized cost utilization — all while adhering to security best practices.

{{< tweet user="localstack" id="1670858503693402139" >}}

LocalStack’s cloud emulation capabilities allow you to create resources such as S3 buckets, DynamoDB tables, OpenSearch clusters, and more to replicate cloud environments. With significant recent enhancements to these services, you can now preview features in your cloud applications, and use additional features such as Cloud Pods to pre-seed your infrastructure state automatically. Upon creating a pull request, Shipyard automatically creates an ephemeral environment that allows you to test your application in a state that is infrastructure-identical to production, making it easy to collaborate asynchronously within and across your team!

## How to use Shipyard with LocalStack?

In this section, we’ll run a basic item tracker application on Shipyard using DynamoDB & Simple Email Service (SES) provisioned by LocalStack. The item tracker application allows users to submit data to a DynamoDB table using a ReactJS client and a Flask backend, using the [AWS SDK for Python (`boto3`)](https://aws.amazon.com/sdk-for-python/) using a basic CRUD interface. It then uses SES to mock the process of sending email reports of work items.

For this walkthrough, you’ll need to have the following prerequisites installed on your local machine:

-   [Docker](https://docs.docker.com/engine/install/) & [Docker Compose](https://docs.docker.com/compose/install/)
-   AWS CLI & [`awslocal` wrapper script](https://docs.localstack.cloud/user-guide/integrations/aws-cli/#localstack-aws-cli-awslocal)
-   [Shipyard account](https://shipyard.build/signup) with an active subscription (sign up for a free trial)
-   [LocalStack Web Application account](https://app.localstack.cloud/sign-up)
-   [`localstack` CLI](https://docs.localstack.cloud/getting-started/installation/#localstack-cli) with [`LOCALSTACK_AUTH_TOKEN`](https://docs.localstack.cloud/getting-started/auth-token/)

### Set up the application on your local machine

The code for the solution in this post is in [this repository on GitHub](https://github.com/localstack-samples/sample-item-tracker-shipyard-application). To get started, fork the repository on GitHub. You can now use `git clone` to clone the repository onto your local developer machine:

```bash
git clone https://github.com/localstack-samples/sample-item-tracker-shipyard-application
cd sample-item-tracker-shipyard-application
```

Replace the GitHub repository URL with the URL of the forked repository. Open the code in your favourite code editor/IDE. You’ll be able to check out the Docker Compose configuration that sets up the application on your local machine. Here is what it looks like:

```yaml
version: '3'

services:

  localstack:
    container_name: "${LOCALSTACK_DOCKER_NAME-localstack_main}"
    image: localstack/localstack-pro:latest
    ports:
      - "127.0.0.1:4566:4566"            # LocalStack Gateway
      - "127.0.0.1:4510-4559:4510-4559"  # external services port range
      - "127.0.0.1:443:443"
    environment:
      - LOCALSTACK_AUTH_TOKEN=${LOCALSTACK_AUTH_TOKEN:?}
      - DEBUG=1
      - DOCKER_HOST=unix:///var/run/docker.sock
      - EXTRA_CORS_ALLOWED_ORIGINS='*'
    volumes:
      - "./init-aws.sh:/etc/localstack/init/ready.d/init-aws.sh"
      - "localstack:/var/lib/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"
    networks:
      ls:
        ipv4_address: 10.0.2.20

  frontend:
    container_name: react
    labels:
      shipyard.route: '/'
      shipyard.primary-route: 'true'
    build: 'frontend'
    volumes:
      - './frontend/src:/app/src'
      - './frontend/public:/app/public'
    ports:
      - '3000:3000'

  backend:
    container_name: flask
    labels:
      shipyard.route: '/api'
    build: 'backend'
    environment:
      SHIPYARD_DOMAIN_FRONTEND: ${SHIPYARD_DOMAIN_FRONTEND-}
      SHIPYARD_DOMAIN: ${SHIPYARD_DOMAIN}
    ports:
      - '8080:8080'
    dns:
      - 10.0.2.20
    networks:
      - ls

networks:
  ls:
    ipam:
      config:
        - subnet: 10.0.2.0/24
volumes:
  localstack:
```

In the above Docker Compose configuration, we have created:

-   A `localstack` service which pulls the `localstack/localstack-pro` Docker image and sets up an initialization hook that creates the AWS resources on the host machine.
-   A `frontend` service that installs the dependencies & builds the ReactJS application served on port 3000 using the `Dockerfile` in the `frontend` directory.
-   A `backend` service that installs the dependencies and starts a development Flask server on port 8080 using the `Dockerfile` in the `backend` directory.
-   An `ls` Docker network that allows the `backend` service to use LocalStack as its DNS server and ensure that it can access the LocalStack container.

Additionally, you will notice that we have configured the Shipyard-specific service labels to specify the frontend domain, primary route, and main domain. This is useful in the context of a multi-domain environment like ours. You can check more about them on [Shipyard’s documentation](https://docs.shipyard.build/docs/docker-compose/#routing).

### Start the application on your local machine

You can now start the application locally, by specifying the following command:

```bash
export LOCALSTACK_AUTH_TOKEN=<your-auth-token>
docker-compose up
```

This will start the `localstack`, `backend`, and `frontend` services, exposing the user client to `localhost:3000`. You can navigate to the local user client to start interacting with the application by adding items which are fully persisted using the emulated DynamoDB table.

{{< img-simple src="item-tracker-application.png" alt="Picture showing the item tracker application working">}}

You can also use the REST API client to add items to the application by performing a `POST` request. Here is an example:

```bash
curl -X POST http://localhost:8080/api/items \
    -H "Content-Type: application/json" \
    -d '{"name":"Me","guide":"python","description":"Show how to add an item","status":"In progress","archived":false}'
```

You can also use the REST API to get any existing active items using the following command:

```bash
curl -X GET http://localhost:8080/api/items?archived=false
```

You can fetch an email report by adding [`hello@example.com`](mailto:hello@example.com) in the **Email Report** tab and clicking **Send report**. Navigate to the LocalStack Web Application, and you’ll be able to find the sent emails on the [SES Resource Browser](https://app.localstack.cloud/inst/default/resources/ses/identities).

{{< img-simple src="ses-resource-browser.png" alt="LocalStack SES Resource Browser">}}

You can also scan the DynamoDB table you have deployed using the following command:

```bash
awslocal dynamodb scan --table-name doc-example-work-item-tracker
```

In the output, you will find the persisted data available.

### Configure your application on Shipyard

You can now create an ephemeral environment for your cloud application on Shipyard. Navigate to your Shipyard dashboard and click on **+ Application**. Choose the GitHub repository you have forked and the `main` branch to start.

{{< img-simple src="create-application-shipyard.png" alt="Create an application on Shipyard">}}

Click on the **Select services** to add services that you would like to deploy. These services are automatically selected from the Docker Compose configuration.

{{< img-simple src="shipyard-select-services.png" alt="Selecting Docker Compose services on Shipyard">}}

Click on the **Add environment variables** to go to the next step, and add the `LOCALSTACK_AUTH_TOKEN` as an environment variable to authenticate with LocalStack. Finally, click on **Create application** to get started with the first build of your application! After a successful build, you can now click on the **Visit** button on your application dashboard to navigate to the deployed application. 

You can start interacting with the application, and review the run logs for your application's services.

{{< img-simple src="localstack-container-logs.png" alt="LocalStack Container logs on Shipyard">}}

### Enable previews for your ephemeral environments

You can now create previews for your ephemeral application deployed on Shipyard. The preview follows the lifecycle of a pull request (PR), and makes sure that:

-   A new ephemeral environment is deployed in Shipyard with the new commits in the PR.
-   The PR is updated with the deployed application URL and the build details for quick debugging.

The sample repository above has already been configured with a sample GitHub Action workflow. The workflow looks like this:

```yaml
name: Deploy on Shipyard

on: [pull_request]

jobs:
  deploy:
    runs-on: ubuntu-latest
    name: Deploy
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Integrate Shipyard
        uses: shipyard/shipyard-action@1.0.0
        env:
          SHIPYARD_API_TOKEN: ${{ secrets.SHIPYARD_API_TOKEN }}
          SHIPYARD_APP_NAME: "localstack-shipyard-item-tracker"
```

The workflow uses the `SHIPYARD_API_TOKEN`. Add it to your forked GitHub repository using secrets in GitHub Actions.

Now create a new branch in your local repository, and make a small change. After making the changes, stage it, push the commits into the branch and create a new pull request. Shipyard will automatically start a new ephemeral deployment. You can see the workflow's status and logs in the `checks` section of the pull request. After a few seconds, the workflow will add the preview URL. Click on it to see your changes in real time.

{{< img-simple src="shipyard-pr-previews.png" alt="Shipyard spinning up another environment for the new Pull request">}}

### Use Cloud Pods to pre-seed infrastructure state

In various testing scenarios, you might often feel the need to create test fixtures or additional resources to bootstrap your testing environment and test your application. [LocalStack’s Cloud Pods](https://docs.localstack.cloud/user-guide/state-management/cloud-pods/) can facilitate and dramatically simplify this task. Cloud Pods allow you to take a snapshot of the state at any point in time, and then selectively restore, merge, and inject it into your LocalStack container.

In this scenario, you can pre-seed your previews with the DynamoDB state using a Cloud Pod that contains the state and configuration of the DynamoDB table. This Cloud Pod would be available in the Cloud Pod storage space on the LocalStack Web Application and can be used in CI to bootstrap the testing environment where the application is then deployed and tested. Moreover, each developer can pull the same Cloud Pod and run some local tests.

To create a Cloud Pod, navigate to the local application and run the following script that inserts some sample data into your DynamoDB table:

```bash
./scripts/seed.sh
```

You can now take an infrastructure snapshot with the Cloud Pods using the following command:
  
```bash
localstack pod save item-tracker-application
```

You will see the following output in your terminal:

```bash
Cloud Pod `item-tracker-application` successfully created ✅
Version: 1
Remote: platform
Services: dynamodb,ses
```

You can also navigate to the [Cloud Pods browser](https://app.localstack.cloud/pods), where you can find the newly created Cloud Pod stored on the LocalStack Web Application. Navigate to the local application setup, and add the following to your Docker Compose configuration file to auto-load the Cloud Pod, using the [`AUTO_LOAD_POD` configuration](https://docs.localstack.cloud/user-guide/state-management/cloud-pods/#environmental-variables), and remove the initialization hook. 

```yaml
AUTO_LOAD_POD=item-tracker-application
```

This will ensure that on the LocalStack container startup, the Cloud Pod will be loaded automatically thus pre-seeding your infrastructure state. The Docker Compose configuration will look like this:

```yaml
environment:
  - LOCALSTACK_AUTH_TOKEN=${LOCALSTACK_AUTH_TOKEN:?}
  - DEBUG=1
  - DOCKER_HOST=unix:///var/run/docker.sock
  - EXTRA_CORS_ALLOWED_ORIGINS='*'
  - AUTO_LOAD_POD=item-tracker-application
volumes:
  - "localstack:/var/lib/localstack"
  - "/var/run/docker.sock:/var/run/docker.sock"
```

Commit and push this on the `main` branch of the repository. After a successful build & deployment, you can now visit the newly deployed application, to find the DynamoDB table successfully seeded in the application setup, which is further reflected in the web client.

{{< img-simple src="application-cloud-pods.png" alt="Application pre-seeded with the Cloud Pods">}}

In addition to preseeding your Shipyard environment, Cloud Pods can also be used at the “other end” of a pipeline, namely to store and push the state of the LocalStack instance after an ephemeral run has been completed. This allows you to replicate the same state onto the local machine, in case your integration test suite fails on the ephemeral environment.

## Key Benefits

There are several key benefits of using Shipyard and LocalStack to develop & test cloud applications:

-   Improve the development process for cloud infrastructure by allowing developers to test code changes in isolation and iterate quickly.
-   Save time and money by eliminating the need to wait for staging to be configured and blocking other team members during the process.
-   Facilitate collaboration across different teams by providing a freshly-configured pre-production environment for everyone to work in.
-   Run your automated end-to-end (E2E) test suite and pre-seed your infrastructure & application state to allow more thorough testing.
-   Streamline the login process (SSO) for your team to prevent anyone outside your organization from accessing your environments.
    

## Conclusion

Congratulations! You’ve successfully deployed an AWS-powered cloud application on an ephemeral environment using LocalStack & Shipyard. With LocalStack, you don't have to worry about losing your infrastructure state after tearing down your ephemeral environments after a quick round of testing. LocalStack allows you to persist the resources created on the host machine for extensive testing and further use Cloud Pods to pre-seed the infrastructure state as and when required!

You can further explore Shipyard and their offering for various use cases such as:

-   Adding [Datadog logging](https://docs.shipyard.build/docs/datadog) and integrating with LocalStack’s serverless resources, such as [Lambda](https://docs.localstack.cloud/user-guide/aws/lambda/), [EventBridge](https://docs.localstack.cloud/user-guide/aws/eventbridge/), [SQS](https://docs.localstack.cloud/user-guide/aws/sqs/), and more.
-   Using [Shipyard’s volume management](https://docs.shipyard.build/docs/volume-management) to revert the data in case of regression and load them across sibling environments.
-   Monitoring key infrastructure metrics such as an overview of [build and deploy times, CPU/Memory usage, and deployment timeline](https://docs.shipyard.build/docs/build-details).

If you have any questions about configuring and running your project, drop by [Shipyard](https://shipyardcommunity.slack.com/join/shared_invite/zt-1y44cpq6u-rJT~kg9wArqxP~N1F3K_pA#/shared-invite/email) or [LocalStack](https://localstack.cloud/slack) community. We would love to hear your feedback about this integration. Happy cloud testing!
