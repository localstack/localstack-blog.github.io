---
title: Mastering AWS Infrastructure Testing with LocalStack Cloud Pods - on localhost, CI and Testcontainers
description: Discover how LocalStack Cloud Pods enable efficient testing of your applications by providing snapshots of AWS infrastructure for local development, continuous integration, and Testcontainers integration.
lead: Discover how LocalStack Cloud Pods enable efficient testing of your applications by providing snapshots of AWS infrastructure for local development, continuous integration, and Testcontainers integration.
date: 2024-02-20
lastmod: 2024-02-20
images: ['cloud-pods-2024.png']
leadimage: 'cloud-pods-2024.png'
contributors: ["Anca Ghenade"]
tags: ['showcase']
weight: 3
---

{{< img-simple src="cloud-pods-2024.png" width=300 alt="A LocalStack Cloud Pods banner">}}

## Cloud Pods, State & Persistence in the LocalStack ecosystem

In a few words, LocalStack is a cloud service emulator that allows you to develop and test your cloud and serverless applications
locally by replicating AWS environments on any machine without the need to deploy them to the actual AWS cloud. This tool 
provides an easy and cost-effective way to test cloud applications by locally providing all the necessary dependencies.

LocalStack ships as a Docker container, making its state ephemeral by default. Once the container shuts down, all the 
services and data are gone. So, in an attempt to solve this issue, the LocalStack engineers came up with some solutions 
that could be applied on a case-by-case basis. These are Persistence, State, and Cloud Pods. To better understand the 
differences between these approaches, we can simplify them in a diagram:

{{< img-simple src="persistence-pods-remote.png" width=300 alt="The difference between persistence, local state and Cloud Pods.">}}

Anything that is inside a LocalStack container constitutes the "state.” The "state" can either be persisted on your local
machine and be loaded at startup, exported anytime as a single local file (state export), or stored on the LocalStack platform (cloud pods). 
The underlying mechanisms of all three solutions are similar, but the layout obviously differs. We can observe how using state 
locally and Cloud Pods are very similar, one might even consider them `local` and `remote` Pods. That being said, in this 
article, we will focus on Cloud Pods and what problems they solve while glancing at the *local state* from time to time.

Simply put, a Cloud Pod is a persistent state snapshot of your LocalStack instance. Think of it as a snapshot of a database 
from which you can restore the tables and all associated data.

Cloud Pods are not very rigid, though. They also enable users to selectively capture the configuration and data of only 
specific services, making them highly adaptable and tailored to diverse development and testing needs.

## Cloud Pods: How do they work?

There are several reasons why the concept of Cloud Pods plays a crucial role in the LocalStack ecosystem.

{{< img-simple src="cloud-pods-tf.png" width=300 alt="A diagram depicting ways to take of snapshot of the LocalStack instance.">}}


Focusing solely on Cloud Pods and state, this diagram illustrates the relationship and workflows between the LocalStack instance,
local state, Cloud Pods, and the LocalStack Platform:

1. The LocalStack Instance runs on a host machine, and the required infrastructure is created via a Terraform configuration
file. The AWS resources interact with each other, meaning that more complex cloud scenarios can be tested.

The LocalStack Instance can locally export its state by using the `state export` command, and at the same time, it can 
inject a state using `state import.`

2. The LocalStack Platform (on the right) stores Cloud Pod artifacts by default when using the `pod save` command. When
pushing an already existing pod, a new version is created and subsequently uploaded to the platform. Complementary, the
`pod load` command is used to inject a Cloud Pod.

