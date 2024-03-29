---
title: Neurolytics enhances the efficiency and agility of its cloud-based development with LocalStack
description: Neurolytics leverages the full spectrum of LocalStack's capabilities to revolutionize its AI-driven behavioral video analysis platform. In this case study, we explore the journey of Neurolytics with Yashar Hosseinpour, delving into how they utilize LocalStack for seamless local development and testing, which resulted in substantial cost savings, showcasing the transformative impact of LocalStack on their local infrastructure development and overall engineering productivity.
lead: Neurolytics leverages the full spectrum of LocalStack's capabilities to revolutionize its AI-driven behavioral video analysis platform. In this case study, we explore the journey of Neurolytics with Yashar Hosseinpour, delving into how they utilize LocalStack for seamless local development and testing, which resulted in substantial cost savings, showcasing the transformative impact of LocalStack on their local infrastructure development and overall engineering productivity.
date: 2024-01-25
lastmod: 2024-01-25
contributors: ["LocalStack Team"]
logo: 'neurolytics-logo.png'
leadimage: 'localstack-xiatech.png'
tags: ["case-study"]
layout: case-study
properties:
  - key: Name
    value: Neurolytics.ai
  - key: Description
    value: Neurolytics offers a SaaS solution that uses AI and video technology to provide objective behavioral analytics for effective candidate selection, helping businesses find employees who best fit their team and company culture.
  - key: Location
    value: Utrecht, Netherlands
  - key: Industry
    value: Recruiting Software
  - key: AWS Services
    value:
      - DynamoDB
      - Elastic Container Registry (ECR)
      - S3
      - Lambda
      - Elastic Container Service (ECS)
      - CloudWatch Logs
      - EventBridge
  - key: Integrations
    value:
      - AWS CLI
      - AWS SDK
      - Docker SDK
---

<div class="quote-container mt-4">

  > _“LocalStack has made cloud development easier to test and debug, significantly reducing our development and testing loops by approximately 30%. This enables faster iterations and quicker feature releases and translates to substantial financial savings.”_
  <div class="quote-author">
    <p><a href="https://www.linkedin.com/in/yashar-hosseinpour/">Yashar Hosseinpour</a>,</p>
    <p>Lead Developer at <a href="https://neurolytics.ai/">Neurolytics</a></p>
  </div>
</div>

<div class="lead-content">
  <p>Neurolytics, a startup in the field of behavioral analysis, has developed a platform for enhancing the recruitment process through AI-driven video analysis. Their solution offers a transformative approach for companies and job candidates, ensuring a match between their profiles and the company culture. This is crucial for hiring individuals who are not only qualified but are also likely to thrive and stay long-term within the organization.</p>

  <p>Hosted on Amazon Web Services (AWS), Neurolytics' infrastructure leverages the robustness and scalability of the platform. However, the challenge of time- and cost-efficient development in such a dynamic environment led them to LocalStack. This move has enabled the Neurolytics team to significantly streamline their development and testing cycle, thereby enhancing their agility and innovation capabilities.</p>

  <p>In this case study, we talk with Yashar Hosseinpour, who offers some detailed insights into how LocalStack has streamlined Neurolytics' dev&test loops. This includes a reduction in development time, cost savings, and the overall impact on their engineering processes. We explore the challenges they faced, the solutions they implemented, and the benefits they've reaped from integrating LocalStack into their AWS-based development workflows.</p>
</div>

## Challenge

Neurolytics faced considerable challenges in its development cycle due to the complexities of integrating with various AWS services. Their platform required the orchestration of multiple AWS services, which turned out to be a significant hurdle, as each update or feature development necessitated direct deployment and testing on AWS, which led to lengthy and inefficient development cycles. Moreover, the need to maintain individual AWS accounts for each developer added a layer of operational cost and complexity.

The process of real-time testing and debugging in the AWS environment posed another significant challenge: The inability to perform quick local tests for immediate feedback considerably slowed down the pace of development. Consequently, the development team found themselves in a constant loop of building, deploying, and waiting, which was not only time-consuming but also detracted them from an agile development process.

In response to these challenges, Neurolytics began exploring solutions that could replicate the AWS environment locally. The aim was to establish a rapid, efficient feedback loop, enabling developers to test and debug in a local setup that closely mirrored the cloud environment. It was in this context that LocalStack emerged as a potential solution, promising to transform its cloud development and testing strategy.

## Solution

In response to the challenges faced in their development process, Neurolytics turned to LocalStack. This shift was geared towards achieving faster development iterations, reducing operational costs, and enhancing the overall efficiency of their development process. The integration of LocalStack into Neurolytics’ development toolstack marked a turning point. It allowed the team to run multiple cloud services locally, maintaining parity with the real AWS cloud.

<img src="neurolytics-localstack-aws-overview.png" alt="Overview of how Neurolytics use LocalStack on developer machines and the real AWS cloud for production">

This shift not only streamlined their development cycle but also offered an environment for quicker debugging and feature validation. The benefits extended beyond just technical improvements — financially, it translated into considerable cost savings by eliminating the need for individual AWS accounts for each developer, and operationally, it simplified the management of their cloud resources.

<div class="quote-container mt-4">

  > _“By eliminating the need to deploy main critical services to individual AWS accounts for each developer, we've saved around 20x per month per developer. With LocalStack, we've successfully streamlined our development process, making it a game-changer for our small team working on multiple cloud services.”_
  <div class="quote-author">
    <p><a href="https://www.linkedin.com/in/yashar-hosseinpour/">Yashar Hosseinpour</a>,</p>
    <p>Lead Developer at <a href="https://neurolytics.ai/">Neurolytics</a></p>
  </div>
</div>

Moreover, Neurolytics developed a custom Command Line Interface (CLI) tool in Golang, leveraging AWS and Docker SDK, to manage their local infrastructure more effectively. This tool enabled them to start up their LocalStack environment with a single command and manage each service individually, further enhancing their processes and developer experience (DevX).

The CLI also included features for backing up and exporting the state of services, as well as importing functionality, allowing for easy sharing of the local environment state among team members. This level of control and flexibility was instrumental in achieving a more efficient and collaborative development environment, powered by LocalStack.

## Results

<div class="img-group d-block d-sm-flex align-items-start">
  <img src="30-percent-reduction-development-testing-time.png" alt="Accelerating Resource Creation by 70% Compared to AWS" class="img-1">
  <img src="cost-saving-operational-efficiency.png" alt="Streamlined Testing and Enhanced IaC Code Validation with LocalStack" class="img-2">
</div>

### 30% Reduction in Development and Testing Time

Integrating LocalStack into Neurolytics' development workflow brought about a remarkable reduction in their development and testing cycles. Previously, the process of deploying and testing on AWS significantly extended their development time. With LocalStack, this time was reduced by approximately 30%, allowing for faster iterations and feature releases. This development efficiency accelerated the pace of innovation and enabled the team to respond more rapidly to the evolving needs of their platform.

### Cost Savings and Operational Efficiency

By eliminating the need to deploy critical services to individual AWS accounts for each developer, Neurolytics saved around 20x the cost of a LocalStack license per month per developer. This reduction in operational costs was a significant financial benefit, particularly for a startup managing tight budgets. Moreover, the ease of managing cloud resources locally through their custom CLI tool added to the overall operational efficiency, streamlining the management of their complex cloud infrastructure locally.
