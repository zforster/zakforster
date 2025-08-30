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

Fargate is a serverless compute engine that enables you to scale containerised applications. As it operates on containers it is very flexible, allowing you to run any application that you can containerise without being bound by any particular run times. Thi is in contrast with AWS Lambda, which unless you are willing to understand how to 

## What Is AWS Lambda?
<!-- ---

{{< newsletter >}}

--- -->
