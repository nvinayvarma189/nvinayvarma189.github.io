+++
title = "GitLab CI/CD Intermediate"
date = 2023-02-25
updated = 2023-03-20
type = "post"
description = "Some basic to intermediate level concepts of GitLab CI/CD"
in_search_index = true
[taxonomies]
TIL-Tags = ["gitlab", "CI/CD"]
+++

I've been working on a CI job that would help me apply a mandate for a Git workflow for all dev teams at my company. During this process, I discovered a lot of functionalities that GitLab CI/CD offers that you can use to do cool things. I also got to correct some misconceptions I had.

### Misconceptions

These made me feel dumb but that's okay.

1. I realised that you don't have to use GitLab CI/CD when your code is on GitLab. You can use CI tools like Circle CI, Jenkins etc. GitLab itself has links on how to use these external plugins. [Reference](https://docs.gitlab.com/ee/user/project/integrations/index.html#available-integrations).  
a. You can also use GitLab CI/CD from GitHub. [Reference](https://docs.gitlab.com/ee/ci/ci_cd_for_external_repos/github_integration.html).

2. For the longest time, I was under the assumption that GitLab runner is a machine provided by GitLab where we can run our CI/CD pipelines. I realised, GitLab runner is a software written in Go. You can take a server from your private cloud and install the GitLab runner software on it and get the server registered on your GitLab instance (all tiers).  
a. A GitLab CI/CD pipeline is run in a CI server that we (a company if you work for one) own by the GitLab runner software. The GitLab runner software running on our CI server receives a hook from the GitLab project when there is a new merge request (this setting on what will trigger a CI pipeline must be configured on the GitLab project repository). Then the GitLab runner software clones the repository files into the CI server and executes the CI/CD jobs written in the `.gitlab-ci.yml` file.

### New Learnings

1. Adding a `.gitlab-ci.yml` file would be enough to trigger pipelines for every merge request and push it into the Gitlab instance. Inferred this from: "When you add a `.gitlab-ci.yml file` to your repository, GitLab detects it and an application called GitLab Runner runs the scripts defined in the jobs." [Reference](https://docs.gitlab.com/ee/ci/yaml/gitlab_ci_yaml.html)

2. **Branch Pipeline V/S Merge Request Pipeline:** You can configure your pipeline to run every time you commit changes to a branch. This type of pipeline that gets created by a `git push` is called a **Branch pipeline**. You can also configure your pipeline to run when there is a new merge request. This type of pipeline is called a **Merge Request pipeline**. It is important to note that a merge request pipeline is run every time you make changes to the source branch for a merge request.

3. By default, a pipeline is triggered for every git push and every MR. If you have an open MR and you pushed to the source branch of it, you will trigger two pipelines (one triggered by a push event and another by an updated merge request event). If you don't want a pipeline to be triggered for every push, you can do it like this

	```yaml
	lint-job:
	image: ubuntu
	stage: lint
	script:
		- echo "My milkshake brings all the boys to the yard"
		- some_linting_script.sh
	rules:
		- if: $CI_PIPELINE_SOURCE == "merge_request_event" # only runs when there is a merge request.
		exists:
			- '**/*.go' # only runs if there are Go files present
	```

	You can also configure a job to run on only the creation of a merge request like below

	```yaml
	script:
		- echo "I'm in love with the shape of you!"
	only:
		- merge_requests
	```
	You can also disable triggering pipelines for a git push with git push options like `git push -o ci.skip` but every push into an open MR would still re-trigger the pipelines even if you had the above condition. [Reference](https://docs.gitlab.com/ee/user/project/push_options.html)

4. You can configure it to require a successful pipeline for a merge (between two branches) to happen. However, this setting can conflict with configs like only/expect, rules that you write in your `.gitlab-ci.yml` file that do not trigger new pipelines.  
**Example:** Let's say you added a condition that your pipeline should run only when merge requests are created to the master branch like below:
	```yaml
	dummy-job:
	stage: dummy_stage
	rules:
		- if: $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master"
	script:
		- echo "I like to move it move it"
	```
	and you configured that a pipeline must succeed for an MR to be labeled as ready to merge, then you would not be able to merge MRs to any other branches. [Reference](https://docs.gitlab.com/ee/user/project/merge_requests/merge_when_pipeline_succeeds.html#require-a-successful-pipeline-for-merge)

5. When you make an MR, the `gitlab-ci.yml` present in the source branch is used for running the pipeline. When a pipeline is triggered by an MR (Merge Request pipeline), you will see this message on it: "This pipeline ran on the contents of this merge request's source branch, not the target branch".

6. When there are multiple Merge Requests where the source branch of one MR is the same as the target branch of another MR, GitLab automatically updates them when one of them is merged. [Reference](https://docs.gitlab.com/ee/user/project/merge_requests/#update-merge-requests-when-target-branch-merges)

7. If you want the entire pipeline (all the jobs present in the `.gitlab-ci.yml` file) then you can specify it like this:

	```yaml
	workflow:
	rules:
		- if: $CI_PIPELINE_SOURCE == 'merge_request_event'

	```
	or
	```yaml
	workflow:
	only:
		- merge_requests
	```

8. If you want to write a multiline script for a job, you can do it like this
	```yaml
	dummy-job:
	stage: dummy_stage
	rules:
		- if: $CI_PIPELINE_SOURCE == "merge_request_event"
	script:
		- |
		  if [[ "SP Balasubramanyam" == "Greatest Singer"]]
		    then
			echo "Allantha dhoorala aa thaaraka..."
		  else
			echo "Whatever";
		  fi
	```

9. You are allowed to use regex for file matching in the `exists` field. However, if you want to do regex matches in the `rules` field itself (when you want to run some job based on the branch, for example)
	```yaml
	dummy-job:
	stage: dummy_stage
	rules:
		- if: $CI_PIPELINE_SOURCE == "merge_request_event" && $CI_MERGE_REQUEST_SOURCE_BRANCH_NAME !~ /^hotfix-/
	script:
		- echo "Merge request is coming from a hotfix branch"
	```
	The above dummy-job will run as part of your pipeline only when there is a merge request coming from a hotfix branch.

10. You can lint check your `.gitlab.yml` file on the pipelines page. You can also try simulating a pipeline to figure out issues with needs/rules that would only pop up in the run time. [Reference](https://docs.gitlab.com/ee/ci/lint.html).  
a. If you are using VSCode, you can use this extension to have the yaml file CI linted right there [Reference](https://docs.gitlab.com/ee/ci/lint.html#simulate-a-pipeline)

11. If you want to quickly try out editing CI variables, you can use it like this: `git push -o ci.variable="MAX_RETRIES=10" -o ci.variable="MAX_TIME=600"`. Once you are satisfied with the CI variables, then you actually edit them. This way you can avoid making several commits while you are experimenting with CI variables.

12. Git push options for Merge Requests. You can configure the Merge Request behavior when pushing your changes. For example: you want to create a new merge request, and target a branch named `my-target-branch`
	```yaml
	git push -o merge_request.create -o merge_request.target=my-target-branch
	```

13. GitLab CI/CD jobs produce artifacts. These artifacts can then be accessed via the GitLab API. These artifacts also have an expiry date that you specify in the job config itself
	```yaml
	build_and_push_image:
	stage: build
	allow_failure: false
	script:
		- bash build_image.sh
	artifacts:
		paths:
		- env.file
		expire_in: 1 week
	```


14. Mandating that a review is required for an MR to be merged is a GitLab premium feature. You can also setup a check that requires approval before merging code that causes test coverage to decline.


15. This is how we can setup IaC with k8s, GitLab CI/CD, AWS, and helm:
	1. Consider that we have multiple GitLab repos to host application code (usually the code of each microservice) and one GitLab repo for maintaining all the helm charts for their corresponding deployments to k8s (Let's call this state repo)
	2. A developer merges an MR to the staging branch on their application repo.
	3. As part of the CI steps (scripts to run tests and lint checks), a docker image is created and is pushed to AWS ECR if the tests run successfully.
	4. Usually, even these CI steps are imported from a common GitLab repo that hosts all the common GitLab job configurations (let's call this CI-CD GitLab repo).
	5. Then there will be a job on the application GitLab repo to update the docker image tag in the values.yml file (or the k8s cluster-specific overrides file staging.yml) of that service in the state repo via an MR.
	6. Once the MR is merged, there will be a job on the state repo which shall run only when there is a change to the staging.yml in the overrides folder of that service's helm chart. This job will run the script to apply the helm changes in that cluster.
	7. A new version of the application is now rolled out to the staging cluster. Likewise for other clusters as well.

![image](/images/til/gitlab-ci-cd-2/20230315_192634.jpg)

<!--- {{ resize_image(path="/images/gitlab-ci-cd-2/20230315_192634.jpg", width=, height=150, op="scale") }} -->

