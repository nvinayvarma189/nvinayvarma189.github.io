+++
title = "Software Deployment Stratagies"
date = 2022-10-17
updated = 2022-10-17
type = "post"
description = "Notes on Blue Green v/s Canary v/s A/B deployment stratagies"
in_search_index = true
[taxonomies]
TIL-Tags = ["devops"]
+++

1. Blue Green deployment
    1. We have an existent environment where our service is working well (green environment)
    2. We replicate the existing (green) environment into another (blue) environment.
    3. We release the new version of our model into the blue environment
    4. Edit the reverse proxy to start sending requests to our new model version in the blue environment.
    5. Monitor for errors. If it works well, we replace the green environment with the blue environment.
2. Canary deployment
    1. The fact that regardless of the immense level of testing you do in lower environments you will still have some bugs in production
    2. Deployment is done in phases. At each phase, we monitor for errors. If we find errors, moving to the next phase is aborted.
        1. Phase 1: new version of the model receives 10% of the load
        2. Phase 2: new version of the model receives 50% of the load
        3. Phase 3: new version of the model receives 100% of the load
    3. Canary releases let you test the waters before pulling the trigger on a full release.
3. A/B deployment
    1. We have two versions of a model which we are sure that there are no unresolved bugs
    2. But we want to collect feedback on which version is more likable to the users (or) is more beneficial to us.
    3. We usually split the traffic by 50:50

**Note:** Since both Canary and A/B deployment stratagies involves the users experiencing different versions of out app/model, they are often confused. However, they can be differentiated by the intention behind adoptiong the stratagies.

We employ Canary when we are not sure how our new version works in comparison to the existing version of our app/model. Whereas, when we know that both existing and older versions work perfectly fine but don't know which one is liked by the users, we employ A/B testing.