---
title: AWS re:Invent - Ten Things You Need to Know About LocalStack
description: Discover LocalStack's versatility through our curated list of top 10 features, catering to both beginners and experts in the cloud journey, from core emulation to advanced analytics.
lead: Discover LocalStack's versatility through our curated list of top 10 features, catering to both beginners and experts in the cloud journey, from core emulation to advanced analytics.
date: 2023-11-27
lastmod: 2023-11-27
images: ['10-things-to-know.png']
contributors: ["Anca Ghenade"]
tags: ['showcase']
show_cta_1: true
leadimage: '10-things-to-know.png'
weight: 3
---


{{< img-simple src="10-things-to-know-banner.png" alt="ten things about localstack">}}


In an era dominated by cloud computing, developing and testing software locally before deploying it to the cloud for production is essential,
as it allows for thorough, controlled testing environments, quicker iteration, and reduced dependency on cloud resources during the development
phase, leading to more robust and reliable cloud-deployed applications.

Here at AWS re:Invent, a recurring question captures our attention: *"What is LocalStack?"* - A query that sparks conversations about innovation,
local development solutions, and the seamless transition to the cloud environment. So, to address the excitement and to celebrate our participation 
as an official sponsor of this event, we've decided to compile a comprehensive list highlighting the top 10 features of LocalStack that you should get excited about.

This blog post will provide insights that cater to everyone's needs - whether you're just starting out and are intrigued by the idea of a core cloud 
emulator, or you're a seasoned pro looking for more advanced functionalities like dashboards and stack analytics. This list promises 
to unveil aspects of LocalStack that are both foundational and fascinating, ensuring that no matter where you are in your cloud journey, 
there's something valuable for you.

## Navigating the Cloud Environment

{{< img-simple src="cloud-emulator-banner.png" alt="cloud emulator">}}

