+++
title = 'Basics about Continuous Delivery (study notes)'
date = 2020-07-05T15:49:00-04:00
comments = true
math = false
+++

Notes taken from a [CD course](https://cloudacademy.com/course/introduction-to-continuous-delivery) by Ben Lambert.

## What is continuous delivery?

Continuous delivery (CD) is a way of building software such that it can be deployed to a specified environment, whenever you want to and deploy only the highest quality versions to production. In particular, you should be able to deploy to production and ideally with one command or a push of a button.

You should be able to select the version of software you want, without worrying it is going to break. Nothing should ever break in a continuous delivery setup. If it does break, it should be the exception and not the rule.

## Process:
0. Continuous delivery server picks up code from CI.
1. It starts by deploying your software to a testing environment that mirrors production.
2. It runs the automated acceptance tests, which ensure that the software requirements have been met and also serve as a set of regression tests.
3. When the acceptance tests are passing, then any non-functional automated tests can be run, e.g. load testing, in-depth security audits.
4. Once the automated tests have passed, downstream teams can deploy the software to a testing environment and perform any required manual tasks such as user acceptance testing.
5. Assuming the software passes all manual tests, then it's ready to deploy to production.

## Benefits:
* Inceased quality
* Improved cycle time
* Better resource management
* Reduced deployment risk
* Better customer feedback loops

## Continous deployment:
Similar to continuous delivery except that, when the software makes it through all of the automated testing and the manual testing phases then it is automatically deployed to production. There is no button or command.

## Coding for CD:
* feature toggles
* Inversion of control
* Defensive coding

## Architecting for CD:
Monolith or microservices? There's no perfect architecture. Even the best thing we have at any moment in time may not be suitable as technology continues to evolve.

## Mutable vs immutable servers?
These servers refer to virtualized instances, and the term immutable does not include changes of memory and log files, but means to apply once the configuration is set and everything is loaded, no other outside changes will be made.
* Snowflake servers - a server that's unique and difficult to reproduce
* Phoenix servers - if a server was to die, it could be reborn
* Mutable servers - servers whose configuration and settings will change over time (updating OS, adjusting firewall rules, etc.)
  - Pros:
    * fantastic tools out there for handling configuration management
    * simple ad hoc commands for things like security patches
    * configuration management scripts under version control
  - Cons:
    * if an upgrade or deployment fails, the server can be left in a broken state
    * not starting with a known working configuration each time we deploy
    * may have to use our configuration management tool to handle things like scaling
    * any change to the OS needs to be tested separately
* Immutable servers - a server whose settings don't change; the server is only ever replaced
  - Pros:
    * server in a known working state for each deployment gives us a higher level of trust in it
    * typically use deployment and scaling features that are available with cloud platform
    * if a deployment fails, it's a matter of using the previous server images
    * OS changes can be tested via testing gates, no additional testing process is required for OS level changes
  - Cons:
    * build times are longer since a base operating system is to be merged with the application and a server image is created based on that
    * any changes in the operating system require a bake and redeploy, which can be more time consuming
    * in addition to using configuration management tools to configure our base image, additional tools to handle the baking and deployment are required

Both options are viable, as long as using phoenix servers.

## Two common deployment methods:
* blue-green deployments
* canary deployments
