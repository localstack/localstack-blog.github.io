---
title: Building LocalStack with LocalStack
description: We are increasingly building various parts of the LocalStack Web Application using our core cloud emulator, leveraging various features for local multi-cloud development. In this blog, we share how we are dogfooding our own software to promote faster feature development and reduce inefficient testing loops.
lead: We are increasingly building various parts of the LocalStack Web Application using our core cloud emulator, leveraging various features for local multi-cloud development. In this blog, we share how we are dogfooding our own software to promote faster feature development and reduce inefficient testing loops.
date: 2024-03-19T9:21:02+05:30
lastmod: 2024-03-19T9:21:02+05:30
images: []
show_cta_1: true
contributors: ["Lukas Pichler", "Harsh Mishra", "Vlad Gramuzov"]
tags: ['showcase']
---

## Introduction

[LocalStack Pro was announced in 2019](https://twitter.com/localstack/status/1181338405315256320), shipping along with it the [LocalStack Web application](https://app.localstack.cloud) to support developer toolings & features to make local cloud development a breeze. As our team expanded and we envisioned a broader scope for the LocalStack product, we started dogfooding our software to leverage the same features our customers use LocalStack for. 

[LocalStack’s core cloud emulator](https://docs.localstack.cloud/getting-started/installation/) allows us to run our own cloud application - including its infrastructure - locally, which provides an efficient developer experience at the start of the entire software development lifecycle (SDLC). This experience enables us to build our product features in a way that closely matches what our customers are looking for — a comprehensive developer platform that facilitates local multi-cloud development across different providers and services.

In this blog, we highlight how we use the LocalStack core cloud emulator and other novel solutions, to build, test, and integrate new features in our LocalStack Web Application. We’ll also detail some of the lessons we have learned, recommendations for success, and how our experience has further helped us improve the base emulation layer.

- [Application Overview](#application-overview)
- [How do we enable local cloud development?](#how-do-we-enable-local-cloud-development)
- [How do we use LocalStack in CI?](#how-do-we-use-localstack-in-ci)
- [How do we use LocalStack to enable application previews and E2E testing?](#how-do-we-use-localstack-to-enable-application-previews-and-e2e-testing)
- [Conclusion](#conclusion)

## Application Overview

The LocalStack Web Application comprises two central components — the client application and the related backend. Our whole infrastructure is hosted on [Amazon Web Services (AWS)](https://aws.amazon.com/) and is deployed using the [Cloud Development Kit (CDK)](https://aws.amazon.com/cdk/). We use various AWS services, such as [Lambda](https://aws.amazon.com/lambda/), [S3](https://aws.amazon.com/s3/), [SNS](https://aws.amazon.com/sns/), [SQS](https://aws.amazon.com/sqs/), [CloudFront](https://aws.amazon.com/cloudfront/), [DynamoDB](https://aws.amazon.com/dynamodb/), [ECS](https://aws.amazon.com/ecs/), [EC2](https://aws.amazon.com/ec2/), [Cognito](https://aws.amazon.com/cognito/), [Secrets Manager](https://aws.amazon.com/secrets-manager/), to name just a few. We use [ReactJS](https://react.dev/) & Typescript for our client application while using [Flask](https://flask.palletsprojects.com/en/3.0.x/) & Python for the backend.

{{< img-simple src="web-application-architecture.png" alt="LocalStack Web Application architecture">}}

The complexity of our cloud infrastructure and various managed dependencies mean that there is no straightforward way of testing it. While [AWS’s official recommendations](https://docs.aws.amazon.com/prescriptive-guidance/latest/best-practices-cdk-typescript-iac/development-best-practices.html) push us forward to using fine-grained assertions and snapshot tests, there are inherent limitations and hurdles such as protracted deployment periods and expensive cloud resources.

## How do we enable local cloud development?

When we started our cloud application development, we had a long test and release process that used a dedicated staging environment to test the latest changes and make adjustments before moving them to production. By allowing us to develop our application entirely on our developer machines, we reduced the time spent waiting for feedback, as there are no real resources or deployment involved.

This section will detail parts of our setup to help you implement a similar local development flow.

### Boto3 Configuration

Our Flask backend connects to various AWS resources using `boto3`. To integrate LocalStack, we use a simple configuration when creating the `boto3` client. This configuration determines if `boto3` connects to LocalStack during development or to actual AWS services in staging/production environments.

We use [Dynaconf](https://www.dynaconf.com/) for configuration management, enabling us to set different settings for each environment. The following example shows the default AWS settings for production:

```bash
[default]
aws.endpoint_url = ""
```

For local development, we point `boto3` to LocalStack by setting the AWS endpoint URL to LocalStack's address:

```bash
[development]
aws.endpoint_url = "http://localhost.localstack.cloud:4566"
```

These variables are used when creating the boto3 client as follows:

```
def get_aws_config(self) -> AWSClientConfig:
    endpoint_url = None

    if settings.get("aws.endpoint_url"):
        endpoint_url = settings.get("aws.endpoint_url")

    return AWSClientConfig(
        endpoint_url=endpoint_url,
    )

class LambdaClient(AWSClient):
    def __init__(self, config: AWSClientConfig):
        config = get_aws_config()
        self.client = boto3.client("lambda", config.dict())
```

This setup routes all boto3 calls in our backend to LocalStack instead of real AWS services when developing locally.

### Infrastructure deployment & testing

With the configuration described earlier, we connected the Python backend to local resources on LocalStack, such as DynamoDB. However, our infrastructure includes serverless services like Lambdas, ECS, and EC2. The next logical step was to deploy the entire infrastructure onto LocalStack to develop and test the whole application locally.
  
We are using [`cdklocal`](https://github.com/localstack/aws-cdk-local), our open-source wrapper script around the CDK library, to deploy our CDK stacks against LocalStack - our core cloud emulator. To achieve this, we first bootstrap the environment, and deploy the backend stack afterwards.

```bash
cd backend
export AWS_ACCOUNT_ID=000000000000 AWS_DEFAULT_REGION=eu-central-1
cdklocal bootstrap aws://$$AWS_ACCOUNT_ID/$$AWS_DEFAULT_REGION
cdklocal deploy --require-approval=never 
```

This set of commands is wrapped up in a single Makefile target, which can then be invoked with one CLI command:

```bash 
make deploy-local
```

To deploy the frontend to LocalStack as well, we have a similar sequence of commands:
```bash
cd ../frontend
cdklocal deploy --require-approval=never
```

The key tenet of our local cloud development model is agility — deploying our CDK stack on AWS for development & testing **used to take around 15 minutes**. With LocalStack, we were able to cut it down to **less than sixty seconds**. It enables a quick feedback loop and confidence with *our app runs locally!* while ensuring we are not handcuffed, as we deploy our applications dozens of times a day locally.

{{< img-simple src="localstack-web-app-running-locally.png" alt="LocalStack Web App running locally">}}

With the ability to deploy our application locally we are able to develop and test various flows manually, for example if one can sign up, which involves Cognito and a few Lambdas.

Deploying the application locally allows us to run our integration test suite against it. This includes testing end-to-end flows that involve Lambdas and SQS queues, Cognito triggers and authentication flows, and a DynamoDB-powered persistence layer with asynchronous stream handlers. The local integration suite enables us to eliminate cloud-based developer environments and use emulated resources to test our infrastructure locally with the highest level of fidelity.

For example, in the signup process, our application logic performs the following steps:

* Create a new user in the database.
* Create a user in Cognito.
* Send an email to the user to confirm their account creation.

With our infrastructure deployed, which we set up before running our test suite, we use pytest to implement tests that assert the behaviors described above.

```python
def test_signup_user(self, smtp_server):
    user_email = f"{short_uid()}@localstack.cloud"

    response = requests.post(
        url=API_PATH_SIGNUP, # path that points to signup endpoint
        json=dict(
            email=user_email,
            password="TestingPassword123!",
        ),
    )

    assert response.status_code == 200
    assert response.json["id"] is not None
    assert response.json["email"] == user_email

    user_exists = cognito_client.admin_get_user(
      UserPoolId=USER_POOL_ID, # user pool id for cognito
      Username=user_email
    )
    assert user_exists is not None

    messages = smtp_server.list_messages()
    assert len(messages) == 1
    assert messages[0]['Subject'] == 'LocalStack Account Activation'
```

### Development & debugging

Apart from this, we further wanted to leverage LocalStack’s debugging tools in our development process. We were able to incorporate [Lambda Hot Reloading](https://docs.localstack.cloud/user-guide/lambda-tools/hot-reloading/) & [ECS Code Mounting](https://docs.localstack.cloud/user-guide/aws/ecs/#mounting-local-directories-for-ecs-tasks).

With just a few of many LocalStack features, we streamline our developer experience and improve the local development setup even further:

-   Our infrastructure, containing our Lambdas, is deployed in hot-reload mode, which makes LocalStack watch the Lambdas for any changes.
-   We can trigger these Lambdas either during integration tests or by invoking them manually — or through the locally running web application.
-   We can make on-the-fly changes to the function and subsequent executions of the affected Lambda will change depending on the adjustments made.

This is especially useful during development, as well as our extensive integration test suites, where developers can iterate quickly without the need to wait for code changes to be applied.

With the following lines of code, our CDK stack can be enabled to use the Hot Reloading feature mentioned above:

```python
lambda_bucket = s3.Bucket.from_bucket_name(scope, f"lambda_name_local", "hot-reload")
lambda_code = lambda_.Code.from_bucket(bucket=lambda_bucket, key=HOT_RELOADED_CODE_PATH) # path pointing to our root project directory
```

Our team furthermore benefits from hot-reloading qualities by incorporating LocalStack’s ECS features. We can mount our backend code from the host filesystem into the ECS containers. Similar to Lambda Hot Reloading it makes development a breeze, enabling faster development loops and increased debuggability where the changes are made without having to build and (re-) deploy any infrastructural changes or even Docker images each time. 

Here is an example, where we register a task definition, mounting the host path `./localstack_platform` into the container under `/app/localstack_platform`:

```python3
task_def = ecs.TaskDefinition(
        ...
        volumes=[
                ecs.Volume(name="test-volume", host=ecs.Host(source_path="./localstack_platform"))
        ]
)

platform_container = task_def.add_container(...)

platform_container.add_mount_points(
        ecs.MountPoint(
                container_path="/app/localstack_platform",
                source_volume="test-volume",
                read_only=False,
        ),
)
```

The CDK enhancements are wrapped in what we call "if-local blocks," which are only executed when CDK detects that it is in a local environment.

```python
if is_local_development:
  # Do something that's only needed for the local deployment
```

Of course, we don't want similar mount points for ECS tasks or buckets named `hot-reload` in production, so be sure to use "if-local blocks" where necessary!

### LocalStack Extensions

[LocalStack Extensions](https://docs.localstack.cloud/user-guide/extensions/) provide a straightforward way to start custom service emulators together with LocalStack. As part of our effort to improve the user experience for extensions, we have released a couple of new extensions that we also actively use internally. 

Our development & testing workflows make use of the [Stripe](https://pypi.org/project/localstack-extension-stripe/) and [Mailhog](https://pypi.org/project/localstack-extension-mailhog/) extensions - both locally and in CI. We configure LocalStack to start with these extensions automatically by setting the following environment variable:

```bash
EXTENSION_AUTO_INSTALL=localstack-extension-mailhog,localstack-extension-stripe
```

The LocalStack Web Application handles various aspects around account management, such as Purchases, Subscriptions, Billing, and more. The Stripe extension fully allows us to test these flows, using an emulated Stripe service that runs on our machines locally. With the Stripe extension, we can now test user flows like purchasing a subscription or updating billing details, and other Stripe API operations. 

Routing the calls to the locally running Stripe emulator is achieved by making use of Dynaconf again, which sets the API endpoint of the [Stripe SDK](https://docs.stripe.com/libraries) to the locall running stripe extension, during development.

```python
[default]
stripe.api_base = "https://api.stripe.com"

[development]
stripe.api_base = "http://localhost.localstack.cloud:8420"
```

When initializing our stripe provider, we simply override the API endpoint of the [Stripe SDK](https://docs.stripe.com/libraries) with the value in our Dynaconf config, and hence are able to split the application code from different environments.

In the case of testing our checkout flow, which means purchasing a subscription, calls to Stripe are automatically routed to the locally running extension, and we can afterwards check whether postconditions are fulfilled - all done locally.

```python
def test_subscribe(self):
    subscription = self.stripe_provider.subscribe(
        plan=TeamPlan,
    )
    assert subscription
    assert subscription.plan.id == TeamPlan.id
    ...
```

The Mailhog extension allows us to emulate a local email server for testing user flows that require our platform to send emails, such as account activation, trial expiry notifications, and much more. Using this extension automatically configures LocalStack to use the Mailhog SMTP server when sending emails over SES. 

This means that any mails we send from our application logic ends up in the mailbox of Mailhog, which we can then either view via the user interface or fetch via the API that comes with the extension.

After instructing LocalStack to start with the Mailhog extension, it automatically starts the Mailhog service on port 25. Then, we just need to adjust our application to connect to the SMTP host running on port 25 locally, again with the help of Dynaconf.

```python 
[default]
smpt_host = "email-smtp.eu-central-1.amazonaws.com:587"

[development]
smpt_host = "localhost.localstack.cloud:25"

# used in application code:

s = self._connect_smtp(smtp_host, smtp_user, smtp_pass)
s.sendmail()
```
On below image you can see the user interface of the Mailhog extension, displaying the email which is sent when signing up for an account.
{{< img-simple src="localstack-mailhog-extension.png" alt="LocalStack Mailhog extension">}}

An example on how we write tests against Mailhog has been given in the previous section called [Infrastructure deployment & testing](##infrastructure-deployments--testing).

## How do we use LocalStack in CI?

By running our cloud deployment & test suite locally, we were able to demystify critical pain points of the local cloud developer experience, which further helped us improve the parity, performance, and robustness of our core cloud emulator. However, we wanted to extend that improved developer experience across continuous integration (CI) pipelines with LocalStack. While it is easy just to use LocalStack as a drop-in replacement for AWS, and run our test suites just like we do it locally, it is hard to retrieve detailed API telemetry, critical CI analytics, and discover flaky tests that need remediation.

This led us to embark on a journey to identify the missing puzzle pieces in the LocalStack CI experience. It made us build internal homegrown systems, which have now spun into critical LocalStack features that we continue to leverage for our CI pipelines.

### LocalStack GitHub Action

We primarily use [GitHub Actions](https://github.com/features/actions) to build, deploy, and test our web application & backend. Previously, setting up LocalStack on GitHub Actions (or any CI provider in general) was a pain, which required pulling the Docker image, installing the `localstack` CLI and other associated tools, before you could start LocalStack. To simplify this process, we created the [`setup-localstack` GitHub Action](https://github.com/localstack/setup-localstack) which:

-   Pulls the `latest` - or a specific - version of the LocalStack Docker image
-   Installs the `localstack` CLI alongside setting up configurations & wrapper scripts 
-   Starts the LocalStack container with or without the pro capabilities, depending on whether a valid CI Key is provided

The GitHub Action allowed us to set up LocalStack and related tooling for running our test suite in CI by using a simple workflow step:

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

LocalStack is ephemeral, which means that all state is gone when the container is stopped. However, we wanted to leverage our mechanism that can restore the emulator to a particular state before we run our tests against it to enable various test scenarios. This is possible with two options:

-   Running an initialization hook or an infrastructure-as-code (IaC) deployment against the emulator.
-   Using a state snapshot that restores a previously-created state and pre-seed it in a test environment.

[LocalStack’s persistence mechanism](https://docs.localstack.cloud/user-guide/state-management/persistence/) (enabled via `PERSISTENCE=1`) was useful for local development & testing needs. However, we further wanted to leverage state snapshots that can be stored, versioned, and shared across different development & testing environments. [Cloud Pods](https://docs.localstack.cloud/user-guide/state-management/cloud-pods/) are a mechanism to save LocalStack state onto a remote backend, allowing to restore infrastructure and state of various services when required.

{{< img-simple src="persistence-state-cloud-pods.png" alt="LocalStack Persistence vs Cloud Pods">}}

Using Cloud Pods, we were able to cut down the total infrastructure deployment time from a minute to less than 10 seconds, both locally and in our CI pipelines. To maintain an up-to-date version of the Cloud Pod, we have a GitHub action which creates a pod with the latest infrastructure, that’s triggered on each merge to the `main` branch. We then use that pod in combination with our [auto-loading Cloud Pods](https://docs.localstack.cloud/user-guide/state-management/cloud-pods/#environmental-variables) feature, which allows us to load cloud pods on the start-up of LocalStack automatically.

```yaml
AUTO_LOAD_POD=localstack-backend-pod
```

Now, when LocalStack starts up, our entire backend loads from the Cloud Pod, allowing us to perform various tests locally and in CI.

This setup is also extremely beneficial for our frontend engineers, as they don't need to start or install any backend dependencies on their machines. They simply use our up-to-date pod and LocalStack to develop the customer-facing UI.

### Continuous Integration (CI) Analytics

With the LocalStack v3 release, we released a private preview of our [CI Analytics](https://docs.localstack.cloud/user-guide/ci/ci-analytics/) offering. CI Analytics allow us to collect, analyze, and visualize critical metrics from our CI pipelines, helping us understand the execution of our test suites in CI builds and related outcomes.

{{< img-simple src="ci-analytics.png" alt="LocalStack CI Analytics">}}

This allowed us to get detailed insights and traceability across our CI pipeline run by:

-   Recording all interactions throughout a CI build to get a detailed timeline of API calls using Stack Insights.
-   Select and drill into the infrastructure & application state at a particular point of execution using Cloud Pods.
-   Correlate the timeline of API calls with state changes, to identify hot spots and defect root causes.

This can be enabled by just setting a simple configuration variable in your LocalStack GitHub Action (or any other CI provider in general):

```yaml
LS_CI_PROJECT=name-of-your-project
```

With CI Analytics we can drill down into the request & response traces for every AWS API call that we make with our integration test suite. With the help of CI Analytics, we have brought together the critical missing pieces of CI observability & analytics into one single feature which has massively improved our CI troubleshooting and debugging experience.

Additionally, we can now instrument the important paths and processes, capture the infrastructure state which can be restored locally, and enrich our API telemetry to capture relevant data that help our developers understand flaky CI tests. 

{{< img-simple src="ci-analytics-request-reponse-traces.png" alt="CI Analytics Page showing the request/response traces">}}

## How do we use LocalStack to enable application previews and E2E testing?

After running our integration tests, both locally and in CI, the next step was to deploy the CDK stack in our staging environment. The staging environment allowed us to run our end-to-end (E2E) integration test suite, which rely on the Playwright framework, and further use it for acceptance testing, to get alignment across cross-departmental projects. It allowed us to achieve the final degree of validation before we shipped a new release to production.

With the LocalStack v3 release, we released a private preview of [Ephemeral Instances](https://docs.localstack.cloud/user-guide/cloud-sandbox/ephemeral-instance/). These ephemeral instances allow us to run a short-lived encapsulated instance of LocalStack in the cloud. It allows us to run our E2E tests, preview features in our cloud application, and collaborate asynchronously within and across the team.

{{< img-simple src="ephemeral-instances.png" alt="LocalStack Ephemeral Instances">}}

With these ephemeral instances, we can now deploy our entire application (frontend, backend, and infrastructure) on an ephemeral instance, and expose the instance to us internally, which allows us to test our changes with every pull request. This has allowed us to replace our staging environment with ephemeral instances, which we use to continuously run automated tests and check out individual features in parallel manually.

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
        make deploy-local

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

{{< img-simple src="ephemeral-preview-pull-request-comment.png" alt="A PR comment displaying the Ephemeral Instance URL">}}
    
Though application previews have been ubiquitous in the frontend space, LocalStack can now spin up your entire application, including any infrastructure relying on AWS services, and ship you a live deployment with every change. This first-class preview-on-pull request support is instrumental in helping LocalStack become a true cloud development platform — not only for developers, but also for QA & management, to fully become the backbone of cloud development throughout the entire SDLC.

## Conclusion

That’s the long and short of how we are building LocalStack with LocalStack. LocalStack has enabled rapid design and development of sophisticated solutions by reducing the number of test and UAT environments while improving the quality and lead time. The best way we can improve our product is to iteratively adopt it, and ensure we can leverage the same features as our customers do and continue to nail down the developer experience. Over many months, we have continued to ship improvements to enable teams, like ours, to scale and mitigate common bottlenecks while developing on the cloud.

As we continue our work in fleshing out the LocalStack experience, we aim to further support enterprise compliance & insights, with features like Chaos engineering, Productivity metrics, Cost optimizations, and more. This will allow us to expand from our initial focus on the inner dev loop to an outer dev loop experience, to accelerate your cloud journey and to put developers back in charge. Building a cloud emulator is hard, and this sets us up towards solving larger problems at hand — state management, SDLC, collaboration, and more.

Stay tuned for more news and awesome features in the upcoming months — or if you would like to get access to some of the features we’ve been using, get in touch with us.
