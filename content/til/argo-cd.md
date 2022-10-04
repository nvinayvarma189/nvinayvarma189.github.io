+++
title = "Notes on ArgoCD"
date = 2022-10-04
updated = 2022-10-04
type = "post"
description = "Beginners introduction to Argo CD and its advantages over alternatives"
in_search_index = true
[taxonomies]
TIL-Tags = ["CI/CD"]
+++

The below is the notes I've taken while understanding what argoCD is about. I will refer to CD providers like GitLab and Jenkins as alternatives.

1. With alternatives, we would need to setup tools like AWS CLI, kubectl, helm, etc inside each CD job of each project.  
a. We will end up providing k8s access to all the different client code bases. This can be a security challenge  
b. These tools get installed and need to be authenticated for every new release.
2. The alternatives won't give us the deployment status of our apps once the `kubectl apply` is done. It is possible that we have a successful pipeline run (kubectl apply is executed ) but a failed deployment (pods failing to startup).  
a. Though we can have liveness and startup checks to let us know if deployment failed, argoCD gives us that capability by default.
3. argoCd also encourages that we follow the principles of GitOps which ensures we visbility, security, speed, and single source of truth.
4. Since argoCD is deployed within k8s cluster itself:  
a. We don't need to have cluster credentials outside of k8s  
b. no need to give external cluster access to non human users  
c. argoCd is able to do this as it is deployed within a k8s cluster itself. It is an extension to k8s.
5. With argoCD we get an UI to visualise deployment status of any app.
6. Manual rollbacks are also super simple. We will have the history of all the previous version of deployment. We can select a version and click on rollback.

## Reference
1. [ArgoCD Tutorial for Beginners](https://www.youtube.com/watch?v=MeU5_k9ssrs)