LocalStack provides a [**high fidelity**](/2022-08-04-parity-explained/) replication of the AWS cloud environment, enabling developers to emulate AWS services in a container 
on any machine. It's an invaluable tool for testing and development, offering realistic and detailed AWS service 
interactions. This makes the developing journey of cloud applications a breeze, allowing for cost-effective and efficient 
experimentation and integration before deployment to the actual AWS cloud. You can check out all the services in detail on our 
[**service coverage**](https://docs.localstack.cloud/references/coverage/) page.

## Rapid Testing Feedback Loops

{{< img-simple src="cloud-testing-banner.png" alt="cloud testing">}}

LocalStack serves as a powerful cloud testing tool, offering a sandboxed environment that replicates AWS services, allowing users to 
experiment with and test AWS functionalities without incurring costs or impacting live environments. By simulating AWS on a local machine,
developers can rigorously test their applications, ensuring compatibility and robustness before deploying to the AWS cloud. This approach
not only accelerates development cycles but also significantly reduces the complexity and expense associated with cloud-based testing,
streamlining the path from development to production. See how easy it is to [**get started**](https://docs.localstack.cloud/getting-started/).

## Streamlining DevOps Processes

{{< img-simple src="ci-banner.png" alt="continuous integration">}}

LocalStack is a pivotal tool for continuous integration DevOps workflows, allowing [**teams**](/2023-04-24-case-study-knowbe4/) to forego the complexities of AWS testing environments. It integrates
smoothly with existing continuous integration platforms, embedding AWS cloud emulation directly into CI pipelines. This integration 
facilitates robust testing and delivery of cloud-native applications by leveraging LocalStack's features like Cloud Pods and CI analytics.
Teams can confidently execute their test suites and integration checks, ensuring that applications are vetted thoroughly in a controlled 
setting before any code is deployed, aligning with best practices in modern software development.

It’s as easy as running any container in CI. You can find all the information in our [**CI documentation**](https://docs.localstack.cloud/user-guide/ci/).

## Test and Validate Infrastructure as Code

{{< img-simple src="iac-banner.png" alt="infrastructure as code">}}

LocalStack seamlessly integrates with Infrastructure as Code (IaC) tools such as Terraform, Pulumi, and AWS CDK, providing developers with 
a powerful environment to pre-test infrastructure provisioning configs. By mirroring AWS services locally, it allows users to validate their 
infrastructure code and ensure all components are configured correctly before deploying to the actual cloud. This early testing capability is 
crucial for catching issues upfront, reducing errors in production, and accelerating development cycles by enabling immediate feedback and 
iterative improvements, all within the confines of a developer's local environment. 
Find out more about all the [**integrations**](https://docs.localstack.cloud/user-guide/integrations/).

## Validate IAM Policies and Enhance Security

{{< img-simple src="iam-banner.png" alt="iam policies">}}

Security testing within LocalStack provides a robust mechanism to validate IAM policies and permissions, mirroring AWS's security enforcement. 
This enables a comprehensive examination of application security in an environment that accurately reflects the real AWS setup. Key features 
include the enforcement of IAM directives within the stack to verify security postures, the retrieval of IAM policy engine logs for in-depth 
analysis of policy evaluations, and the use of live policy streams to identify and rectify permission-related logical errors, thereby enhancing 
the overall security framework before deployment. Read the full story in our [**IAM security user guide**](https://docs.localstack.cloud/user-guide/security-testing/).

## Engineering for Robust Cloud Systems

{{< img-simple src="chaos-banner.png" alt="chaos engineering">}}

Chaos engineering with LocalStack enables teams to proactively enhance system resilience by categorically testing against controlled disruptions. 
For software developers, this means employing Fault Injection Simulator (FIS) experiments to test application behaviors and error responses. 
Architects leverage it to confirm system design robustness, notably through Route53 failover scenarios. Operations teams assess the dependability 
of infrastructure provisioning against outages and anomalies. By embedding these chaos experiments early in development, potential frailties are 
identified and mitigated, leading to robust systems capable of withstanding unpredictable conditions. These practices are outlined through 
practical examples in our [**user guides**](https://docs.localstack.cloud/user-guide/chaos-engineering/).

## Spin Up Cloud Sandboxes

{{< img-simple src="ephemeral-banner.png" alt="cloud sandbox">}}

LocalStack ephemeral environments offer a cloud-based sandbox, enabling developers to utilize LocalStack instances directly in the cloud. 
This approach streamlines the development and testing cycles and supports the creation of preview environments for each pull request, enhancing 
the review process for application changes. Additionally, it fosters collaborative development by providing a shared environment for testing new 
features, ensuring that all team members can work in sync and test in a uniform setting. 
You can read about it further in [**our documentation**](https://docs.localstack.cloud/user-guide/cloud-sandbox/).

## Save, Share, and Restore Cloud State with Cloud Pods

{{< img-simple src="cloud-pod-banner.png" alt="cloud pods">}}

Cloud Pods offer a dynamic way to manage the state of LocalStack instances, with the capability to save, version, share, and restore these states 
as persistent snapshots. This functionality is invaluable for collaborative debugging by allowing team members to share specific instance states. 
It streamlines continuous integration by pre-seeding environments, thereby automating the initialization of testing pipelines. Additionally, 
it facilitates the creation of consistent and reproducible local development and testing environments. Through the Cloud Pods CLI, alongside 
LocalStack’s remote storage backend, teams can manage and share their application’s state effortlessly, all while interfacing seamlessly via the 
LocalStack Web Application. Read more about Cloud Pods in our dedicated [**user guide**](https://docs.localstack.cloud/user-guide/cloud-pods/).

## Expand your local testing stack with Extensions

LocalStack Extensions offer developers the flexibility to enhance and personalize their LocalStack experience. These extensions provide a 
framework for launching custom services within the LocalStack container and utilizing the platform's robust ecosystem. They empower developers 
to introduce new services, augment existing ones, or incorporate unique functionalities. With the Extensions API, custom logic and services can 
be seamlessly integrated, enabling tasks like starting bespoke services, enriching AWS requests with extra data, or redirecting AWS API call logs 
to specialized backends. MailHog, Miniflare, AWS replicator, httpbin, Stripe, Diagnosis Viewer are only a few examples of what users can find in 
the Extension Library. [**Extend your LocalStack Experience**](https://docs.localstack.cloud/user-guide/extensions/).

{{< img-simple src="extensions-banner.png" alt="localstack extensions">}}

## Manage Resources and Observe Telemetry With the Web App

The LocalStack Web Application is an all-encompassing web interface designed to comprehensively manage LocalStack functionalities, including 
account services, Stack Insights, Cloud Pods, and CI analytics. It facilitates resource and configuration management, acts as a gateway to the 
Pro CLI, and streamlines team collaboration by managing roles and permissions. Users can monitor AWS API usage with Stack Insights, navigate 
local AWS resources via the Resource Browser, and manage Cloud Pods through a dedicated browser. Additionally, the Extensions Library broadens 
LocalStack's capabilities, while features for state export/import and custom endpoints enhance user experience. LocalStack also supports 
Single-Sign On configurations, providing a seamless and integrated environment for enterprise needs. Learn more about our [**Web Application**](https://docs.localstack.cloud/user-guide/web-application/).

{{< img-simple src="webapp-banner.png" alt="localstack webapp">}}

## Conclusion

LocalStack presents a comprehensive suite of tools and features designed to revolutionize the way teams approach cloud-based development and 
testing. From the robustness of security testing, the resilience offered by chaos engineering, to the convenience of cloud-based ephemeral 
environments and dynamic Cloud Pods, LocalStack is no longer just a cloud emulator where you test your applications and infrastructure as code.
You can now embrace the full potential of LocalStack and transform your cloud development into a more efficient, reliable, and collaborative process.

To wrap up, don't forget to visit us at **Booth 1074** on the AWS re:Invent expo floor. We'd love to meet you, share in the excitement, and show you the engaging
activities we have in store. Come by, say hi, and experience the buzz of innovation with us!
