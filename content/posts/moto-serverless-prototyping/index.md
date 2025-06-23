+++
date = "2025-06-23T21:00:00+01:00"
draft = true
title = "Build Rapid Serverless Prototypes with Moto + Python 🚀"
categories = ['Python', 'Software Engineering', 'moto']
+++
## Introducing Moto
[Moto](https://docs.getmoto.org/en/latest/) is a popular Python library commonly used by developers to mock out calls to AWS services. Moto's functionality and ease of use have made the library the preferred choice for Python engineers seeking to unit test their applications without the overhead of deploying real cloud resources. 

Moto does not provide full coverage of all AWS services and functionality, however the library is in active development, and in general I have found it caters for the most common use cases. It especially shines for serverless stack services such as DynamoDB and S3, at the time of writing having 68% and 72% coverage respectiveley. The full list of coverage details are [available here.](https://github.com/getmoto/moto/blob/master/IMPLEMENTATION_COVERAGE.md)

## Server Mode
While the praise for Moto as a testing library is justified, a lesser known feature, or certainly one I hear less about, is Moto's [server mode.](https://docs.getmoto.org/en/latest/docs/server_mode.html) I believe server mode makes Moto not only a great testing tool, but also a great prototyping tool for serverless applications.

---

{{< newsletter >}}

---
