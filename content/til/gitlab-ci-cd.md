+++
title = "GitLab CI/CD basics"
date = 2022-10-02
updated = 2022-10-02
type = "post"
description = "Some basic hidden workings of GitLab CI/CD I discovered recently"
in_search_index = true
[taxonomies]
TIL-Tags = ["gitlab", "CI/CD"]
+++

1. GitLab runs all CI/CD jobs within a Docker container. Ruby Docker image is
   used as default  
a. We can overwrite this default image by specifiying a image name globally (in
the `.gitlab-ci.yml` file).  
    ```yaml
    image: <default_image>

    test_project:
        script:
            - pytest
    ```  
    b. It is also allowed to use different docker images for different jobs.
    Each job will can have a key called `image`.  
    ```yaml
    image: <custom_image>

    # test python backend with pytest. So python is needed
    test_backend:
        image: python:3.9-slim-buster
        before_script:
            - pip install -r requirements.txt
        script:
            - pytest

    # test js frontend with npm. So node is needed
    test_frontend:
        image: node:alpine
        before_script:
            - npm install
        script:
            - npm test

    # test the integration. Some custom environment is needed
    test_integration:
        script:
            - make test # run within contianer of <custom_image> container
    ```
2. We can import jobs from other files by the URL of GitLab file like below  
`lint.yml` (`https://gitlab.com/<org-name>/<repo-name>/-/blob/master/lint.yml`):
    ```yaml
    lint_python:
        stage: lint
        image: <image_name>
        services:
            - postgres:12.4-alpine
        variables:
            POSTGRES_DB: <db_name>
            POSTGRES_USER: <user_name>
            POSTGRES_PASSWORD: <password>
        before_script:
            - pip install --upgrade pip
            - pip3 install tox flake8
        script:
            - flake8 --statistics
    ```
    `.gitlab-ci.yml`:
    ```yaml
    # refer to the above lint.yml
    include:
    - project: '<org-name>/<repo-name>/'
        ref: master
        file: 'lint.yml'

    # Specify stages
    stages:
        - lint
    ```
    
3. We can override the imported Jobs like below:

    `.gitlab-ci.yml`:
    ```yaml
    # refer to the above lint.yml
    include:
    - project: '<org-name>/<repo-name>/ci-cd'
        ref: master
        file: '/templates/jobs/lint.yml'

    # Specify stages
    stages:
        - lint

    # Overwrite imported lint_python job from lint.yml
    lint_python:
        allow_failure: true
        script:
            - pip3 install black
            - black .
    ```  
4. Since all `Jobs` run inside containers, when we need to execute docker
   commands (within a build `Job` for example), we will need docker to be available inside the Job's container. So we have to use [docker in
   docker](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html).  

    a. A Service container is an additioanl container that starts at the same time as the `Job` container. Most common usecase is to run a database container.  
    b. The `services` attribute will make sure that the Service container and the Job container will be on the same network and can talk to each other.
    ```yaml
    build_image:
        stage: build
        image: docker:20.10.16 # for making the docker client available within Job's docker container
        services:
            - docker:20.10.16-dind # for making the docker daemon available within Job's docker container
            entrypoint: ["dockerd-entrypoint.sh", "--tls=false"] # 
        before_script:
            - docker login -u $REGISTRY_USER -p $REGISTRY_PASS
        script:
            - docker build -t $IMAGE_NAME:$IMAGE_TAG .
            - docker push $IMAGE_NAME:$IMAGE_TAG
    ```  
    **PS:** With `dind` as service, we need to configure TLS, else we can disable it like above. [Reference](https://about.gitlab.com/blog/2019/07/31/docker-in-docker-with-docker-19-dot-03/)
5. Add a (`.`) in front of a job name and it will be skipped from execution. It
   is like commenting.
    ```yaml
    .test_experiment: # skipped from execution
        script:
            - make test
    
    test_production:
        script:
            - pytest
    ```  
6. Apart from secret variables, we can add custom variables in the
   `.gitlab-ci.yml` file to avoid repitition. They can either be defined per
   `Job` or `globally`. Just like secret variables, they are also referred with
   the help of `$variable`.  




References: 
1. [GitLab CI/CD course by Tech with Nana](https://www.youtube.com/watch?v=qP8kir2GUgo)