+++
title = 'Basics about DevOps (study notes)'
date = 2020-06-14T19:49:00-04:00
comments = true
math = false
+++

Notes taken from a [DevOps course](https://cloudacademy.com/course/introduction-to-devops) by Ben Lambert.

## What is DevOps?
There isn't consensus on a single definition on DevOps. But the lack of any standard definition doesn't detract from its value. Because DevOps is a philosophy. It's the philosophy of the efficient development, deployment, and operation of the highest quality software possible.

## Three Tenets:
* Culture
  * Culture requires the removal of silos (Dev/Test/Security/Ops) in favor of high collaboration, e.g. cross-functional teams, teams that are responsible for the complete life cycle of a product, where everyone works towards a common set of goals.
  * Blameless culture, one that understands that failure is going to happen and it's important to prevent a similar failure from happening in the future.
* Automation
  * Key point is reducing human intervention to a minimum.
  * There are a lot of places to use automation in the pipeline of development, deployment and operations:
    - Development --> CI
    - Deployment --> CD
    - Operations --> IaC
  * Benefit is it allows the whole software engineering pipeline to move faster, scale better, and produce higher-quality software.
* Metrics
  * Basic metrics that help to quantify the success of DevOps practices: frequency of deployments, mean time to recovery (MTTR), mean time to discovery (MTTD), system availability, service performance, customer complaints, lead time.

## Business values of DevoOps
* improve lead time
* improve system stability (Git, Jenkins, Spinnaker, ...)
* reduce operational costs
