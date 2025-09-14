+++
date = "2025-09-07T15:00:00+01:00"
draft = false
title = "AWS Lambda or AWS Fargate: Comparing Serverless Compute Services"
categories = ['AWS', 'Lambda', 'Fargate', 'Serverless']
+++

All applications require a layer of compute in order to process, transform and store data. The means by which compute is provisioned has evolved over time; before the advent of cloud computing, it was necessary to acquire physical server infrastructure, and run applications directly on those machines, a method known as **running on-premise**.

The launch of cloud computing and associated services such as [EC2](https://aws.amazon.com/ec2/) removed the need to physically acquire hardware: it could now be rented on demand as was necessary. This however, still came with the obligation to patch and maintain those virtual servers.

The next technological leap came with the introduction of [serverless](https://aws.amazon.com/serverless) architectures, which fundamentally changed the way modern applications are constructed. Serverless solutions abstract away both server maintenance and capacity planning, allowing engineers to focus on the code to solve a particular problem, as opposed to the supporting infrastructure. Despite the name, applications using serverless architectures still run on virtual servers; we as engineers are simply abstracted away from those server instances.

Two popular serverless compute engines are [AWS Fargate](https://aws.amazon.com/fargate/) and [AWS Lambda](https://aws.amazon.com/lambda/). In this article, we compare and contrast these two compute engine to help you make a more informed decision as to which most suits your project needs.

## Introducing AWS Fargate

Fargate is a serverless compute engine that enables the scaling of containerised applications. As it operates on containers it is very flexible, allowing you to run any application that you can containerise. This is in contrast with AWS Lambda, which unless you are willing to understand [how to configure a custom runtime](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-custom.html), binds and limits you to runtimes that AWS has pre-configured such as Python, NodeJS or Java. As such, if you require the usage of a runtime language outside of those that are officially supported by Lambda, using Fargate is the clear choice.

The fact that Fargate runs containers has another advantage in that you can easily migrate an existing containerised application onto the cloud without the need for heavy re-architecting.

### Fargate Components

To run an application with Fargate, the following components are required:

#### Elastic Container Registry\*

- [Amazon Elastic Container Registry](https://aws.amazon.com/ecr/) (ECR), is a managed container registry that hosts your application images. ECR serves as a location from which the Fargate tasks and services can pull the application image that they need to run.

- \*ECR is not strictly required as you can use alternative registries, although ECR is the easiest and most convenient option given it exists with AWS.

#### ECS Cluster

- An [ECS Cluster](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/clusters.html) is a logical grouping of tasks and services that are responsible for running your application. It defines the boundary within which tasks or services are executed. For instance, you could run distinct **dev**, **test** and **prod** environments as separate clusters, or decide to run a web API and a front-end within different clusters to clearly segregate their concerns.

#### Task Definition

- The [task definition](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html) serves as the blueprint to describe how to run an instance of your application. The configuration includes: CPU and memory requirements, the IAM role to be used by the running task, logging configuration, and the application image to be pulled from the image registry.

#### Task

- The task is an individual running instance of the application, it runs the image pointed to at the task definition level.

#### ECS Service

- When running a service such as an API, it is important to maintain a minimum number of instances that are always available to respond to requests. This is what the [ECS Service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html) achieves. The service ensures we have a desired number of tasks running at all times; if a task was to exit or fail unexpectedly, the service will automatically restart it in an attempt to auto-heal.

- The secondary function of the service is to handle application scaling; it is here that we attach auto-scaling policies that can automatically provision more task instances based on CPU or memory usage metrics, therefore ensuring the application meets the user compute demand.

## Introducing AWS Lambda

[AWS Lambda](https://aws.amazon.com/lambda/) is a serverless compute engine that runs a function in response to events. With Lambda, it is feasible to create an **event-driven architecture**. There are two types of events that can trigger the execution of Lambda functions: push events such as HTTP requests or S3 uploads, and pull events that replace the need for polling by fetching from an SQS queue or database stream.

Event-driven architectures are typically more efficient than polling or batch architectures: data is processed in near real time, the application only runs when required which can reduce costs, and the handlers are smaller in scope and less complex as they only handle specific events.

Lambda function definitions must contain a handler that specifies the function to execute when the event is triggered, along with the location of the function within the source code.

#### Packaging Methods

When you create a Lambda function, you package your function code into a deployment package. Lambda has two methods to achieve this:

The first method is to [deploy your Lambda function as a .zip file](https://docs.aws.amazon.com/lambda/latest/dg/configuration-function-zip.html). This method constrains your application and dependency size significantly; the maximum uncompressed file size of the uploaded zip file is only 250 MB. This may be fine for a smaller function, but one requiring multiple larger dependencies will quickly hit this limit.

If you function exceeds the maximum permitted size you can [create a Lambda function using a container image](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html). This involves building an image and uploading it to ECR. The maximum size constraints are more generous when containerising, with a maximum uncompressed image size of 10 GB permitted per function.

The size restrictions are logical as they reflect the fact that the ideal Lambda function is lightweight, limited in scope and fast to spin up. Having larger file sizes would incur additional I/O overheads upon start-up, leading to a slower cold start, something that doesn't impact Fargate architectures in the same way. If you have particularly large dependencies, you will need to containerise. If your image exceeds the limit. running the image with Fargate is an option, which will be very easy to do given the application is already containerised.

## Comparing Fargate & Lambda

#### Execution Duration

A key consideration when choosing between Fargate or Lambda is the execution duration supported by each compute engine.

If your application requires long-running compute exceeding 15 minuets, for example when executing overnight batch jobs, then Fargate is the clear choice; Fargate tasks have no timeout and they can run for as long as is necessary until processing is complete.

In contrast, Lambda is designed for shorter-lived use cases; the maximum Lambda execution duration is 15 minutes. Lambda could be a great option if you are developing a greenfield application that can take advantage of an event-driven architecture, or has lightweight workloads that you are confident can process within the time constraints.

#### Application Scaling

Being serverless solutions, both Lambda and Fargate handle application scaling for you, albeit with slightly different methods.

Lambda scales horizontally and independently per request. If a sudden burst of 100 requests hits our Lambda-based API, 100 independent Lambda executions are triggered concurrently. As each of these invocations are independent and short-lived, no shared state exists between them.

Recently invoked Lambdas stay **warm** for a period of time. If a new request is received within the warm peroid an initialised environment is immediately available to process the request. If however, a request is received and no warm Lambdas are available in the pool, the execution will be subject to a **cold start**. A cold start is an additional period of latency lasting between 100 ms to 2 seconds, within which AWS will initialise a lightweight [Firecracker VM](https://firecracker-microvm.github.io/) and pull your source code or container image. It is for this reason that it is best to keep Lambda package sizes small as the I/O influences the cold start duration that adds additional latency for the end user.

To avoid cold starts it is possible to provision concurrency, this means you always have a certain number of warm Lambdas available to immediatley process a request. This introduces additional cost, and since you are essentially running an instance at all times to avoid the cold start, this could be an indication that Fargate is more suited to your needs.

Lambda is fully responsive to your traffic patterns, making it is a great candidate for spiky, irregular and hard to predict workloads. If no event is received, nothing will run. If a large volume of traffic suddenly arrives, it will scale horizontally to meet the demand.

In Fargate, scaling is handled differently. All user requests are routed to an existing running instance. The benefit of this is that your end user is not subject to a cold start as there is always a running instance immediately available to process the incoming request while the new instance spins up. New instances of your application are only created if the CPU or memory usage of the existing instances hit the limits defined within the Fargate service definition.

Fargate services are generally much slower to scale than Lambda. Time to launch a new Fargate task ranges from 35 seconds to 2 minutes. A contributing factor to this is the heavier I/O resulting from the larger image sizes permitted. If all running containers are at maximum capacity and another request comes in, the request will still be routed to an existing instance even if it is at max capacity. Once the new instance is available, future requests can be routed to that newly created instance. As with Lambda, the running instances are independent and do not share state.

Due to the slower scaling speeds, a system utilising Fargate requires some thought as to the minimum provisioned capacity you make available to ensure you can meet typical user demand before scaling occurs. Fargate is a good option if you need to reduce latency, have relatively predictable traffic, and cannot suffer a cold start.

#### Instance Configuration

Of the two computing methods, Fargate services are more freely configurable. With Fargate, CPU and memory are independent variables, meaning you can configure each, tailoring your service to reflect if it is more CPU or memory intensive.

Lambda is also configurable, but to a lesser extent as CPU and memory configuration is linked. This fact renders Lambda less flexible for workloads needing specific CPU or memory allocations as you cannot tailor the resources specifically. Lambda does not enable the CPU to be altered directly, that option only exists for memory, which can be adjusted from 128 MB to 10 GB. If you increase the memory AWS automatically increases the CPU in proportion, up to a maximum of 6 vCPUs. Roughly each 1,800 MB increase to memory corresponds to an additional 1 vCPU being added to the resourcing. This means to get more compute you need to increase the memory.

It should also be noted that the total memory and CPU limitations are more restrictive for Lambda than they are on Fargate. For highly intensive tasks, Fargate may be the preferred choice.

#### Response Sizes

When designing your application, consideration should be given to the expected payload sizes you will be returning from your API services. Using Lambda can be incredibly restrictive. If you decide to use an API Gateway and Lambda-based architecture your maximum response size is 6 MB. This is the maximum return size for Lambda. This drops to just 1 MB if you choose to use Lambda with an Application Load Balancer (ALB). Using Lambda without recognising this limitation leads to the implementation of workarounds such as storing the data in S3 and returning it via a pre-signed URL to bypass the Lambda entirely. While this solution works, in a corporate environment using pre-signed URLs may not be possible due to the security considerations they introduce.

Fargate allows for 10 MB of data when used with API Gateway. When using with an ALB there isn't a size limitation.

## Summary

AWS Lambda and AWS Fargate are both **serverless compute services** from AWS, but they serve different use cases depending on workload requirements.

Fargate runs containerised applications without managing servers. It is highly flexible, supports any language that can be containerised, and is well-suited for **long-running, predictable workloads**. Fargate offers configurable CPU and memory, larger image sizes, and no execution time limits. However, scaling is slower (35s–2min), so some **minimum capacity planning** is often necessary.

Lambda runs functions in response to events, enabling **event-driven architectures**. It is best suited for **short-lived, spiky, unpredictable workloads** with execution durations up to **15 minutes**. Lambda scales instantly per request but can suffer from **cold starts**, especially with larger deployment packages. It is less configurable than Fargate, with CPU tied to memory (max 6 vCPUs and 10 GB RAM), and response sizes are limited (6 MB with API Gateway, 1 MB with ALB).

---

{{< newsletter >}}

---
