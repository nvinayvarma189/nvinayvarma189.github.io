+++
title = "Maintainable DevOps for AWS EC2"
date = 2023-10-29
updated = 2023-10-29
type = "post"
description = "Create and maintain EC2 instances with TF and carry out CI/CD with GH actions and Ansible"
in_search_index = true
[taxonomies]
TIL-Tags = ["devops", "CI/CD"]
+++

Let's say, you have a backend web server that you want to deploy. On AWS, there are offerings like EC2, AWS Beanstalk, AWS ECS, AWS EKS, AWS Fargate, AWS Lambda and AWS AppRunner.

Depending on your requirements, you can make trade-offs between maintenance overhead and the control you need over your deployments. You can learn more about it here. The below is one way to manage your deployments on AWS EC2 instances. This is especially useful when you are in the pre-MVP stage where you expect your app to receive a lot of changes but less traffic.

## Requirements
1. Need to have a staging and a production environment. Ideally, separate EC2 instances for each.
2. Need to have a CI/CD pipeline that can deploy to staging and production environments based on pushes/merges to respective branches from the GitHub repo.
3. Need a way to manage secrets and other configuration variables.

We will explore a set up with GitHub Actions, EC2 machines, Caddy, TerraForm, Ansible, and Docker.

## Creating the EC2 instances
1. Use TerraForm to create the EC2 instances. The reason this is beneficial over manual creation of resources from the AWS console is that you can create more instances with the same configuration (network setting, security groups, elastic IPs). Another advantage is that you can manage the state of all the cloud resources from multiple environments in a single place. You can also easily bring down instances/resources that you do not need.
2. Here is a list of things you might want to do with TerraForm:
    1. Create two new workspaces called production and staging in addition to the default workspace. You should able to create resources in a staging or production environment by switching to the respective workspace.
    2. Configure VPC, security groups, and elastic IPs in such a way that the staging and production instances are isolated from each other. TerraForm allows you to use the workspace name inside the configuration file (by which you can reuse the same configuration file for both staging and production environments)
    3. Add an SSH public key of your's (or some team members') so that you can SSH into the EC2 instances once they are created.
    4. Create an AWS role and attach the necessary policies to the role. Then assign the role to the EC2 instances so that you can use the AWS CLI within the EC2 instances without having to manage the AWS credentials. This way you can also keep all actions that can be performed from the EC2 instances in check.
    5. Create an Elastic IP and assign it to the EC2 instances so that the IP address of the EC2 instances does not change when you stop and start the instances.
    6. Create Route53 records for the EC2 instances and the services running inside them so that your other services can communicate with the EC2 instances via the domain names.
    7. Finally, create an Ansible inventory file. This will be used by Ansible to run the playbooks on the EC2 instances. You can use the TerraForm output variables to populate the inventory file.  

At this point, you should be able to create the EC2 instances and other resources (security_group, aws_key_pair, aws_iam_instance_profile, aws_iam_role, aws_eip, aws_route53_record, local_file (for ansible inventory file)) with the following command:
```bash
terraform workspace select production
terraform apply
```

```bash
terraform workspace select staging
terraform apply
```
But these are just bare EC2 instances. You need to install the necessary software and configure them to run your app. This is where Ansible comes in. But Before that, let's ensure that we have a build and test process in place for your app.

## set up CI with GitHub Actions
1. In the GH repo of your codebase, write a workflow that will get triggered for every push/merge to the staging and production branches.
2. It should build the docker images according to the Dockerfile or the Docker Compose file and push the built images to AWS ECR.
3. If an image is being built from the main branch, tag the image with `latest` tag. If an image is being built from the staging branch, tag the image with `staging-latest`. Basically, we need to differentiate between the images built from staging and production branches.
4. Once the newly built images are pushed to ECR, the EC2 instances should pull the images and run containers from them. There are two ways to do it, either you can SSH into the EC2 instances and run the necessary commands (as part of the GH workflow itself) or you can set up a GH runner inside the EC2 machines (install the GH runner software inside the EC2 machine and register it inside your GH repo) and . The latter is more secure (because, unlike the former, the GH workflow commands run on our own EC2 machine and no GitHub's runner machines won't have access to our AWS EC2 machines) and is the recommended way.
5. Following the latter option will look something like this:

