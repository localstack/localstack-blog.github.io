---
title: Add Authentication & Authorization to LocalStack with the Authress Extension
description: We are excited to announce the LocalStack Extension for Authress, which enables a high-fidelity, fully local authentication and authorization service and operates on your local machine.
lead: We are excited to announce the LocalStack Extension for Authress, which enables a high-fidelity, fully local authentication and authorization service and operates on your local machine.
date: 2024-01-21T12:41:54+05:30
lastmod: 2024-01-21T12:41:54+05:30
images: []
contributors: ["Harsh Mishra", "Warren Parad"]
tags: ['news']
---

## Introduction

The [LocalStack Extension for Authress](https://pypi.org/project/localstack-extension-authress/), our first [community extension](https://docs.localstack.cloud/user-guide/extensions/developing-extensions/), enables running Authress directly on your machine. It provides a high-fidelity, fully local authentication and authorization service, which bypasses the need to rely on the live version for local development.

Ensuring your users can access your software while maintaining the privacy of their data and clean tenant separation is not a trivial challenge. It’s not just about supporting different login mechanisms and protecting the personal data included in the user identity. It is also about deciding who gets to access what as the users interact with the software. The logic to support all of these security concerns gets complex real quick, and that complexity further increases whenever your software supports different data access patterns, multiple user roles, or nested resource hierarchies.
  
Building an in-house authentication and authorization system may sound simple, but it’s a task rife with problems that you may otherwise not need to face. These include complying with privacy regulations, ensuring high availability, and monitoring for security incidents - to name a few. And that’s before accounting for all the edge cases that come up as your software and user base evolves. [Authress](https://authress.io) directly tackles these challenges by providing a simple yet powerful API you can integrate into your software to take care of all your auth needs, from login and identity aggregation, to access control and endpoint protection.

## What is Authress?

[Authress](https://authress.io) is a login and access control API that lets you add authentication and granular authorization rules to any software you’re building. It’s easy to integrate into your project thanks to many different language SDKs, or, if you’re brave, directly over the REST API.

While developing your software, you need to provide a way for your users to access it securely. Depending on who your users are, you may need to support a variety of login methods, passwordless login, or SSO integrations. What’s more, your software is likely going to be used by different people, possibly also by different companies or tenants. This means that not all logged-in users are the same in terms of what they can access. If your access patterns are dictated by nested hierarchies, whether on the user group side, permission sets, resource side, or a combination of the above, decoupling access from identity becomes crucial for the security of your application.

Authress simplifies your software development by taking care of the complexity surrounding authentication and authorization. You only need to worry about your core functionality, what login methods to offer and what kind of access restrictions you want to put in place. You don’t even need to design all the access rules up front - Authress is inherently flexible and lets you easily change the permission schemas as your software evolves. It’s also inherently modular - whether it’s only user login, permissions, machine-to-machine authentication, or the entire auth stack, you can choose which parts to use.

Authress released [Authress Local](https://authress.io/knowledge-base/docs/SDKs/authress-local) in July 2023 to provide a local running version for integrating login, authentication, authorization, API keys, and enhanced security into your service's development & testing workflow. Authress Local mirrors the Authress Production API, enabling you to achieve a significant parity with the cloud service, and guaranteeing consistent performance when deployed in production.

## Why run Authress as a LocalStack Extension?  
 
[LocalStack Extensions](https://docs.localstack.cloud/user-guide/extensions/) allow you to customize LocalStack by starting custom services in the same container while leveraging the existing ecosystem & feature set. With Authress Local as an Extension, you now have an easier and faster way to setup user management and access control for your application in a local environment.

The LocalStack Extension for Authress replicates the Authress API, allowing you to integrate authentication & authorization directly into your locally running AWS applications. The Extension also allows you to jumpstart your development & testing workflows with Authress without an account. You can install the Authress Extension using the [LocalStack CLI](https://docs.localstack.cloud/getting-started/installation/#localstack-cli) or using the [Extensions Library](https://docs.localstack.cloud/user-guide/web-application/extensions-library/) on the [LocalStack Web Application](https://app.localstack.cloud/).


The [Authress extension](https://github.com/Authress/localstack-extension?tab=readme-ov-file) is loaded by the [LocalStack Extensions framework](https://docs.localstack.cloud/user-guide/extensions/) directly into your LocalStack environment. It works by dynamically starting up and configuring the [Authress Local](https://github.com/Authress/authress-local?tab=readme-ov-file#authress-local) container. The locally running container provides a copy of the [Authress API](https://authress.io/app/#/api) directly in your development environment so that you can:

* **Authenticate users** and generate JWTs offline without having to log in.
* **Increase security** during development by verifying your authorization checks are correctly configured.
* **Validate SDK and API** usage to catch any early bugs with your integration.
* **Orchestrate complete integration tests** combining your identity and authorization infrastructure with your AWS infrastructure already provided by LocalStack.

## How to use the LocalStack Extension for Authress

In this section, we’ll run a basic application that emulates authentication, authorization, user identity and role management by running the LocalStack Extension for Authress. For this tutorial, you’ll need to have the following prerequisites installed on your local machine:

-   [LocalStack CLI](https://docs.localstack.cloud/getting-started/installation/#localstack-cli)  
-   [LocalStack Web Application account](https://app.localstack.cloud/sign-up)   
-   [Node.js](https://nodejs.org/en/download/current)


### Install the LocalStack Extension for Authress

To install the extension, you can either use the LocalStack CLI or the Extensions Library on the LocalStack Web Application. To install the extension using the LocalStack CLI, start your LocalStack container, configure your [Auth Token](https://docs.localstack.cloud/getting-started/auth-token/) (`LOCALSTACK_AUTH_TOKEN`) as an environment variable and run the following command:

```bash
localstack extensions install localstack-extension-authress
```

You will see the following output after the installation of the extension is successful:

```bash
[13:03:07] Extension successfully installed         extensions.py:85
┏━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━━━━━━┓
┃ Name         ┃ Summary       ┃ Version ┃ Author   ┃ Plugin name  ┃
┡━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━━╇━━━━━━━━━━━━━━┩
│ localstack-… │ LocalStack    │ 0.1.23  │ Authress │ localstack-… │
│              │ Extension:    │         │          │              │
│              │ Authress      │         │          │              │
└──────────────┴───────────────┴─────────┴──────────┴──────────────┘
```

Alternatively, you can navigate to the [Extensions Library on the LocalStack Web Application](https://app.localstack.cloud/extensions/library). The library allows the installation and management of Extensions as simple as the click of a button. Click on the **+&nbsp;Install** button on the Web Application to install the extension on your local machine.

{{< img-simple src="localstack-extensions-library-authress.png" width=300 alt="A screenshot of the LocalStack Extensions Library showcasing the Authress Extension">}}

Irrespective of whether you use the CLI or the Extension Library to install the Extension, you will see the following in the LocalStack container logs:

```bash
2024-01-21T07:43:08.760  INFO --- [  MainThread] Authress                   : Starting up ghcr.io/authress/authress-local:latest as localstack-authress-af6bd0fc
2024-01-21T07:43:08.762  INFO --- [  MainThread] Authress                   : setting up proxy to http://localhost:8888
2024-01-21T07:43:09.778  INFO --- [e_proxy_job)] Authress                   : Authress API Started on: http://authress.localhost.localstack.cloud:4566
```

The Authress extension runs at `http://authress.localhost.localstack.cloud:4566/`. You can now configure API calls to the locally running authorization server specifying this URL as the `Authress API URL` in the Authress SDKs. In the following sections, we'll see just how to do that.

### Adding Authentication to your Application using Authress

Authress supports authentication and authorization. Here we’ll review how to configure one of the [Authress Starter kits](https://authress.io/knowledge-base/docs/introduction/quick-start-guides) for authentication and authorization, however, if you already have an application you can use that instead. The application code is located in [repositories on GitHub](https://github.com/Authress/react-starter-kit?tab=readme-ov-file#authress-starter-kit-react). 

Below is a sample application that showcases a basic React example that uses Authress to log in (for other languages and frameworks, see the other [starter kits](https://authress.io/knowledge-base/docs/introduction/quick-start-guides)). After logging in, you can navigate to the Authress protected page, which won’t be accessible to you unless you are logged in. Clone the repository onto your local developer machine with the following command:

```bash
git clone https://github.com/Authress/react-starter-kit.git 
```

After cloning the sample application, install the dependencies using the following command:

```bash
npm install
```

You can open the sample application in VS Code or your favourite IDE. Now that you have the Authress LocalStack extension running, navigate to the `src/authressClient.tsx` file, and complete the Authress configuration.  Update the `authressApiUrl` to point at the Authress LocalStack extension URL, in the following manner:

```typescript
loginClient = new LoginClient({
    applicationId: 'app_default',
    authressApiUrl: 'http://authress.localhost.localstack.cloud:4566',
  });
```

You can leave the rest of the configuration as it is, or make changes that suit your needs.  While using the extension with other language SDKs, you can pass the local Authress API URL as the `authressApiUrl` or the `authress_api_url` (depending on which SDK you are using).

### Running the Application

Navigate to the terminal and run the following command to start a local server:

```bash
npm run start
```

The application will now be available on [`http://localhost:8080/`](http://localhost:8080). You can open the URL on your browser, and you will see the following page.

{{< img-simple src="authress-react-starter-kit.png" width=300 alt="A screenshot of the Authress React Starter Kit">}}

You can try navigating to the protected page by clicking on the **Authress Protected Route Page**. You will immediately see a page remarking that you are not logged in, and the application has prevented you from accessing a protected page.

{{< img-simple src="you-are-not-logged-in.png" width=300 alt="A screenshot of the Authress React Starter Kit showing that you are not logged in">}}

On the main page or in the navigation bar, you can now click on **Login**, and you will be immediately logged in and redirected to the main page. On this page, you can now see a valid user authentication session with the user profile highlighted. Here, you will notice because you are offline, there is no redirection to the authentication page. Once your app is deployed to production a customized login box will appear to complete the login process.

Navigate to the LocalStack container logs to see the authentication APIs being triggered:

```bash
2024-01-10T09:01:10.725  INFO --- [   asgi_gw_0] localstack.request.http    : OPTIONS /api/authentication => 204
2024-01-10T09:01:10.793  INFO --- [   asgi_gw_0] localstack.request.http    : POST /api/authentication => 200
```

You can now click on the **Authress Protected Route Page** to see the protected page. 

{{< img-simple src="authress-logged-in-page.png" width=300 alt="A screenshot of the Authress React Starter Kit showing that you are now logged in">}}

### Adding Authorization to your application

Above, we successfully added authentication and user login to an application using the [Authress React Starter Kit](https://authress.io/knowledge-base/docs/SDKs/javascript?source=localstack&sdk=react). Additionally, Authress can also provide authorization, access control, and api key management to web services. That means we can use the Authress LocalStack Extension to enable offline authorization.

Here, we’ll see how that works by using the [Authress Starter Kit for Express](https://authress.io/knowledge-base/docs/SDKs/javascript?source=localstack&sdk=express). Alternatively, if you already have a running local application, you can add Authress authorization to that instead. For using the starter kit, clone the repository onto your local developer machine with the following command:

```bash
git clone https://github.com/Authress/express-starter-kit.git
```

After cloning the sample application, install the dependencies using the following command:

```bash
npm install
```

As with the React application, the next step is to update the Authress configuration. Navigate to the `src/authressPermissionsWrapper.tsx`, and update the `authressApiUrl` to point at the Authress LocalStack extension URL.

```typescript
const serviceClientAccessKey = 'sc_c.0.acc.MC4CAQAwBQYDK2VwBCIEIALXnuHrzrj++Q54hCHI/GvULrovX9SNis8+rFKgEM2v';
const authressApiUrl = 'http://authress.localhost.localstack.cloud:4566';
```

Make sure to also update the `serviceClientAccessKey` to any valid key. The one shared above can be used.

### Testing your application authorization

Now that the service application is configured to use the Authress LocalStack Extension, you can start the service.
Navigate to the terminal and run the following command to start a local server:

```bash
npm run start
```

If the service starts correctly, you should see output from the local service:

```bash
> nodemon src/index.ts
[nodemon] 2.0.21
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: ts,js,json
[nodemon] starting `node --loader ts-node/esm src/index.ts`
(Use `node --trace-warnings ...` to show where the warning was created)
App Running on http://localhost:8080

*******************************************
Try hitting one of the existing endpoints: curl -XGET http://localhost:8080/accounts -H"Authorization: Bearer YOUR_TOKEN"
```

Since this is an API based service, we can validate it works correctly, by making a call to one of the protected resources in this service. If you are using your own application service, choose one of the available endpoints there.

The [Authress Starter Kit for Express](https://authress.io/knowledge-base/docs/SDKs/javascript?source=localstack&sdk=express) has user, account, and resource endpoints already available. Here you can test the resource endpoint. And to do that, we first need a valid access token. You can generate a valid access token in the [Authress Management Portal](https://authress.io/app/#/authress-local?focus=token-generation) or you can receive one from the Authress Extension.

To get an access token from the Authress Extension, run the following command:

```bash
curl -XPOST -d'{ }'
  http://authress.localhost.localstack.cloud:4566/api/authentication
```

The result will be an access token within an JSON response:

```bash
200
{
    "accessToken":
            "eyJ0eXAiOiJKV1QiLCJhbGciOiJFZERTQSIsImtpZCI6ImF1dGhyZXNzLWxvY2FsIn0.eyJhdWQiOiJsb2NhbGhvc3QiLCJleHAiOjE3MDU4NjIzNjksImlhdCI6MTcwNTc3NTk2OSwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4ODg4Iiwic3ViIjoibWUifQ.F_PiFTg7ir0bDhet_AVs6aeIxbBycXUso-J7sUA22iWK6p4pOUFoyMJTr6ZD_GCEQqI314WMmjllK9GZ9spTBQ"
}```
```

Copy out that access token and use it with the running application service:

```bash
curl http://localhost:8080/accounts/001/items/resource_001 
 -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJFZERTQSIsImtpZCI6ImF1dGhyZXNzLWxvY2FsIn0.eyJhdWQiOiJsb2NhbGhvc3QiLCJleHAiOjE3MDU4NjIyMDIsImlhdCI6MTcwNTc3NTgwMiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4ODg4Iiwic3ViIjoibWUifQ.bXGudtoYcEjohAWtJLeUgGWeh5PtbXIXNCVa4EEasjag8TqZaYfJA6MaUs3KA8owYKxm2wZTszRNx2Ayd-n_Cw"

200
{...}
```

### Conclusion

Congratulations! You’ve successfully added authentication and authorization to your running application on your machine. You can further explore Authress APIs and their cloud offering for various use cases such as:

-   Quickly adding user authentication to your application with an [Authress Login Box](https://authress.io/knowledge-base/docs/authentication/user-authentication#login-configuration)
-   Adding authorization and [securing a Multitenant application architecture](https://authress.io/knowledge-base/articles/creating-a-multitenant-application#a-multitenant-application)
-   [Implementing signup and user onboarding flows](https://authress.io/knowledge-base/docs/usage-guides/onboarding-users) for enterprise businesses
-   … and more!

The Authress Extension for LocalStack makes it easy to manage and integrate authentication and authorization into locally-created cloud resources, and we hope to make your development process easier and faster! If you are interested in developing a [LocalStack Extension](https://docs.localstack.cloud/user-guide/extensions/) to customize and extend your local development experience, check out our [guide on building LocalStack Extensions](https://docs.localstack.cloud/user-guide/extensions/developing-extensions/).

If you have questions about Authress and how it enables login and role or resource-based granular access control, check out the [Auth Academy](https://authress.io/knowledge-base/academy/topics) or join their [Developer community](https://authress.com/community).
