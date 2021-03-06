---
layout: post
title:  "Don’t forget to check your Microservice"
date:   2021-06-01 11:15:41
categories: android
---
# Don’t forget to check your Microservice

Over the past few years, I’ve been doing some deep-diving into the world of microservices. As a product developer, I’ve implemented a huge number of microservices. And everytime I need to be sure that I didn’t forget anything, so I’ve created a checklist for myself. And out of my list I decided to write this short article to mention some important things. So let’s go through each point.

## Is the microservice self-contained?

By self-contained I mean it should be independent as much as possible from any other services. Which means it should have a dedicated database, responsible team and ci/cd pipeline. Let’s go through each of these points.
* Why is it important not to share a database?

If microservices share a database, it means changes could impact on other services. And for any changes you need to know which services use the database, communicate with responsible teams and figure out a plan of rolling out all changes to production. So I think it’s too much work to do. And therefore microservices should communicate through API, queues or streams. You should have only dedicated models that are shared, and how data persisted in microservices should be the responsibility of microservice.
* Why should one dedicated team be responsible for the microservice?

In case of problems in production, the team should check it. And if several teams participate in supporting then they should have synchronization and quick reaction on the problem, and know service-well. With a dedicated team it’s easy to manage problem-solving and share knowledge between teammates. Worth mentioning it doesn’t mean that one team should have only one service, and it doesn’t mean that only one team could make changes, but this team is mostly responsible for the review changes and supporting service.
* Why shouldn’t you forget about the CI/CD pipeline?

For each particular change you should have the ability to check if it doesn’t break anything. For this you should have the ability to check code-style, build your branch, check for potential vulnerabilities, run unit tests, and maybe integration tests, and then do all the steps again after merge. Rollout to the staging environment, and later to production if all checks passed should be automated. This is done by Continuous Integration and Continuous Delivery.

## Is observability provided?

Observability based on three pillars: metrics, logging and distributed tracing. Those things are very important to check the health of the service and find potential problems. Let’s share more details on each of these.
You should put metrics on all API calls, on all downstream calls, and important internal processes. Check that metrics have all appropriate types: gauges, counters and timers. And don’t forget to add aggregating information, like tags, e.g. response based on code, or different types of requests. Out of these you can create dashboards, where you could see the health of the system in one place.
Based on metrics, alerts help you notify about potential problems, e.g. you could have alerts on too low traffic, or too much on one particular request type, or response code with technical problems. Or too many errors in logs. This should trigger an alert for further investigation.
Logging should help you find information on all the processes in the system in one place, particularly ELK is a very popular way to do it. Furthermore, it’s good practise to have one particular identifier throughout the whole system.
Tracing helps you find the route of the request throughout the whole system: how long does it take on a particular service and which methods are invoked.

## Was it covered by comprehensive documentation?

Good documentation helps you avoid misunderstanding between teams for the way of using implemented API. If there is no clear explanation that others teams should spend precious time on understanding how API works. And even within the team it could be a problem to find an answer if you should check code or ask teammates how it works.
Not only the API should be documented, but all the internal processes and external communication. And all the changes in services should be documented [at least!] in release changes. Every change should have reasons and links to the bug-tracking system where more information is provided.
All discussions should be documented even if no solution was provided. Following these rules significantly helps to support and improve the project.

Let’s sum up everything in one checklist:
* Don’t share database
* Dedicated team
* CI/CD configured
* Observability
* Full documentation

In conclusion, I hope those things were very helpful. Don’t forget to create your checklist for all the microservices and check one-by-one if you didn’t omit anything from your sight. Let your service be independent, observable and well-documented.