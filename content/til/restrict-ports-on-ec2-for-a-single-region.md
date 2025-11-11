+++
title = "Restricting public access to EC2 for a single region"
date = 2024-09-06
updated = 2025-11-11
type = "post"
in_search_index = true
[taxonomies]
TIL-Tags = ["AWS", "EC2", "Networking"]
+++

## Setup:

We had an EC2 instance in a VPC (us-east-1 region) where we hosted a backend server (fastapi app). We had an nextjs app and hosted it on amplify (us-east-1 region). Amplify, due to its serverless nature, cannot be hosted inside our VPC and it doesn't have a fixed IP. Due to this. So we had to open our backend to all public internet.

## Problem:
We were getting malicious requests from people in other regions. Mostly from China. We noticed all such requests are coming from outside the US. This was raising guard durty alerts on AWS and was blocking us in being SOC-2 compliant. We wanted to block all those requests and only serve people in the US as that was our only user base.

So anyone on the internet can access the UI (amplify app) but only the IP addresses from the us-eat-1 region should be able to access the backend server.

## Solution:
1. We knew amplify is in us-east-1 region only. But we can't put all the IP ranges of us-east-1 in the security group of the EC2 machine.
2. So we had to modify the IP tables of the EC2 machine to accept all requests from IP address of us-east-1 and drop everything else. Prior to this we allowed all public traffic to the backend server through security group rules. Anyone outside us-east-1 can still access the UI (amplify app) but not the backend server.
3. There is a cron on the EC2 machine that regularly runs and updates the IP tables of the EC2 machine to flush the IP addresses of us-east-1 app. AWS provides an API that returns a JSON of all the us-east-1 ip addresses.
4. One pitfall is that, when AWS rotates the IP addresses of us-east-1 and amplify happens to request the backend from an newly addedd IP address of us-east-1, then we will have downtime on the app until the iptables are flushed again. But this is a rare case and can be mitigated by adding a delay in the cron job.

Misc:
1. AWS Cloudshell gives you a terminal in a certain region.
This is useful to test if people from us-east-1 are able to access the backend. while people from other regions are not able to access the backend.
2. AWS EC2 Connect gives you a terminal inside the EC2 machine. You can use this and block ssh access from everywhere else.