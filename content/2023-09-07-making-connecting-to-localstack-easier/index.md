---
title: How we are making connecting to LocalStack easier
date: 2023-09-07
lastmod: 2023-09-26
lead: Connecting your applications to LocalStack has not always been easy. In this post we describe the journey we went through to streamline the LocalStack networking experience.
tags:
- news
contributors:
- Simon
- Daniel
- Joel
---


<!-- picture -->

<!-- why is connecting hard? -->
LocalStack normally runs in a Docker container, meaning that it is isolated from the host system.
By default, LocalStack _publishes_ its edge port (usually `4566`) to the host.
Publishing a port means that a port on the host forwards network communications to the LocalStack container.
Requests made to `localhost:4566` are then forwarded to the container.

This works well when interacting from the host, for example using [`awslocal`](https://docs.localstack.cloud/user-guide/integrations/aws-cli/#localstack-aws-cli-awslocal) commands.
However, making requests to `localhost:4566` does not work when trying to connect to LocalStack from your own containers, or LocalStack compute resources such as Lambda functions or ECS containers.

Sometimes, users wish to use multiple different methods to connect to LocalStack at the same time.
For example, application code running on the host triggers a Lambda function, which in turn invokes more AWS services.
In this situation, there is not one single hostname that can be reached from a Lambda function (which runs in a separate Docker container) and the host machine.

As per usual, our fantastic community have been very resourceful in trying to solve this problem.
One idea was to connect their application containers to the host network (`--network host`) or by making requests to `host.docker.internal:4566` when using Docker Desktop.
In some cases, using the host networking solves the problem, but it causes other problems:

1. If SSL is used, then certificate validation must be turned off:
    * LocalStack presents a certificate for a set of registered domains;
    * if using host networking (`--network host`), requests are made to an IP address or `localhost`, which is not included in the certificate; and
    * when using the gateway domain (`host.docker.internal`), this domain is also not included in the set of certificate domains.
2. Subdomains created by resources such as S3 buckets or OpenSearch clusters will not resolve to the LocalStack container.
3. host port can only be published once, whereas container ports are separate from each other and multiple containers can publish the same port.

We already solve the first two issues by using the domain name `localhost.localstack.cloud` in our documentation and examples.
This domain name is publicly registered and resolves to the IP address `127.0.0.1`.
Any (possibly nested) subdomain of this domain name also resolve to `127.0.0.1`.
This allows us to present a valid TLS certificate when using HTTPS from the host, but does not remove the connectivity problem.
You can check that the domain maps to `127.0.0.1` by running:

```sh
dig @8.8.8.8 localhost.localstack.cloud
```

You will see the following output:

```
; <<>> DiG 9.10.6 <<>> @8.8.8.8 localhost.localstack.cloud
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 54676
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;localhost.localstack.cloud.	IN	A

;; ANSWER SECTION:
localhost.localstack.cloud. 600	IN	A	127.0.0.1

;; Query time: 54 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Fri Sep 08 11:23:20 BST 2023
;; MSG SIZE  rcvd: 71
```

This command queries the Google public nameserver (`8.8.8.8`) for the `localhost.localstack.cloud` domain.
In the "ANSWER" section, an `A` record resolves our domain to the IP address `127.0.0.1`.

Unfortunately, this domain name is not suitable for use in compute environments such as Lambda functions.
When you create a Lambda function, ECS container, or EC2 instance, we create a new Docker container running your application code.
Prior to LocalStack v3.0, the domain name `localhost.localstack.cloud` did not resolve to the LocalStack container as you may have expected, but the compute environment container itself.

So how did we go about making connectivity to LocalStack easier?

* Providing helpful advice
* Introducing dynamic name resolution with our DNS server
* Updating and improving our advanced configuration
* Providing tooling to help users debug their network configuration

# Providing helpful advice

We created comprehensive troubleshooting advice in the form of our [Network troubleshooting guide](https://docs.localstack.cloud/references/network-troubleshooting/).
We outlined different user setups and described settings were best practice, and were proven to work.

{{< img src="docs-screenshot.png" >}}

With this guide, common networking scenarios are described, with example configuration for achieving connectivity.


Our main suggestion involved relying on Docker's networking capabilities, and for the user to use docker networks.
In this mode, the name of the LocalStack container resolves correctly.
Unfortunately, this is not without its limitations.
Mainly that subdomains do not resolve to a valid IP address, and LocalStack had to be configured to return its container name in resource identifiers such as URLs, rather than `localhost`.
Previous to this initiative, we supported setting `HOSTNAME_EXTERNAL` and `LOCALSTACK_HOSTNAME` to provide this functionality.
The user could set `HOSTNAME_EXTERNAL=localhost.localstack.cloud` to gain the benefits of subdomain support and TLS certificates, though its use in Lambda functions was still a problem.
Unfortunately, the use of these two configuration variables within LocalStack services was inconsistent, or worse: nonexistent, and there was confusion as to why two variables were needed to support the same functionality.

There needed to be a more general solution that would reduce the amount of complexity for users, as well as providing seamless connectivity.

# Dynamic name resolution

We wanted our users to be able to use the same domain name regardless of where their code was running from.
So for example, on the host `localhost.localstack.cloud` would resolve to `127.0.0.1`, but inside a separate container (such as a Lambda function, or the user's own docker container) the name would resolve to the IP address of the LocalStack container.

To implement this feature, we designed a system based on DNS.
We brought our existing DNS server from LocalStack Pro into the LocalStack Community edition, and updated it to to support this new use case.
Requests made to our DNS server to resolve the name `localhost.localstack.cloud` will respond with the IP address of the LocalStack container.
Requests made to resolve names that we don't specifically handle (e.g. `example.com`) will be forwarded to your system DNS resolver:

{{< mermaid >}}
stateDiagram-v2
    direction LR
    Application --> LocalStack
    LocalStack --> Upstream
{{< /mermaid >}}

We are now able to resolve the three issues mentioned above:

1. If SSL is used, then certificate validation must be turned off since LocalStack does not present a valid certificate for the domain used (either `localhost` or `host.docker.internal`).
    * LocalStack presents a valid certificate for `*.localhost.localstack.cloud` domains.
2. Subdomains created by resources such as S3 buckets or OpenSearch clusters will not resolve to the LocalStack container.
    * Subdomains of `localhost.localstack.cloud` also resolve to the LocalStack container (for example `mybucket.s3.us-east-1.localhost.localstack.cloud`). 
3. Each host port can only be published once, whereas container ports are separate from each other and multiple containers can publish the same port.
    * Now all inter-container networking can be done over the Docker network, and no ports have to be published to the host at all.

So how can you make use of this new feature?

For AWS services like Lambda or ECS, we are running your application code in an environment pre-configured to use this feature.

_For your own containers, there is some configuration required._

Docker allows the configuration of a container DNS resolver.
This is done by overriding the `/etc/resolv.conf` file inside the container.
When using the Docker CLI, you can use the `--dns` flag, or the `dns:` entry of a Docker Compose service.
This flag accepts an IP address to use for resolving domain names.
To use this flag, your LocalStack container will need to have a known IP address.

For example, when using Docker Compose, you can specify the IP address that the LocalStack container will be assigned by using a user-defined network, and using the `ipam` configuration settings.
An example configuration would look similar to:

```yaml
services:
  localstack:
    image: localstack/localstack
    networks:
      ls:
        # Set the container IP address in the 10.0.2.0/24 subnet
        ipv4_address: 10.0.2.20

  # Example application container that connects to LocalStack
  application:
    image: amazon/aws-cli
    depends_on:
      - localstack
    command: ["s3api", "list-buckets"]
    environment:
      - AWS_ACCESS_KEY_ID=test
      - AWS_SECRET_ACCESS_KEY=test
      - AWS_ENDPOINT_URL=http://localhost.localstack.cloud:4566
    dns:
      # Set the DNS server to be the LocalStack container
      - 10.0.2.20
    networks:
      - ls

networks:
  ls:
    ipam:
      config:
        # Specify the subnet range for IP address allocation
        - subnet: 10.0.2.0/24
```

We have created a demo application to demonstrate this functionality: [https://github.com/localstack/networking-demo-application](https://github.com/localstack/networking-demo-application).
This sample uses `*.localhost.localstack.cloud` throughout to seamlessly configure AWS SDK clients to communicate with LocalStack.

* The deploy process runs in a separate Docker container.
* The application container connects across the Docker network to LocalStack.
* A Lambda function communicates with LocalStack to subscribe to SQS messages, access objects in S3, and write to a DynamoDB table.

# Supporting additional customization

Our aim with the networking improvements was to make the default configuration by default.
Despite this, it might be the case that additional customization is required.
For example, some users cannot configure their application Docker containers, or they wish to change the port LocalStack binds to.
For those users who require additional customization, we provide two approaches:

* "functional" configuration, and
* "cosmetic" configuration.

## Functional configuration

This type of configuration changes the behaviour of LocalStack.
Of most relevance to this article, we previously used the configuration variables:

* `EDGE_PORT` (default: `4566`)
* `EDGE_PORT_HTTP` (default: `0`)
* `EDGE_BIND_HOST` (default: `0.0.0.0` if inside the docker container, `127.0.0.1` outside)

to define what host and port the LocalStack server bound to.

These variables only allowed customization of a single bind host, and one or two ports.
We stopped using different ports for HTTP and HTTPS in 2019, so the names were not accurate.
It was also a lot of similar sounding configuration to configure these two bind addresses.

With LocalStack 2.0 we introduced `GATEWAY_LISTEN` as an alternative, which allowed multiple listen addresses to be specified in a single configuration variable.
This meant more flexibility for our users, and a simpler configuration.
Multiple bind addresses could be configured with a single variable.
With this change, we reduced the number of required configuration variables down to 1.

For example, the following configuration:

```
EDGE_PORT_HTTP=5000 EDGE_PORT=9000 EDGE_BIND_HOST=0.0.0.0 localstack start
```

can now be set with:

```
GATEWAY_LISTEN=0.0.0.0:5000,0.0.0.0:9000 localstack start
```

## Cosmetic configuration

This type of configuration changes the domain names and ports returned by services that return URLs.
For example: before the start of this initiative, creating an SQS queue returned a queue URL:

```bash
$ awslocal sqs create-queue --queue-name myqueue
{
    "QueueUrl": "http://localhost.localstack.cloud:4566/000000000000/myqueue"
}
```

but this domain resolved to `127.0.0.1` only, and as such was not usable from other Docker containers.
From very early on in LocalStack's history, this was accounted for via the "cosmetic" configuration variables: `LOCALSTACK_HOSTNAME` and `HOSTNAME_EXTERNAL`.

If our DNS-based improvements are not available, or do not solve the connectivity problem, the user can configure cosmetic variables to a name that resolves to the LocalStack container.
Where previously there were two variables that performed the same role in an inconsistent way, we now have a single variable: `LOCALSTACK_HOST` which is used internally by all services that return URLs.

For example, by running LocalStack with:

```
LOCALSTACK_HOST=foo.bar:5000
```

a queue URL returned by SQS will be:

```bash
$ awslocal sqs create-queue --queue-name myqueue
{
    "QueueUrl": "http://foo.bar:5000/000000000000/myqueue"
}
```

Even though the port `5000` is included in the URL in this example, the port specified by `LOCALSTACK_HOST` and `GATEWAY_LISTEN` may be different.

# Providing tooling to help debug your network configuration

The final part of this networking initiative was to provide a way for you to debug your networking configuration.
We released a tool: [https://github.com/localstack/localstack-docker-debug](https://github.com/localstack/localstack-docker-debug) that provides advice for users who are facing connectivity issues.

For example, if your application code is running in a separate docker container, but that container cannot talk to LocalStack, you can run:

```bash
docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    ghcr.io/localstack/localstack-docker-debug:main \
        diagnose \
        --source-container "<application-container-name>" \
        --target-container "localstack-main" \
        --localstack
```

the tool attempts to connect to LocalStack.
If it cannot, it temporarily adjusts the networking configuration of the application container name until connectivity is reached.
Once this occurs, it prints helpful suggestions on what changes were needed to make the connection.
If this does not work, the tool is able to capture your Docker network topology to help us understand your networking layout.

# Conclusions

We hope that with this new functionality available today, accessing LocalStack should be considerably easier.
By moving the DNS server into LocalStack and configuring spawned AWS compute environments to use it by default, your Lambda functions, ECS containers, and EC2 instances should already be able to access LocalStack at `localhost.localstack.cloud`.
With a small change in configuration, your application containers will also be able to reach LocalStack at `localhost.localstack.cloud`.

If further customization is required, we have streamlined and expanded on configuring both LocalStack itself, as well as cosmetic URLs returned by services that may be required for more complex networking setups.

Finally, if you have difficulties connecting to LocalStack, we provide a debug utility to help diagnose the cause of the problems.

As always, let us know if any issues using the [GitHub issue tracker](https://github.com/localstack/localstack/issues), or if you are a Pro customer, feel free to [reach out to us directly](https://docs.localstack.cloud/getting-started/help-and-support).
We want to hear your feedback on our networking initiative, so please get in touch via ou!
