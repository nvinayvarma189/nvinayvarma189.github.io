+++
title = "ML System Design Exercise"
date = 2023-05-19
updated = 2023-05-19
type = "post"
description = "Trying to document some brainstorming I've done while trying to design an ML platform."
in_search_index = true
[taxonomies]
Blog-Tags = ["system-design"]
+++

Back in October 2022, my manager ([Amresh](https://github.com/ltbringer)) at that time gave me a problem statement to design a platform for ML engineers to carry out the tasks in an ML model lifecycle. The purpose was to help me get better at system design.

Looking back at it, the homework I did and the material I read for this exercise seemed quite fruitful and is still interesting to me. This prompted me to document the way I attacked this problem statement as a future reference for myself.

# Problem Statement

*Design a system to train, deploy, serve, evaluate ML models.*

1. We receive about 100k model training requests every day.
2. Assume labelled data exists but you have to design abstractions around it.
3. We want to deploy quickly most of the time but attempts to deploy repeatedly should come with a cost (i.e. careless attempts to deploy should be penalized).
4. Some data scientists can be clueless about testing and can try load-testing the production API. How to address this?

*you want to think of other problems like 4 too, this is intentionally a small list*

# Questions

Below is the list of questions I asked to further understand the scope of the platform. This section is written in a question-answer fashion, where the question was asked by me and the answer was given by my manager.

### Data 

1. **Question:** How do the users provide data to me? Do they upload it to some storage location from which I have to read from? or do I have to provide them an interface to upload the datasets to a storage location of my choice (which is abstracted to them)?  
    **Answer:** They have an API. Imagine an interface where they can upload a dataset as a file or a URL.

2. **Question:** Can a user upload multiple versions of the same dataset?  
    **Answer:** Not for a single training request. Dataset reuse is out of scope, but think about retries if anything fails, you don’t want the user to re-upload the dataset.

3. **Question:** What is the largest size of data that I can expect?  
    **Answer:** 100 kb avg. 10GB max

4. **Question:** What kind of datasets should I expect? Can it be of any modality (image/text/audio/video/tabular, etc) or of a single modal only? Can I expect a single dataset comprising multiple modalities together?  
    **Answer:** Any of these should be possible for this platform.

5. **Question:** Should they be able to write pre-processors and post-processors?  
    **Answer:** Out of scope

### Training

1. **Question:** Should the users be allowed to customize how to train the model?  
    **Answer:** Definitely, but this is an abstraction. How part of the training is assumed to exist.

2. **Question:** Should the users be provided with the ability to tune the hyperparameters before each training request?  
    **Answer:** optional but bonus points for solving this

3. **Question:** What type of models should we support? Is it just one type of task like text classification or can it be anything (image classification, linear regression, K-means clustering, video segmentation, image generation, GAN, etc)  
    **Answer:** This platform should be vendor and model agnostic. Think of it this way: You have N types of models. Their input and output vary. How do you still wrap this complexity under a single predict API?

4. **Question:** Should the users be able to choose what type of hardware they need for their training task?  
    **Answer:** The users shouldn’t need to know this. Assume all training jobs require GPUs.

5. **Question:** Should the users be allowed to write their custom training code?  
    **Answer:** Assume they already have and it's in a separate service.

### Evaluation

1. **Question:** Should they be allowed to edit the loss functions? what about customizing the evaluation script?  
    **Answer:** Yes

2. **Question:** Should they be able to compare the evaluation results of multiple models that they have trained?  
    **Answer:** Optional but bonus points for solving this

3. **Question**: Should some triggers automatically deploy the model when the evaluation results cross a certain threshold?  
    **Answer:** Out of Scope.

### Serving

1. **Question:** What type of machines are they looking to deploy it on? Is it an on-cloud or some edge device or some on-prem server?  
    **Answer:** On Cloud

2. **Question:** Do they require only HTTP serving? Do they need additional servings like gRPC?  
    **Answer:** HTTP for now

3. **Question:** Do they need support for auto-scaling enabled?  
    **Answer:** They shouldn’t need to know what this is. They just want a model trained but your business will crash without it.

4. **Question**: What kind of service and ML metrics do they want to see?  
    **Answer:** ML metric depends on the model. Service metrics like RQS and logs should be enabled.

5. **Question:** What is the maximum latency they are expecting?  
    **Answer:** 100 ms

6. **Question:** How much can the load of requests per model vary?  
    **Answer:** Each model gets a uniform amount of load

7. **Question:** Should the users have the flexibility of writing their own docker image? or can I use a base image of my own?  
    **Answer:** Too much work exposed by letting them know docker etc.

8. **Question:** Should they be allowed to apply some inference optimizations like half-precision, quantization, and pruning?  
    **Answer:** Out of scope

9. **Question:** Should they require batch processing or is every model supposed to be giving predictions in real-time? **Answer:** Real-time is required

10. **Question:** Do they require a staging environment?  
    **Answer:** Optional

11. **Question:** Do they require deployment strategies like A/B testing or canary deployment?  
    **Answer:** Yes

### Misc

1. **Question:** Do my users know how to use a notebook?  
    **Answer:** Yes


### Retrospective Questions
While I was compiling these questions, I realized I did not bother enough about the user persona back then:  
1. Who are they?
2. What is their technical background?
3. What is and is not their expertise in the whole Software and ML development lifecycle?
4. What problems are they facing currently?
5. What technologies are they most comfortable with?
6. What level of control/abstraction would fit them?

# Solutioning

I've started studying the design of existing ML tools like MLFlow, Sematic, TrueFoundry, AWS Sagemaker, Label Studio ML Backend, Kubeflow, etc. Below are a few rough sketches of how I thought each component would look like and how they integrate.

## Frontend

The primary interface to interact with the ML platform would be a CLI. There will be CLI commands via which a user can carry out the following:
1. User authentication
2. Submit a training/evaluation request with custom params
3. List previous training/evaluation requests
4. View and compare metrics from different runs
5. Inspect training/evaluation progress of an active run

There can also exist a UI via which the user can carry out all these actions.

## Data

1. The datasets can be of any type and be present in any form.
2. It is expected that the users have the data dump with them via ETL pipelines from data sources like 
3. How the data gets loaded (data loaders, streamed, curled via a URL, inbuilt datasets from pytorch or hugging face, etc) and pre-processed will be written by the user.
4. DVC can be employed to track the data versions via git itself.
5. The file path of the data will be an ENV variable. When executing locally, the variable can be set to `/root/user/local/path`. When executing remotely, the data can be uploaded to a storage location which will be mounted to the machine where training/evaluation is conducted. Hence the variable can be set to `/mnt/cloud/storage/location`.

## Development Setup

### Code Structure

The users can write an ML model with the following structure. They are free to use any library or any model or any dependencies. They just have to have a model created in this format.

```python
class MyModel():
	def __init__():

	def train():

	def test():

	def predict(): # must return a json object
```
### Parameterization
Users can use special variables inside their code that our library provides. These values will have a default value. They can also override them when creating a training request. Let's call our library SDK `mllib` for now.

```python

# model.py

from mllib import sentry_plugin
import sklearn # global dependency

sentry_plugin.enable()

class MyModel():
    def __init__(self):
        self.learning_rate = mllib.Param(0.1)
        self.train_data_path = mllib.Param("/path/to/train/dataset")
        self.evaluation_data_path = mllib.Param("/path/to/evaluation/dataset")

    def train():
        import numpy as np # local dependency for training step alone
        import optuna

    def evaluate():
        import torch # local dependency for evaluation step alone


    def predict(): # must return a json object
        import pands as pd
        import tf

        return prediction
```

### Training/Evaluation Requests

The user can make a training/evaluation request via the CLI in this fashion:

```bash
mllib model.py --train --learning_rate=0.001 --local
```
This will use the ENV variables defined in `.env.local` and run the code defined in `train` function locally with a learning_rate of `0.001`. 

Likewise for running **evaluation** as well:

```bash
mllib model.py --evaluate --learning_rate=0.001 --local
```
Once the user is satisfied that the model is working properly and they have made some verifications to gain confidence, the training can be switched to the cloud via:

```bash
mllib model.py --train --learning_rate=0.001 --remote
```

Upon the execution of this command, the training code is packaged, and submitted to a pool of GPU-enabled machines where code inside the **train** and **evaluate**** functions is executed.

### Deployment
Executing the below command would take the predict function and serves via [firefly-python](https://firefly-python.readthedocs.io/en/latest/) in a serverless fashion.

```bash
mllib model.py --serve
```

## Monitoring

### System Monitoring

1. The sentry plugin can be enabled to capture exceptions.
2. Prometheus-Loki-Grafana to capture logs and display them to the user. Additionally, the user can customize the Grafana dashboard with custom PromQL queries to their liking.

### ML Model Monitoring

1. All evaluation results made via `mllib model.py --evaluate` will be available for visualizing in the UI.
2. For continuous monitoring post-deployment, the evaluate function can be run on a schedule where the dataset will be sourced from annotation tools like Label Studio.

## Missing Pieces

I did not think through the following aspects when designing the solution. Mostly they revolve around reliability and failure resolution

1. If a training request has failed, what should the resolution flow look like?  
    a. What if it failed because of a runtime error? what would be different if it was a network/systems error?
2. Once the training is done, what happens to the artifacts (model checkpoints, loss graphs, etc)? Where do they get stored and how do they get accessed?
3. How exactly do we handle the queue of training and evaluation requests?
4. How to seamlessly shift between local and remote execution?
5. What if the user wants to use their own cloud resources due to data privacy concerns?
6. How to package the dependencies and container creation without getting in the user's way? Especially because the users do not have a good understanding of docker.
7. What is the right way to handle data movement between the local execution and the cloud execution?
8. How is the fleet of GPU-enabled systems managed and receive training/evaluation requests?
9. ...some more?

# Final Note

I've had great fun and learning working on this exercise. There are just too many moving components across the product, service, and infrastructure level and I was forced to think about a lot of aspects that lie outside my expertise. I realized I liked designing not just for usability but for also the user's delight.

I might have a go at it again, this time with better research, mockups, and maybe even a PoC :eyes: