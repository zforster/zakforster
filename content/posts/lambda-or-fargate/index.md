+++
date = "2025-08-30T15:00:00+01:00"
draft = true
title = "AWS Lambda or AWS Fargate: Comparing Serverless Compute Services"
categories = ['AWS', 'Lambda', 'Fargate', 'Serverless']
+++

## Introducing Serverless

All applications require a layer of compute in order to perform the tasks we require of them. Before the advent of cloud computing, companies would aquire their own server infrastructure and run their applications on-premise. At this time, scaling was typically vertical in nature, meaning bigger, more powerful machines were installed to satisfy compute and database demand.

With the launch of cloud computing and services such as [EC2](https://aws.amazon.com/ec2/) it was no longer required to physically aquire hardware or manually scale compute; virtual servers could now be rented from cloud providers on demand. However, technology teams were still responsible for maintaining and patching the operating system and applications on those servers, even when scaling was handled automatically through features like auto-scaling groups.

The introduction of [serverless](https://aws.amazon.com/serverless/) fundamentally changed the way compute was provisioned. Serverless solutions abstract away both server maintainence and capacity planning, allowing engineers to focus on the code to solve a particular buisness problem, rather than the supporting infrastructure. Despite the name, applications using serverless architectures still run on virtual servers, we as engineers are simply abstracted away from those instances; both AWS Lambda and AWS Fargate utilise lightweight [firecracker](https://firecracker-microvm.github.io/) microVMs under the hood.

## What Is AWS Fargate?

Fargate is a serverless compute engine that enables you to scale containerised applications. As it operates on containers it is very flexible; Fargate allows you to run any application that you can containerise. This is in contrast with AWS Lambda, which unless you are willing to understand [how to configure a custom runtime](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-custom.html) binds you to those that AWS have pre-configured such as Python, NodeJS or Java. As such, if you require the usage of a language outside of those that are officially supported, using Fargate is the suitable choice.

### Fargate Components

To run an application with Fargate, the following components are required:

#### Elastic Container Registry\*

[Amazon Elastic Container Registry](https://aws.amazon.com/ecr/) (ECR), is a managed container registry that hosts your application images. ECR serves as a location from which the Fargate tasks and services can pull the application image that they need to run.

\*ECR is not strictly required as you can use alternative registries, although ECR is the easiest and most convient option.

#### ECS Cluster

An [ECS Cluster](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/clusters.html) is a logical grouping of tasks and services that run your application. It defines the boundary within which tasks or services are executed. For instance, you could run distinct **dev**, **test** and **prod** environments as seperate clusters, or run two different workloads such as a web API and a front-end within two different clusters.

#### Task Definition

The [Task Definition](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html) serves as the blueprint to describe how to run an instance of your application. The configuration includes: CPU and memory requirements, the IAM role to be used by the running task, logging configuration, and the image to be pulled and run.

#### Task

The task is an individual running instance of the application.

#### ECS Service

When running a service such as an API, it is important to maintain a minimum number of instances that are always available to respond to requests. This is what the [ECS Service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html) achieves. The service ensures we have a desired number of tasks running at all times; if a task was to exit or fail, the service will automatically restart it in an attempt to auto-heal.

The secondary function of the service is to handle application scaling; it is here that we attatch auto-scaling policies that can automatically provision more task instances to meet compute demand based on CPU or memory usage metrics.

## What Is AWS Lambda?

[AWS Lambda](https://aws.amazon.com/lambda/) is a serverless compute engine that runs a function in response to events. With Lambda it is feasible to create an **event-driven architecture**. There are two types of events that can trigger lambda functions: push events such as HTTP requests and S3 uploads, and pull events that replace the need for polling by fetching from a queue or database stream.

Event driven architectures are typically more efficient that polling or batch architectures; data is processed in near real time, the application only runs when required, and the handlers are typically smaller in scope and less complex.

Lambda function definitions must contain a handler that specifies the function to execute when the event is triggered, along with the location of the function within the source code.

#### Packaging Methods

When you create a Lambda function, you package your function code into a deployment package. Lambda has two methods to achieve this:

The first method is [deploy your lambda function as a .zip file](https://docs.aws.amazon.com/lambda/latest/dg/configuration-function-zip.html). This method constrains your application and dependency size signficiantly; the maximum uncompressed file size of the uploaded zip file is only 250 mbs. This may be fine for a smaller function, but one requiring multiple larger dependencies will quickly hit this limit. The second method is to [create a lambda function using a container image](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html). This involves building an image and uploading it to an ECR repository. The maximum size constraints are more generous when containerising, with a maximum uncompressed image size of 10 gb.

The size restrictions are logical as they reflect the fact that the ideal Lambda function is lightweight, limited in scope and fast to spin up. Having larger file sizes would incur additional I/O overheads upon start up, leading to a slower cold start, something that doesn't impact Fargate architectures in the same way.

If you have particuarly large dependencies, containerising and running with fargate is the more suitable option.

---

{{< newsletter >}}

---