The user can emulate AWS cloud services on their local machine with LocalStack, save the configurations into versioned 
Cloud Pods, and additionally share or store these configurations on the LocalStack Platform. It's a way to manage development
and testing environments with reproducibility and version control. 
The [state management](https://docs.localstack.cloud/user-guide/state-management/pods-cli/) section of the documentation 
offers a comprehensive explanation of all the CLI commands.

## Why use Cloud Pods

### Sharing and Collaboration

LocalStack's snapshot feature significantly enhances team collaboration on cloud applications by enabling the capture 
and sharing of AWS services' precise configurations and data. This ensures consistent development environments across 
teams, resolves discrepancies, and facilitates efficient onboarding and troubleshooting. By simplifying the synchronization 
of workspaces through LocalStack's remote platform, Cloud Pods accelerates development cycles and ensures faster and more 
dependable tests.

### Modularity and Flexibility

LocalStack's Cloud Pods offer enhanced modularity and flexibility, enabling teams to configure distinct, purpose-specific 
testing environments. For instance, one Cloud Pod can replicate a production database for final-stage testing, while another 
can host a database filled with edge cases for validation testing. This nuanced approach not only allows targeted testing of 
application components under changing conditions but also strengthens test integrity by preventing data overlap and ensuring 
isolation.

Another practical application is providing layering of different datasets on the same infrastructure, which is crucial for machine learning applications to create reproducible samples. This method ensures that training datasets and the corresponding code logic for ML models are packaged together, enhancing reproducibility and streamlining development workflows in an engineering-focused environment.

### Efficiency and Speed

Cloud Pods boost efficiency in cloud application development by cutting down environment setup times. Compared to 
traditional infrastructure provisioning, like a full Terraform stack setup, LocalStack's injection of pre-configured 
Cloud Pods reduces setup time from minutes to mere seconds. This rapid deployment not only accelerates the development 
cycle but also allows developers to focus more on actual work and less on waiting, aligning perfectly with the demands of 
agile methodologies and fast-paced project timelines.

## "Talk is cheap. Show me the code.”

We’ve mentioned some of the strong points of Cloud Pods. Let’s now see them deliver on those promises. You can follow 
along with all the examples by cloning the [**demo repository**](https://github.com/localstack-samples/localstack-cloud-pods-blogpost-demo).

### Accelerating development and testing

The following recording shows the comparison in time spent waiting for the same infrastructure to be set up, first using 
Terraform and then via Cloud Pods:

{{< youtube aDis2kOtqtw >}}
<br>

The results speak for themselves: 1 minute 32 seconds vs 15 seconds. If your stack is generally stable and is not actively
being built, you can easily focus on your application logic and use Cloud Pods rather than provision your infrastructure 
using IaC tools every time. Alternatively, you can also export the state of your LocalStack instance or create a Cloud Pod
using only the desired services with the `--services` flag.

Now, we can kick it up a notch and introduce the same speed in CI. This example follows three workflows described in the
same [**repository**](https://github.com/localstack-samples/localstack-cloud-pods-blogpost-demo). All the runs are doing the same thing: 
setting up the environment, starting LocalStack, spinning up the desired resources, and, as a final check, verifying 
if the buckets with the correct name exist.

We can see that both workflows using Pods have a shorter running time, whether they’re using local files or remotely stored ones.
This is also because you can skip some more steps in the workflows, such as installing Java and Maven and building the Lambda jars.
For a one-time run, the differences are 1 minute and 42 seconds and 1 minute 40 seconds, but let’s remember that this stack is
only using a few services: IAM, STS, SQS, SNS, DynamoDB, S3, Lambda, KMS, Kinesis, and CloudWatch. All these minutes add up
with every workflow you run, and in this scenario, the run time went down by 42,8% and 42%, respectively. Meanwhile, the
developer running this on their machine has reduced the waiting time by 84%.

{{< img-simple src="ci-workflows.png" width=300 alt="Three GitHub action workflows using Cloud Pods.">}}

### Optimizing collaboration and enhancing teamwork

In this instance, we'll see how Cloud Pods improve team collaboration by optimizing AWS infrastructure updates. From a bird’s eye
view, the process looks like this:

{{< img-simple src="localstack-in-ci-create-pod.png" width=300 alt="Creating Cloud Pods on changes to the IaC file.">}}

We utilize a GitHub Action to trigger a workflow whenever the IaC configuration files are 
modified (`.github/workflows/create-cloud-pod-on-infra-change.yml`). This action creates a new Cloud Pod and uploads it to 
the LocalStack Platform, providing a centralized access point for the team. Consequently, our application's build and test 
workflows can incorporate the newest Pod (`terraform-shipment-pod`), ensuring that everyone consistently works with the latest 
infrastructure changes. This method ensures that each team member is aligned with the most recent developments, promoting 
efficiency and coherence in engineering processes.

After each run, the updated Cloud Pod version is visible in the web application dashboard.

{{< img-simple src="localstack-pods-dashboard.png" width=300 alt="LocalStack Cloud Pod dashboard">}}

Upon closer inspection, the user can discover more information about each version of the Cloud Pod and, if needed, can choose from any older rendition to recover the LocalStack state. Be careful though, there is an auto-cleanup setting that will make sure old versions of Cloud Pods are cleaned up automatically. All versions of a Cloud Pod that exceed a threshold set by the users will be deleted. This can be 5, or it can be 20, depending on your needs.

{{< img-simple src="cloud-pod-details.png" width=300 alt="Cloud Pod detail in web app">}}

The integration of Cloud Pods into your CI pipelines significantly streamlines the process. Once new code is pushed and the workflow starts, LocalStack retrieves a Cloud Pod and pre-seeds its state in a matter of seconds, enabling integration tests to start as fast as possible. This swift setup significantly accelerates CI runs, ensuring rapid feedback and a more efficient development cycle while also enhancing the overall robustness and reliability of the development pipeline.

{{< img-simple src="localstack-remote-platform-ci.png" width=300 alt="Diagram showing Cloud Pods pulled from the LocalStack Platform for CI use">}}

### Integration with Testcontainers

Integrating LocalStack Cloud Pods with Testcontainers will improve and accelerate your testing process, particularly addressing
the prevalent pain point where Infrastructure as Code (IaC) tools like Terraform or Pulumi face challenges running against 
LocalStack during tests. Traditionally, this limitation nudged developers towards the more cumbersome and time-intensive route of
utilizing SDK clients to create resources manually. However, the synergy between Cloud Pods and Testcontainers' features 
elegantly circumvents this bottleneck. By enabling developers to swiftly inject predefined state into the LocalStack container,
these tools collectively foster a more efficient and seamless integration. This capability not only saves precious time but
also allows developers to bypass the initial setup hurdles and leap directly into the crux of actual testing. Consequently,
this integration harmonizes the flow between local and cloud environments, ensuring a smoother and more productive development
lifecycle.

In our sample application, we have covered two scenarios of provisioning our infrastructure for testing: using Cloud Pods
stored on the LocalStack Platform and using the locally stored state file. You can view the full test suites in the same
[**repository**](https://github.com/localstack-samples/localstack-cloud-pods-blogpost-demo), under the `src/test` folder.

#### Cloud Pods

Setting up the testing environment for running application tests against LocalStack is easy and straightforward:

```java
@Container
    protected static LocalStackContainer localStack =
            new LocalStackContainer(DockerImageName.parse("localstack/localstack-pro:latest"))
                    .withEnv("LOCALSTACK_AUTH_TOKEN", System.getenv("LOCALSTACK_AUTH_TOKEN"))
                    .withEnv("LOCALSTACK_HOST", "localhost.localstack.cloud")
                    .withEnv("LAMBDA_RUNTIME_ENVIRONMENT_TIMEOUT", "60")
                    .withEnv("AUTO_LOAD_POD", "terraform-shipment-pod");
```

- The ***@Container*** annotation is used in conjunction with the Testcontainers annotation (at the top of the class) to mark containers that should be managed by the Testcontainers framework.
- A new instance of a `LocalStackContainer` using the LocalStack Docker image tagged `localstack-pro:latest` is created.
- `LOCALSTACK_AUTH_TOKEN` environment variable refers to your own token for the Pro license.
- `LOCALSTACK_HOST` is the name of the host that exposes the services externally. This host is used, for example, when returning queue URLs from the SQS service to the client.
- `LAMBDA_RUNTIME_ENVIRONMENT_TIMEOUT` sets the timeout for the Lambda runtime environment to 60 seconds.
- Most importantly, to get the ball rolling,  `AUTO_LOAD_POD` will tell LocalStack which Cloud Pod to fetch and inject from the remote platform.

Before testing can start, we still need to remember that setting up the necessary resources takes time, even if it’s just a few seconds. 
That’s why we need to make sure that they exist before we use them. In this instance, we picked the Lambda function as a checkpoint,
but depending on your own resources and workflows tested, this can be adapted:

```java
@BeforeAll
    static void waitForLambdaToBeReady() {

        LambdaClient lambdaClient = LambdaClient.builder()
                .region(Region.of(localStack.getRegion()))
                .endpointOverride(localStack.getEndpointOverride(LocalStackContainer.Service.LAMBDA))
                .credentialsProvider(StaticCredentialsProvider.create(
                        AwsBasicCredentials.create(localStack.getAccessKey(), localStack.getSecretKey())))
                .build();

        WaiterOverrideConfiguration overrideConfig = WaiterOverrideConfiguration.builder()
                .maxAttempts(20)
                .build();

        LambdaWaiter waiter = LambdaWaiter.builder()
                .client(lambdaClient)
                .overrideConfiguration(overrideConfig)
                .build();

        GetFunctionRequest getFunctionRequest = GetFunctionRequest.builder()
                .functionName("shipment-picture-lambda-validator").build();

        WaiterResponse<GetFunctionResponse> waiterResponse = waiter.waitUntilFunctionExists(getFunctionRequest);
        waiterResponse.matched().response().ifPresent(response -> LOGGER.info(response.toString()));

    }
```

To summarize, this code snippet configures a Lambda client and uses a waiter to check and wait until the specified Lambda 
function (`shipment-picture-lambda-validator`) is ready before proceeding with the tests. This ensures that the tests only 
run when the necessary AWS resources are available in the LocalStack environment.

After everything is in place, the Spring Boot application will start up, and the tests can make sure the business logic is behaving
as expected.

{{< img-simple src="testcontainers-workflow-tests.png" width=300 alt="Workflow tests using Testcontainers">}}

### Local state

Similarly to using Cloud Pods, we can harness a similarly efficient strategy with locally exported state files. These files are 
integrated into the Docker container as mounted volumes, residing at a designated path (`etc/localstack/init-pods.d`). Just like
with initialization hooks, this setup ensures that the container scans this specific folder during startup, identifying and 
utilizing relevant files, thereby optimizing infrastructure initialization with ease. The setup will look the same, with the 
exception of one field, where we provide the path to our file:

```java
@Container
    protected static LocalStackContainer localStack =
            new LocalStackContainer(DockerImageName.parse("localstack/localstack-pro:latest"))
                    .withEnv("LOCALSTACK_AUTH_TOKEN", System.getenv("LOCALSTACK_AUTH_TOKEN"))
                    .withEnv("LOCALSTACK_HOST", "localhost.localstack.cloud")
                    .withEnv("LAMBDA_RUNTIME_ENVIRONMENT_TIMEOUT", "60")
                    .withFileSystemBind("src/test/resources/init-pods.d",
                            "/etc/localstack/init-pods.d");
```

The exported state resides in the `src/test/resources/init-pods.d` folder, which binds to the `/etc/localstack/init-pods.d` 
path in the container.

Integrating LocalStack’s Cloud Pods with Testcontainers allows for efficient infrastructure setup and data preparation across
various scenarios. It provides a practical approach to layering different datasets on the same infrastructure, which is crucial for testing as many scenarios as needed.

## Additional Details

Cloud Pods are typically stored remotely, with the default storage location being the LocalStack platform. Nonetheless, for 
organizations with specific data regulations or sovereignty concerns that preclude the use of remote storage infrastructures,
LocalStack offers the flexibility to store Cloud Pods on-premises, ensuring full control over storage. In terms of alternative
storage options, LocalStack supports two main types: S3 bucket remote storage and ORAS (OCI Registry as Storage) remote storage.
Additionally, LocalStack facilitates the management of these storage options through a command-line interface (CLI), providing
functionalities to create, delete, and list these remote storage locations for Cloud Pods. You can find all the necessary tools
for that in the [**documentation**](https://docs.localstack.cloud/user-guide/state-management/cloud-pods/#remotes).

## Limitations

Cloud Pods are a powerful feature, though they are optimized to work with certain configurations. Primarily, matching the major
version of LocalStack with that of the Cloud Pod ensures compatibility and smooth functionality. Additionally, while most services
are fully supported, there are instances where certain service information may not be completely serialized into a Cloud Pod. 
These nuances are minor and generally manageable within the framework and are also constantly [**documented**](https://docs.localstack.cloud/user-guide/state-management/persistence/#known-limitations) and updated.

## Conclusions

Cloud Pods are fundamentally transforming the landscape of testing for AWS-powered applications, by offering a scalable 
and efficient way to replicate AWS infrastructure locally and in CI, whether it's for fast development or integrating with 
popular frameworks. This breakthrough, coupled with remote storage for Cloud Pods, significantly enhances team collaboration,
ensuring seamless and consistent testing experiences across the board.