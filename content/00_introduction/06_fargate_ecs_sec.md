---
title: "AWS Fargate and ECS Security"
draft: false
weight: 06
---

**AWS Fargate and ECS** allow you to deploy containerized workloads quickly. Those services are so convenient that many people leave them unattended, risking exposure to vulnerabilities inside their containers that can exfiltrate secrets, compromise business data, impact performance, and increase their AWS costs.

For example, think of some **credentials** mistakenly included in an image, later deployed on Fargate. They will be exposed to anyone with access to the image (think on the repository), or to the Fargate service.

Consider a **known vulnerability**. Imagine you deploy a Fargate task to manage your API, and that it uses an old HTTP library version that ignores the setting to limit a request size. That could be catastrophic! Say you expect requests no bigger than 1MB, but a malicious actor exploits this vulnerability to send requests as big as 80GB. This will absolutely take a toll in your AWS bill, and might cause your service to throttle.

Those are serious threats.

You might be reluctant to implement further security checks, thinking they will take away the same flexibility you were looking for when you decided to use Fargate on ECS.
