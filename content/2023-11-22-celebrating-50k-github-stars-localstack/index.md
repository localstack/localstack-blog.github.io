---
title: Celebrating 50,000 GitHub Stars for LocalStack!
description: We're excited to announce that LocalStack has hit a significant milestone of achieving 50,000 GitHub stars. It has been an incredible journey since we started out to build the best possible cloud developer experience, and we are thrilled to share this news with you!
lead: We're excited to announce that LocalStack has hit a significant milestone of achieving 50,000 GitHub stars. It has been an incredible journey since we started out to build the best possible cloud developer experience, and we are thrilled to share this news with you!
date: 2023-11-22
lastmod: 2023-11-22
images: []
contributors: []
tags: ['news']
contributors: ["Waldemar Hummer", "Harsh Mishra"]
---

Seven years ago, on August 16, 2016, the LocalStack project was born. The journey since then has been nothing short of incredible. LocalStack has now crossed 50,000 stars on GitHub after three major releases and dozens of minor releases. For the past 16 months since [**LocalStack 1.0**](https://blog.localstack.cloud/2022-07-13-announcing-localstack-v1-general-availability/) went live, it has been on a consistent upward trajectory, and we are proud of everyone who is discovering, starring, and joining our project & community!

Our long-term goal is to not only create the best platform and tools for local cloud development but also to influence the industry's perspective, prioritizing developer experience (**DevX**) in all cloud-based and managed service offerings. With the groundbreaking and pioneering work that the LocalStack team and our community are driving, we have demonstrated that it is indeed possible to fundamentally re-think the way how modern cloud applications are developed.

Cloud providers like Cloudflare have already embarked on a *local-first API development journey*, and we expect more to follow suit. LocalStack aims to provide a comprehensive developer platform that facilitates local multi-cloud development across different providers and services - metaphorically becoming the **central app store** for local cloud developer tools.

{{< img-simple src="localstack-50k-stars.png" width=300 alt="A screenshot of the LocalStack GitHub repository, indicating 50k stars and details of recent activity such as commits and pull requests">}}

We'll use this milestone to look back at LocalStack's growth, some important milestones, and what our community users can expect next!
 
## How did we get here?

LocalStack's journey to achieving 50,000 GitHub stars is a story of community-driven growth and engagement. The project's inception occurred at Atlassian with the initial commit ([`44326584`](https://github.com/localstack/localstack/commit/44326584#diff-b335630551682c19a781afebcf4d07bf978fb1f8ac04c6bf87428ed5106870f5)), introducing an *easy-to-use mocking framework* tailored for cloud application development, especially for AWS Cloud. Starting with eight AWS services, including Lambda, API Gateway, and DynamoDB, LocalStack was developed to replicate AWS APIs locally, enabling offline development. The initial idea was to enable developers to work on their applications offline, even in circumstances such as commuting on a train or flight.

LocalStack, originally a side project, first garnered public attention with a post on [Hacker News](https://news.ycombinator.com/item?id=13966088) (also archived on the [Internet Archive](https://web.archive.org/web/20231122065745/https://news.ycombinator.com/item?id=13966088)). This post introduced the concept of a fully local mocking and testing framework for cloud applications, drawing the interest of the broader tech community, including notable AWS figures. A key moment in LocalStack's journey was when Jeff Barr, Chief Evangelist at AWS, [shared a tweet about LocalStack](https://twitter.com/jeffbarr/status/846382903210663936) (also archived on the [Internet Archive]()), leading to an overnight surge in GitHub stars, with thousands joining our community.

Here is a graphic that captures the watershed moment and the subsequent growth of LocalStack as a project:

{{< img-simple src="localstack-growth-graph.png" width=300 alt="An infographic highlighting key milestones of LocalStack, including GitHub stars and Docker pulls growth, along with version releases and team expansion from August 2016 to November 2023">}}

## Engaging with the community

The community's response was overwhelmingly positive — developers began actively [contributing to LocalStack](https://github.com/localstack/localstack/graphs/contributors), [creating issues](https://github.com/localstack/localstack/issues), [providing suggestions](https://discuss.localstack.cloud/), and [submitting pull requests](https://github.com/localstack/localstack/pulls). This engagement wasn't just from individual developers — entire teams from various companies, specifically those focused on developer experience, started using LocalStack to create efficient development environments. 

Going forward, we will continue to focus on strengthening the LocalStack ecosystem and integrations with frameworks such as *[Testcontainers](https://testcontainers.com/), [CloudFormation](https://docs.localstack.cloud/user-guide/aws/cloudformation/), [CDK](https://docs.localstack.cloud/user-guide/integrations/cdk-for-terraform/)* and more! With the new [extension mechanism](https://docs.localstack.cloud/user-guide/extensions/), you can now extend and customize LocalStack to your needs. We have shipped official extensions for [Miniflare](https://miniflare.dev/), [Mailhog](https://github.com/mailhog/MailHog), [Stripe](https://github.com/adrienverge/localstripe), [httpbin](https://httpbin.org/), and more to provide you with a smooth local development experience.

After years of active development, we have achieved what we set out to do — building **faster development cycles** with instant feedback loops, providing **better isolation** of application logic in test environments, and setting up **enhanced security** with no dependency on a remote cloud service for execution. As always, we are grounding our work on user insights and feedback, and we are excited to have such a committed community!

## What's next?

The best is yet to come! We are primarily working on making our users happy with new features and improving LocalStack as much as we can do. With the [3.0 release](https://blog.localstack.cloud/2023-11-16-announcing-localstack-30-general-availability/), LocalStack has evolved from being an open-source project to an enterprise development framework! We have also received a lot of feedback from our individual users, and we've heard you loud and clear — We're super excited to bring our **Hobby Plan** for individuals *who are not using LocalStack commercially*!

We want to **lower the barrier of entry** for everybody and make it **easier to get started**. With the Hobby Plan, we aim to enable more users to use the LocalStack emulator with all its otherwise **paid features**. LocalStack's Hobby Plan allows you to get started with:

- [Advanced AWS services](https://docs.localstack.cloud/user-guide/aws/feature-coverage/)
- [CI Starter Tier](https://docs.localstack.cloud/user-guide/ci/ci-keys/)
- [IAM Policy Enforcement](https://docs.localstack.cloud/user-guide/security-testing/iam-enforcement/)
- [State Persistence](https://docs.localstack.cloud/references/persistence-mechanism/)

LocalStack is deeply rooted in open-source, and by making LocalStack available for free for non-commercial projects, we hope to **further support and give back to the open-source community**. It is the easiest way to get into cloud development and to learn not only the specific AWS services but also general cloud concepts that are also applicable to all the other cloud platforms out there. Using the Hobby Plan does not require a credit card. There is no need to set up an AWS account, and there is also no cost risk (or use up the **AWS Free Tier**).

*Get started with LocalStack Hobby Plan today! [Sign up here](https://app.localstack.cloud/pricing/).*

## Conclusion

To put it in a nutshell, we are incredibly thankful to our community for all the support and feedback we have received over the years! Your contributions have made LocalStack a battle-tested cloud emulation platform. They helped us evolve from a project that leveraged existing tools to a more integrated platform that is capable of emulating complex interactions between various AWS services.

Our mission remains the same: Give developers back control over their environments and free them from wasting time on inefficient development & testing loops in the cloud, so they can focus on creating great products! We are excited about making LocalStack more configurable and adaptable to specific user needs, and we are looking forward to your continued support and feedback!

*Onto hitting the 100,000 GitHub Stars milestone next!* 🚀
