+++
title = "Platform Engineering"
date = 2023-01-05
updated = 2023-01-05
type = "post"
description = "Notes on what platform engineering is about and how it differs from Devops/SRE/SDE"
in_search_index = true
[taxonomies]
TIL-Tags = ["platform-engineering"]
+++

Over the last few months, I started working as an ML Platform Engineer. I have, since then, started learning about what Platform Engineering(PE) is about. The following is a loose set of points that I have been gathering about PE.

# What is a Platform?
It is a set of tools or systems that you can use to build more things on top of.

## Division of Platform teams
By the above definition of a Platform, the scope of Engineering is huge. Some companies typically break it down into three teams.

1. **SRE team** - focuses on monitoring, observability, and reliability of systems.  
    Tasks like:  
    a. Setting up Monitoring platforms (Prometheus, Grafana), Messaging platforms (Kafka, Rabbit MQ, Amazon SQS), and Data platforms (Hadoop, Apache Spark), etc. [Reference](https://youtu.be/0uuOJ1gzcyE?t=101)  
    b. Working on reaching SLAs, SLOs, and SLIs. [Reference](https://cloud.google.com/blog/products/devops-sre/sre-fundamentals-slis-slas-and-slos)
2. **DevX team** - Providing developers with the tools, services, and automated workflows they need to do their job.  
    Tasks like:  
    a. Creating alerts, and providing a framework for building products.  
    b. Creating pipelines, documentation, and setting up standards across the company to help application development teams move quicker.
3. **Cloud Engineering team** - They deal with the nuts and bolts of cloud infrastructure.  
    Tasks Like:  
    a. Setting up a VPC and configuring cloud (example: AWS) resources (compute instances, Kubernetes clusters). Cost management, billing, security, etc.  
    b. Policies, Managing compliance, and Cost Monitoring.

In the company I work for, we have a single Platform team that takes care of SRE & DevX. An Infrastructure team that takes care of Cloud Engineering.

Since our end products are ML-powered, within the Platform team, we have further divisions of Engineering Platform (focuses on improving the DevX for core services and SRE responsibilities) and Machine Learning Platform (focuses on improving the ML model development lifecycles).  

# IaaS v/s PaaS v/s SaaS
Let's take a Cookie factory analogy.

**Infra team** sets up the building blocks of the factory. motors, cogs, machines, belts, etc.

**Platform team** uses the building blocks made by the Infra team to create things like conveyer belts, packaging machines, etc.

**Application developers** use the tools developed by the Platform team to convert the cookie (that they baked) into a selling product that will reach the customers.

Examples of IaaS: Amazon Web Services, Google Cloud Platform, and Microsoft Azure  
Examples of PaaS: Heroku and Railway  
Examples of SaaS: Dukaan (It lets you create, customize, and launch (service) your website (software) to sell your products)


## Difference between an SDE and a Platform Engineer?
1. An SDE writes code for applications that are used by your clients or customers. They are satisfying the needs of the customer.
2. A Platform Engineer writes code for infrastructure to the needs of the application developers (SDE). SDEs are customers of Platform Engineers.
3. One of the responsibilities of a PE is to provide automation of recurring tasks of an SDE thereby reducing the go-to-market time.

## Difference between ML Engineers and ML Platform Engineers?
1. Platform MLEs should help in triggering performance drop alerts so that Task MLEs can act on them.
2. They assist with Monitoring and Observability too. Platform MLEs also take care of creating features.
3. Platform MLEs do not get to change anything around the model, its inputs, or outputs— but they’re responsible for identifying when and how any of them are broken.

For more information check this amazing [reference](https://www.shreya-shankar.com/phd-year-one/).


## Misc
**Who should write Dockerfile?**

1. If Devs write it, DevOps will have surprise errors from production. Devs will seek help from DevOps. Devs usually do not focus on security or any other policies.  
2. If DevOps write it, Devs will complain that they have to wait and it causes delays. Imagine having to wait on a DevOps person just to add an ENV variable.  
3. A hybrid approach is to have the Devs provide the list files, environment variables, and dependencies they need and DevOps builds another Dockerfile on top of it to have security.  
4. These days Devs are expected to have Docker knowledge. However, for security and other nitty-gritty of Docker, the involvement of SRE is sometimes required. [Reference1](https://stackoverflow.com/questions/63043718/who-should-write-the-dockerfile-sre-or-developer) and 
[Reference2](https://devops.stackexchange.com/questions/12042/who-should-write-docker-files)


# Resources
1. [Platform Engineering Youtube channel](https://www.youtube.com/@PlatformEngineering) and [Platform Engineering Blog](https://platformengineering.org/blog) - Produces quality content. Highly recommended to check their playlists
2. [What is Platform Engineering](https://platformengineering.org/blog/what-is-platform-engineering)