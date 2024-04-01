---
title: Ephemeral environments with LocalStack & Shipyard
description: We're excited to announce our partnership with Shipyard ephemeral environments. This integration empowers you to test your cloud applications in a short-lived, encapsulated deployment — allowing you to shift left by fixing things before they break your production.
lead: We're excited to announce our partnership with Shipyard ephemeral environments. This integration empowers you to test your cloud applications in a short-lived, encapsulated deployment — allowing you to shift left by fixing things before they break your production.
date: 2024-04-01T9:41:21+05:30
lastmod: 2024-04-01T9:41:21+05:30
images: []
contributors: []
tags: ['showcase']
---

## Introduction

We’re excited to announce our partnership with Shipyard, the ephemeral environment Self-Service platform, that allows you to spin up on-demand deployments via Kubernetes clusters to turbocharge your application lifecycle. With Shipyard, you can now deploy your cloud applications in a short-lived environment to enable developer teams to run tests, preview features, and get alignment with cross-departmental projects. Shipyard also enables application previews on every pull request and simplifies monitoring, debugging, and deployment all from one dashboard!

LocalStack’s core cloud emulator enables developers to build, test, and deploy cloud & serverless applications locally. Shipped as a Docker image, you can use various integrations such as the `docker` CLI, Docker Compose, or Helm to start LocalStack in a developer environment. Shipyard allows developers to use Docker Compose configurations which are then automatically transpiled into Kubernetes manifests enabling you a pre-production preview of how your applications work! As such, LocalStack does not require any additional configurations and just enables you to start with a pre-defined Docker Compose setup.

In this blog, we’ll detail why you should be using ephemeral environments and how you can use LocalStack alongside Shipyard. We’ll also go through setting up an AWS-powered application on Shipyard & LocalStack, and how you can pre-seed infrastructure state using Cloud Pods!
