+++
date = "2025-06-29T20:00:00+01:00"
draft = true
title = "Build Rapid AWS Serverless Prototypes with Moto + Python 🚀"
categories = ['Python', 'Software Engineering', 'Moto', 'AWS']
+++

## Introducing Moto

[Moto](https://docs.getmoto.org/en/latest/) is a popular Python library used by developers to mock out calls to AWS services. Moto's functionality and ease of use have made the library the preferred choice for Python engineers seeking to unit test their applications without needing to deploy real cloud resources in AWS.

Moto does not have full coverage of AWS services and functionality, however the library is in active development, and I have found it caters for the most common use cases. It especially shines for services commonly used in a serverless stack such as DynamoDB and S3, which at the time of writing having 68% and 72% coverage respectiveley. The full list of implementation details are [available here.](https://github.com/getmoto/moto/blob/master/IMPLEMENTATION_COVERAGE.md)

## Server Mode

While the praise for Moto as a testing library is justified, a lesser known feature, or certainly one I hear less about, is Moto's [server mode.](https://docs.getmoto.org/en/latest/docs/server_mode.html) I believe server mode makes Moto not only a great testing tool, but also a great prototyping tool for serverless applications and a great alternative to SAM CLI, which I found to be clunky.

The main purpose of server mode is to allow developers to use Moto with any of the official AWS SDKs, not just Python. However, I have found server mode to be useful when looking to rapidly prototype and validate ideas locally without having to spend time actually provisioning resources.

With server mode, you can rapidly iterate on a local API instance, sharing concepts with stakeholders early on to receive feedback. I find this perfect for fast-paced working environments where time to deliver is critical; allowing developers to focus on what really matters without the overhead of deployments.

The great thing about server mode, is that mocked AWS services are fully decoupled from your locally hosted API instance, meaning the data is persisted across API restarts, eliminating the need to re-seed upon restart, unless of course your data model changes.

## A Practical Example

---

{{< newsletter >}}

---
