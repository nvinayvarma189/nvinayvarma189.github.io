+++
title = "Retry, Wait, and Fallback"
date = 2023-11-21
updated = 2023-11-21
type = "post"
description = "A look at how to increase the reliability of your workflows"
in_search_index = true
[taxonomies]
TIL-Tags = ["python"]
+++

When you are building a workflow with a set of steps, you generally would want to make sure that the workflow is reliable. For example, if you are running a workflow and one of the step fails due to a network error, you would want to retry it for a certain number of times with some delays between each retry. If the workflow step fails even after retrying it for a certain number of times, you would want to add a fallback option to a different API.

A good example of this can be demonstrated with using the OpenAI API. Let's say you want to use the [OpenAI API](https://platform.openai.com/docs/api-reference/introduction) to generate a completion to your user's input. You would normally do it like this:

```python
from openai import OpenAI
client = OpenAI()

def get_openai_response(user_input):
    input = {
        "model": "gpt-3.5-turbo",
        "messages": [
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": f"{user_input}"},
        ]
    }
    response = client.chat.completions.create(**input)
    return response

response = get_openai_response("tell me a joke")
print(response['choices'][0]['message']['content'])
```

Now, `get_openai_response` can fail due to a number of reasons. You can run into a APITimeoutError, APIConnectionError, PermissionDeniedError, etc. Full list [here](https://platform.openai.com/docs/guides/error-codes/python-library-error-types). Errors like PermissionDeniedError or BadRequestError are not recoverable without any changes, But if you run into a APITimeoutError, RateLimitError, etc you can retry the request after a certain delay which usually solves the issue (especially true for OAI API as they experience a lot of heavy load).

In Python, you can do this via an excellent library called [tenacity](https://github.com/jd/tenacity).

```python

def log_retry(retry_state):
    if retry_state.outcome.failed:
        exception = retry_state.outcome.exception()
        func_name = retry_state.fn.__name__
        wait_time = retry_state.next_action.sleep
        logger.info(f"An exception of type: {type(exception)} has ocurred. Exception Message: {exception}. Will retry {func_name} after {wait_time} seconds.")

@retry(
    stop=stop_after_attempt(5), # the function will be retried upto 4 more times after the first failure
    wait=wait_exponential(multiplier=60, max=600), # the delay between each retry will increase exponentially with a max of 10 minutes. Sequence is 60, 120, 240, 480, 600 seconds
    retry=retry_if_exception_type(OpenAIError), # trigger a retry for all kinds of openAI errors
    before_sleep=log_retry, # execute the log_retry function before each retry
    reraise=True # reraise the original exception if the function fails even after all the retries. If this is set to False, you will get a RetryError exception that tenacity provides
)
def get_openai_response(user_input):
    # a = 1/0 # this will not trigger a retry as we are only trigger it for exceptions that fall under OpenAIError
    input = {
        "model": "gpt-3.5-turbo",
        "messages": [
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": f"{user_input}"},
        ]
    }
    response = client.chat.completions.create(**input)
    return response
```

The values of `wait_exponential` are just an example. You can tweak them according to your needs. Usually, wait_fixed should be good enough for most cases. But if the API you are calling is expected to be under huge load, you might want to use `wait_exponential` to avoid hitting the API too frequently. Please also take into account of how long can your workflow run.