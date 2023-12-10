+++
title = "Live reload of a webserver inside a docker container"
date = 2023-12-05
updated = 2023-12-05
type = "post"
description = "A neat way to go ahead when you have a broken environment (not your home)"
in_search_index = true
[taxonomies]
TIL-Tags = ["docker"]
+++

Environemnt setup errors are always annoying. They suck time better than mousquitoes suck blood. I had a clean environment inside a docker image. The docker file looked like this:


```docker
FROM python:3.10

# install necessary system level packages

WORKDIR /my-server
COPY ./requirements.txt /my-server/requirements.txt

# install python depndencies
RUN pip install --no-cache-dir --upgrade -r /my-server/requirements.txt

COPY ./src /my-server/src
COPY ./tools /my-server/tools
COPY ./application.py /my-server/application.py

EXPOSE 5000

CMD ["uvicorn", "application:application", "--host", "0.0.0.0", "--port", "5000"]
```

and a docker-compose file like this

```yaml
version: "3.7"

services:
  fastapi:
    build: .
    image: my-server-fastapi
    ports:
      - 5000:5000
    container_name: fastapi
    environment:
      - ENVIRONMENT=${ENVIRONMENT}
  worker:
    build: .
    image: my-server-worker
    command: bash tools/run-worker.sh
    container_name: worker
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
```

Since my local env was broken, whenever I had to make a test call, I did `docker-compose up --build`. While this was working, I had to stop and restart the docker containers for ever code change. I needed a way to enable live reload inside the docker containers but I did not want to modify the existing Dockerfle or the docker-compose.yaml file.

I thought attaching the code as a volume to the docker containers to override the code (copied inside at docker image build time) would solve the issue but it did not. The trick was to add `WATCHFILES_FORCE_POLLING=true` as a env variable. 

I added a docker-compose.dev.yaml file like this:

```yaml
version: "3.7"

services:
  fastapi:
    build: .
    image: my-server-fastapi
    ports:
      - 5000:5000
    container_name: fastapi-dev
    working_dir: /my-server
    command: uvicorn application:application --host 0.0.0.0 --reload --port 5000 # hostname had to be 0.0.0.0 for 5000 port to be exposed to the host machine.
    volumes:
      - ./src:/my-server/src
      - ./tools:/my-server/tools
      - ./application.py:/my-server/application.py
    environment:
      - WATCHFILES_FORCE_POLLING=true
      - ENVIRONMENT=${ENVIRONMENT}
```

and I spun up the server like this:
```bash
docker-compose -f docker-compose.dev.yaml up --build
```

This way I could enable live reload inside the docker containers (where I have a working environment) without making adjustments on the original Dockerfile and the docker-compose.yaml file on which deployments depend on.

[Reference](https://stackoverflow.com/questions/69460295/how-to-enable-live-reload-in-a-dockerised-fastapi-application-using-docker-compo/75387355#75387355) about `WATCHFILES_FORCE_POLLING` env variable