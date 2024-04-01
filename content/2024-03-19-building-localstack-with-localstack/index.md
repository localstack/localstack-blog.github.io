---
title: Building LocalStack with LocalStack
description: LocalStack Web Application allows developers to turbocharge their feedback cycles for their inner development processes. We're increasingly building various parts of the LocalStack Web Application using our core cloud emulator, leveraging various features for local multi-cloud development. In this blog, we share how we are dogfooding our own software to promote faster feature development and reduce inefficient testing loops.
lead: LocalStack Web Application allows developers to turbocharge their feedback cycles for their inner development processes. We're increasingly building various parts of the LocalStack Web Application using our core cloud emulator, leveraging various features for local multi-cloud development. In this blog, we share how we are dogfooding our own software to promote faster feature development and reduce inefficient testing loops.
date: 2024-03-19T9:21:02+05:30
lastmod: 2024-03-19T9:21:02+05:30
images: []
contributors: ["Lukas Pichler", "Harsh Mishra", "Vlad Gramuzov"]
tags: ['showcase']
---

## Introduction

[LocalStack Pro](https://twitter.com/localstack/status/1181338405315256320) was announced in 2019, shipping along with it the LocalStack Web application to encompass modern developer toolings & features to make local cloud development a breeze. As our team expanded and we envisioned a broader scope for the LocalStack product, we started dogfooding our software to leverage the same features our customers use LocalStack for. 

LocalStack’s core cloud emulator allows us to run our own cloud application - including its infrastructure - locally and provide an efficient developer experience across the entire software development lifecycle (SDLC). This experience enables us to build our product features in a way that closely matches what our customers are looking for — a comprehensive developer platform that facilitates local multi-cloud development across different providers and services.

In this blog, we highlight how we use the LocalStack core cloud emulator and other novel solutions, to build, test, and integrate new features in our LocalStack Web Application. We’ll also detail some of the lessons we have learned, recommendations for success, and how our experience has further helped us improve the base emulation layer.

## How do we enable local cloud development?

The LocalStack Web Application comprises two central components — the client application and the related backend. Our whole infrastructure is hosted on Amazon Web Services (AWS) and is deployed using the Cloud Development Kit (CDK). We use various AWS services, such as Lambda, S3, SNS, SQS, CloudFront, DynamoDB, ECS, EC2, Cognito, and Secrets Manager, to name just a few. We use ReactJS & Typescript for our client application while using Flask & Python for the backend.

The complexity of our cloud infrastructure and various managed dependencies mean that there is no straightforward way of testing it. While [AWS’s official recommendations](https://docs.aws.amazon.com/prescriptive-guidance/latest/best-practices-cdk-typescript-iac/development-best-practices.html) push us forward to using assertions and snapshot tests, there are inherent limitations and hurdles such as protracted deployment periods and expensive cloud resources.

### Infrastructure deployments & testing
  
With our core cloud emulator, we can run our entire cloud application - including its infrastructure - on our local machines. We are using [`cdklocal`](https://github.com/localstack/aws-cdk-local), our open-source wrapper script around the CDK library, to run our CDK deployments against LocalStack. Here are the commands we execute to bootstrap the local developer environment and deploy both our frontend and backend stacks on developer machines.

```bash
cd backend
export AWS_ACCOUNT_ID=000000000000 AWS_DEFAULT_REGION=eu-central-1
cdklocal bootstrap aws://$$AWS_ACCOUNT_ID/$$AWS_DEFAULT_REGION
cdklocal deploy --require-approval=never 
cd ../frontend
cdklocal deploy --require-approval=never
```

The key tenet of our local cloud development model is agility — deploying our CDK stack on AWS for development & testing used to take around 15 minutes. With LocalStack, we were able to cut it down less than sixty seconds. It enables a quick feedback loop and confidence with *our app runs locally!* while ensuring we are not handcuffed, as we deploy our applications dozens of times a day locally.

We also run our integration test suite against the locally deployed cloud infrastructure. This includes testing E2E flows encompassing Lambdas & SQS queues, Cognito triggers and authentication flows, alongside a DynamoDB-powered persistence layer with asynchronous stream handlers. The local integration suite enables us to further get rid of cloud-based developer environments, and use emulated resources to test our infrastructure locally, with highest level of fidelity.

### Development & debugging

Apart from this, we further wished to leverage LocalStack’s debugging tools in our development process. We were able to incorporate [Lambda Hot Reloading](https://docs.localstack.cloud/user-guide/lambda-tools/hot-reloading/) & [ECS Code Mounting](https://docs.localstack.cloud/user-guide/aws/ecs/#mounting-local-directories-for-ecs-tasks).

With Lambda Hot Reloading, we can continuously apply and test code changes to our locally running Lambda functions, removing the need for any code uploads or function re-deployments. This is especially useful during development, as well as our extensive integration and acceptance test suite, where developers can iterate quickly without the need to wait for code changes to be applied.

For example, with just a few lines of code, our CDK stack can be enabled to use the Hot Reloading feature mentioned above:

```python
if enable_hot_reloading():
    lambda_bucket = s3.Bucket.from_bucket_name(scope, f"lambda_name_local", "hot-reload")
    lambda_code = lambda_.Code.from_bucket(bucket=lambda_bucket, key=HOT_RELOADED_CODE_PATH) # path pointing to our root project directory
```

With just a few of many LocalStack features, we streamline our developer experience and make the setup independent of the cloud:

-   Our infrastructure, containing our lambdas, is deployed in hot-reload mode, which makes LocalStack watch the lambda code for any changes
-   We can trigger these lambdas either during integration tests or by invoking them manually — or through the locally running web application.
-   We can make on-the-fly changes to the function and subsequent executions of the affected lambda will change depending on the adjustments made.

Our team can furthermore benefit from hot-reloading qualities by incorporating LocalStack’s ECS features. We can mount our backend code from the host filesystem into the ECS containers. Similar to Lambda hot reloading it makes development a breeze, enabling faster development loops and increased debuggability where the changes are made without having to build and (re-) deploy any infrastructural changes or even Docker images each time. 

Here is an example, where we register a task definition, mounting a host path `/host/path` into the container under `/container/path`:

// code

### LocalStack Extensions

The idea of LocalStack Extensions is to provide a straightforward way to start custom service emulators together with LocalStack. As part of our effort to improve the user experience for extensions, we have released a couple of new extensions that we also actively use internally. Our development & testing workflows make use of the Stripe and MailHog extensions - both locally and in CI. We configure LocalStack to start with these extensions automatically by setting the following environment variable:

```bash
EXTENSION_AUTO_INSTALL=localstack-extension-mailhog, localstack-extension-stripe
```

// picture

The LocalStack Web Application handles various aspects around account management, such as Purchases, Subscriptions, Billing, and more. The Stripe extension fully allows us to test these flows, using an emulated Stripe service that runs on our machines locally. With the Stripe extension, we can now test user flows like purchasing a subscription or updating billing details, and other Stripe API operations. 

Routing the calls to the locally running Stripe emulator is achieved by simply overriding the API endpoint of the Stripe SDK, depending on whether we’re running locally.

```python
stripe_sdk.api_base = “http://localhost:8420” if is_local_development else “https://api.stripe.com”
stripe_sdk.Customer.create(...)
```

In the case of testing our checkout flow, which means purchasing a subscription, we can make the necessary calls to the Stripe extension, and check whether postconditions are fulfilled.

```python
def test_subscribe(self):
    subscription = self.stripe_provider.subscribe(
        plan=TeamPlan,
    )
    assert subscription
    assert subscription.plan.id == TeamPlan.id
    ...
```

The Mailhog extension allows us to emulate a local email server for testing user flows that require our platform to send emails, such as account activation, trial expiry notifications, and much more. Using this extension automatically configures LocalStack to use the MailHog SMTP server when sending emails. This means that any mails we send from our application logic, ends up in the mailbox of MailHog, which we can then either view via the UI or fetch via the API that comes with the extension.

After instructing LocalStack to start with the Mailhog extension, it automatically starts the MailHog service on port 25. Then, we just need to adjust our application to connect to the SMTP host running on port 25 locally.

```python 
smtp_host = "localhost:25"
```

// picture

## How do we use LocalStack in CI?

By running our cloud deployment & test suite locally, we were able to demystify critical pain points of the local cloud developer experience, which further helped us improve the parity, performance, and robustness of our core cloud emulator. However, we wanted to extend that improved developer experience across continuous integration (CI) pipelines with LocalStack. While it is easy just to use LocalStack as a drop-in replacement for AWS, and run tests just like we would do it locally, it is hard to retrieve detailed API telemetry, critical CI analytics, and discover flaky tests that need remediation.

// picture

This led us to embark on a journey to identify the missing puzzle pieces in the LocalStack CI experience. It made us build internal homegrown systems, which have now spun into critical LocalStack features that we continue to leverage for our CI pipelines.

### LocalStack GitHub Actions

We primarily use GitHub Actions to build, deploy, and test our web application & backend. Previously, setting up LocalStack on GitHub Actions (or any CI provider in general) was a pain, which required pulling the Docker image, installing the `localstack` CLI and other associated tools, before you could start LocalStack. To simplify this process, we created the [`setup-localstack` GitHub Action](https://github.com/localstack/setup-localstack) which:

-   Pulls the `latest` - or a specific - version of the LocalStack Docker image
-   Installs the `localstack` CLI alongside setting up configurations & wrapper scripts 
-   Starts the LocalStack container with or without the pro capabilities, depending on whether a valid CI Key is provided

The GitHub Action allowed us to migrate from our existing Docker Compose setup to using the following workflow step:

```yaml
- name: Start LocalStack
  uses: LocalStack/setup-localstack@v0.1.2
  with:
    image-tag: 'latest'
    use-pro: 'true'
    configuration: EXTRA_CORS_ALLOWED_ORIGINS=*
  env:
    LOCALSTACK_API_KEY: ${{ secrets.LOCALSTACK_API_KEY }}
```

### State Snapshots with Cloud Pods

LocalStack is ephemeral, which means that all state is gone when the container is stopped. However, we wanted to leverage our mechanism that can restore the emulator to a particular state before we run our tests against it. This is possible with two options:

-   Running an initialization hook or an infrastructure-as-code (IaC) deployment against the emulator.
-   Using a state snapshot that restores a previously-created state and pre-seed it in a test environment.

LocalStack’s persistence mechanism (enabled via `PERSISTENCE=1`) was useful for local development & testing needs. However, we further wanted to leverage state snapshots that can be stored, versioned, and shared across different development & testing environments. Cloud Pods are a mechanism to save LocalStack state onto a remote backend, allowing to restore infrastructure and state of various services when required.

// picture

Using Cloud Pods, we were able to cut down the total infrastructure deployment time from a minute to less than 10 seconds, both locally and in our CI pipelines. To maintain an up-to-date version of the cloud pod, we have a GitHub action which creates a pod with the latest infrastructure, that’s triggered on each merge to the `main` branch. We then use that pod in combination with our auto-loading Cloud Pods feature, which allows us to load cloud pods on the start-up of LocalStack automatically.

```yaml
AUTO_LOAD_POD=localstack-backend-pod
```

Now, when LocalStack starts up, our whole backend will be loaded from the pod, and we can immediately run our integration test suite against it. 

### Continuous Integration (CI) Analytics

With the LocalStack v3 release, we released a private preview of our CI Analytics offering. CI Analytics allow us to collect, analyze, and visualize critical metrics from our CI pipelines, helping us understand the impact of cloud infrastructure changes on CI builds.

// banner  

This allowed us to get detailed insights and traceability across the CI pipeline run by:

-   Recording all interactions throughout a CI build to get a detailed timeline of API calls using Stack Insights.
-   Select and drill into the infrastructure & application state at a particular point of execution using Cloud Pods.
-   Correlate the timeline of API calls with state changes, to identify hot spots and defect root causes.

This can be enabled by just setting a simple configuration variable in your LocalStack GitHub Action (or any other CI provider in general):

```yaml
LS_CI_PROJECT=name-of-your-project
```

With CI Analytics we can drill down into the request & response traces for every AWS API call that we make with our integration test suite. With the help of CI Analytics, we have brought together the critical missing pieces of CI observability & analytics into one single feature which has massively improved our CI troubleshooting and debugging experience.

Additionally, we can now instrument the important paths and processes, capture the infrastructure state which can be restored locally, and enrich our API telemetry to capture relevant data that help our developers understand flaky CI tests. 

// picture

## How do we use LocalStack to enable application previews and E2E testing?

The next step in our SDLC after local development/testing and running our test suite in CI, are acceptance tests through application previews, and e2e tests including our web UI. After running our integration tests, both locally and in CI, the next step was to deploy the CDK stack in our staging environment. The staging environment allowed us to run our end-to-end (E2E) integration test suite, which rely on the Playwright framework, and further use it for acceptance testing, to get alignment across cross-departmental projects. It allowed us to achieve the final degree of validation before we shipped a new release to production.

With the LocalStack v3 release, we released a private preview of Ephemeral Instances. These ephemeral instances allow us to run a short-lived encapsulated instance of LocalStack in the cloud. It allows us to run our E2E tests, preview features in our cloud application, and collaborate asynchronously within and across the team.

// picture

With these ephemeral instances, we can now deploy our entire application (frontend, backend, and infrastructure) on an ephemeral instance, and expose the instance to us internally, which allows us to test our changes with every pull request. This has allowed us to replace our staging environments with ephemeral instances, which we use to continuously run automated tests and check out individual features in parallel manually.

Here is how we configured our GitHub Action pipeline to spin up an ephemeral instance for our application changes with every pull request:

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v4

  - name: Deploy Preview
    uses: LocalStack/setup-localstack/preview@main
    with:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      localstack-api-key: ${{ secrets.LOCALSTACK_API_KEY }}
      preview-cmd: |
        npm install -g aws-cdk-local aws-cdk
        pip install awscli-local[ver1]
        cd backend
        export AWS_ACCOUNT_ID=000000000000 AWS_DEFAULT_REGION=eu-central-1
        cdklocal bootstrap aws://$$AWS_ACCOUNT_ID/$$AWS_DEFAULT_REGION
        cdklocal deploy --require-approval=never 
        cd ../frontend
        cdklocal deploy --require-approval=never

  - name: Finalize PR comment
    uses: LocalStack/setup-localstack/finish@main
    with:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      include-preview: true
```

This enables us to:

-   Foster active collaboration with GTM, RevOps, and DevRel teams to demonstrate new features to prospective customers.
-   Remove the need for staging environments, by giving every engineer an isolated environment, essentially unblocking ourselves.
-   Cut down costs around our staging environment by tearing down environments automatically, which avoids unnecessary cloud costs.
    
With the help of auto-loaded Cloud Pods to pre-seed the infrastructure state, we have achieved a game-changing improvement to our pre-release testing due to faster application spin-ups & resource allocation.

Though application previews have been ubiquitous in the frontend space, LocalStack can now spin up your entire application, including any infrastructure relying on AWS services, and ship you a live deployment with every change. This first-class preview-on-pull request support is instrumental in helping LocalStack become a true cloud development platform — not only for developers, but also for QA & management, to fully become the backbone of cloud development throughout the entire SDLC.

## Conclusion

That’s the long and short of how we are building LocalStack with LocalStack. LocalStack has enabled rapid design and development of sophisticated solutions by reducing the number of test and UAT environments while improving the quality and lead time. The best way we can improve our product is to iteratively adopt it, and ensure we can leverage the same features as our customers do and continue to nail down the developer experience. Over many months, we have continued to ship improvements to enable teams, like ours, to scale and mitigate common bottlenecks while developing on the cloud.

As we continue our work in fleshing out the LocalStack experience, we aim to further support enterprise compliance & insights, with features like Chaos engineering, Productivity metrics, Cost optimizations, and more. This will allow us to expand from our initial focus on the inner dev loop to an outer dev loop experience, to accelerate your cloud journey and to put developers back in charge. Building a cloud emulator is hard, and this sets us up towards solving larger problems at hand — state management, SDLC, collaboration, and more.

Stay tuned for more news and awesome features in the upcoming months — or if you would like to get access to some of the features we’ve been using, get in touch with us.