```
deploy-staging:
    runs-on: [self-hosted, staging]
    if: github.ref == 'refs/heads/staging'

    steps:
      - name: pull image
        env:
          ECR_REGISTRY: ${{ needs.build-and-push.outputs.registry }}
          ECR_REPOSITORY: ${{ vars.ECR_REPOSITORY }}
        run: |
          cd ~ && docker pull $ECR_REGISTRY/$ECR_REPOSITORY:${{ env.STAGING_LATEST_TAG }}
      - name: restart containers
        run: |
          cd ~ && docker-compose down && docker-compose up -d
``` 

Note: The run commands are executed inside the staging EC2 machine as we have installed GH runner software inside it and tagged the EC2 instance as staging.

## Configuring the EC2 instances with Ansible

1. Ansible Playbooks lets you automate the task of running basically any set of commands on the EC2 instances. You can use the Ansible inventory file created in the previous step to run the playbooks on the EC2 instances. 
2. Here is a list of things you might want to do with Ansible Playbooks:
    1. As a first step, ensure that the EC2 instances are up to date with the latest security patches. You can use the `apt` module to install the latest updates. If you are using a different OS, you can use the corresponding module (example: `yum` for AWS Linux 2023).
    2. Install the necessary packages for the deployment of your software builds. This includes: docker, docker-compose, git, python3, pip3, nodejs, npm, etc.
    3. Install a reverse proxy like nginx or Caddy. The configuration files of the reverse proxy, just like the production and staging  `.env` files necessary for your backend app should not be pushed to the GH repo containing the config files for TerraForm and Ansible Playbook.
    4. Since you will most likely be using Docker, you can set up a cron service to clean up unused docker images and containers regularly.
    5. Set up a GH runner inside the EC2 instances. This will be used to pull the docker images from AWS ECR and run them.

Now you should be able to configure the EC2 instances with the following command:
```bash
AWS_ACCOUNT_ID=<account_id> AWS_REGION=<region> ansible-playbook --private-key ~/.ssh/id_ed25519 -i ansible/inventories/production ansible/production.yml
```

```bash
AWS_ACCOUNT_ID=<account_id> AWS_REGION=<region> ansible-playbook --private-key ~/.ssh/id_ed25519 -i ansible/inventories/staging ansible/staging.yml
```

Note: For you to run the ansible-playbook commands, you need to have your public keys added to the EC2 machines when they are getting created. You can do this by adding the public keys to the TerraForm configuration files. you also need to set up the AWS ENV variables.

## Final Flow

1. You set up the EC2 machines with TerraForm for the first time.
2. You execute the Ansible playbook commands to configure the EC2 machines. It may fail the first time if you don't have any ECR images to pull and run (if that is the case, you can skip pulling the image as the playbook commands are only run for the first time).
3. You push some changes to the production or staging branch of your Application GH repo and see that an image is built and pushed to ECR. It will also be tagged appropriately.
4. The GH action workflow will trigger the GH Runner inside the EC2 machines pull the image from ECR and run the docker container.

# Short Comings
While the above setup covers 95% of the requirements and needs we had, there are some shortcomings with this approach in terms of convenience and maintenance overhead. Here are some of them:
1. Since we won't be storing the .env files and configurations for reverse proxies, updates to the EC2 machines will happen only when we run the Ansible playbooks from our developer machines. Full CI/CD for the infra repo is not possible without any changes.
2. We need to maintain a slightly different docker-compose file inside the EC2 machines for the deployment to work seamlessly.