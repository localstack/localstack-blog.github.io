---
title: Case Study — LocalStack & CartonCloud
description: CartonCloud maximizes engineering efficiency and agility by leveraging LocalStack to emulate AWS services in a localized environment. In this case study with CartonCloud’s CEO, Vincent Fletcher, we showcase how CartonCloud achieved a 10x reduction in onboarding time and a remarkable 15x improvement in cost and operational efficiency. By standardizing LocalStack across their development environment, CartonCloud not only expedited their engineering endeavours but also significantly enhanced the overall developer experience.
lead: CartonCloud maximizes engineering efficiency and agility by leveraging LocalStack to emulate AWS services in a localized environment. In this case study with CartonCloud’s CEO, Vincent Fletcher, we showcase how CartonCloud achieved a 10x reduction in onboarding time and a remarkable 15x improvement in cost and operational efficiency. By standardizing LocalStack across their development environment, CartonCloud not only expedited their engineering endeavours but also significantly enhanced the overall developer experience.
date: 2024-02-26T7:46:04+05:30
lastmod: 2024-02-26T7:46:04+05:30
images: []
contributors: []
tags: ['case-study']
contributors: ["LocalStack Team"]
logo: ''
leadimage: ''
layout: case-study
properties:
  - key: Name
    value: CartonCloud
  - key: Description
    value: CartonCloud is a 3PL (TMS) transport management system and (WMS) warehouse management system with automation technology to transform the logistics industry to become more efficient and move to paperless systems to reduce administration tasks.
  - key: Location
    value: Burleigh Heads, Queensland
  - key: Industry
    value: IT Services and IT Consulting
  - key: AWS Services
    value:
      - DynamoDB
      - Elastic Container Registry (ECR)
      - S3
      - Lambda
      - Elastic Container Service (ECS)
      - AppSync
      - ElastiCache
      - Relational Database Service (RDS)
  - key: Integrations
    value:
      - AWS CLI
      - CloudFormation
---

<div class="quote-container mt-4">

  > _“WIP”_
  <div class="quote-author">
    <p><a href="https://www.linkedin.com/in/vincent-fletcher/">Vincent Fletcher</a>,</p>
    <p>Co-Founder & Chief Product Office at <a href="https://www.cartoncloud.com ">CartonCloud</a></p>
  </div>
</div>

<div class="lead-content">
  <p>CartonCloud is a cloud-based Transport Management System (TMS) and Warehouse Management System (WMS) designed specifically for the SME market, streamlining end-to-end business logistics operations. With a client base extending across Australia, New Zealand, and North America, CartonCloud's central operating system plays a crucial role in facilitating communication between administration staff, drivers, and warehouse personnel on a day-to-day basis.</p>

  <p>Initially developed as a standalone PHP application and hosted on Amazon Web Services (AWS), CartonCloud underwent a transformative journey as various functionalities transitioned into microservices and serverless architectures. However, this evolution introduced complexities in the staging, QA, and production infrastructures, particularly with incorporating microservices architecture and AWS services. This friction in the development process resulted in challenges for the developer experience, onboarding new engineers, and reliably testing cloud-native architectures.</p>

  <p>In this case study, we talk with Vincent Fletcher, about how LocalStack has significantly alleviated the engineering challenges they encountered during the development and testing of their applications and the specific hurdles faced by the CartonCloud engineering team. He further outlines how LocalStack has played a pivotal role in enhancing the onboarding experience for new engineers, enabling them to run the entire development environment locally.</p>
</div>

## Challenge

The initial challenge CartonCloud faced was replicating production behaviour while developing applications locally. The interplay between critical services like SQS & SNS was limited to QA environments, making remote debugging time-consuming and complex. On average, 15% of engineering hours were spent on finding workarounds or addressing local environment limitations.

Before LocalStack, the recommended solution involved spinning up individual AWS developer environments. However, this approach proved expensive, estimating a cost as high as $500 per developer, which included services such as individual VPCs and API Gateways, which were sparingly used. In an attempt to enhance the developer experience, a hybrid local and cloud development environment was implemented. Unfortunately, this solution proved unsatisfactory due to middleware issues and inherent complexity.

In addition to these challenges, setting up a new development environment from scratch evolved into a laborious, multi-day task for new team members. These issues significantly derailed engineering efforts, prompting the team to explore alternative options for local development and testing. It was during this exploratory phase that LocalStack emerged as a reliable solution, effectively alleviating major pain points related to onboarding, development administration, isolation, fault tolerance, and scaling individual developer environments.

## Solution

LocalStack came into the spotlight when CartonCloud's CTO sought a solution to emulate DynamoDB for a Java-based application. This discovery prompted CartonCloud's engineering team to embark on a 6-month pilot project. The objective was clear: transitioning their entire development environment to a local setup using LocalStack and Docker. The complexity of the task was evident, involving various Java and PHP services intertwined with AWS offerings like CloudFormation, ECR, RDS, AppSync, ElastiCache, and more.

Upon successful completion of the pilot project, CartonCloud engineers were able to standardize LocalStack across the entire development environment. They streamlined the process by introducing a single-shell script, `start.sh`, capable of booting up the entire development environment. This script not only installed all necessary dependencies but also drastically reduced the setup time from a daunting 3-5 days to just a few hours. CartonCloud emerged as an early adopter of LocalStack, marking a substantial enhancement for the team in expediting their engineering endeavours and enhancing the overall developer experience (DevEx).

Throughout this process, the CartonCloud team demonstrated innovation on multiple fronts. One notable achievement was the creation of the Hotpotato service — a NodeJS-based API proxy that exposes a static URL, forwarding requests to a configured Lambda in LocalStack through DNS resolution. Furthermore, Hotpotato monitors SQS queues, ensuring the efficient clearance of messages that have been received but remain unconsumed.

## Results

### 10x reduction in onboarding time

The integration of LocalStack with CartonCloud's developer environments has enabled the emulation of AWS services on developers' machines. This enhancement allows new team members to set up a complete local environment on their very first day, slashing the onboarding time by a factor of almost 10x. This pivotal change addresses challenges related to local development setups and eliminates hurdles around testing on the real cloud. Throughout the pilot project, developers got new laptops which further facilitated running cloud applications locally, which would otherwise be not possible without LocalStack.

### 15x cost and operational efficiency 

LocalStack has enabled more efficient development workflows by enabling the local testing of serverless functions, reducing the reliance on AWS for testing purposes. Furthermore, the elimination of the need for an individual AWS developer environment per developer has resulted in a 15x cost efficiency gain, considering the use of LocalStack licenses on a per-month, per-developer basis. LocalStack has significantly minimized instances of local environment breakdowns for developers, ensuring a more stable environment, fewer issues and a calmer development experience overall. 
