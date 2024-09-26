# Mock

## Purpose

The `mock` processor connects to the Gecholog LLM Gateway to mock and replicate your LLM API responses. It works like this:

-  Make a regular LLM API request to any `gecholog` router, for example `/service/standard/`
-  The `mock` custom processor will record the response payload and response headers
-  Send as many requests as you want to `/mock/service/standard/` to get the same response over and over again

The `mock` processor will randomize the response time to resemble an LLM API. You can change this behavior via the `LAMBDA` environment variable.

## Building

Before you proceed, build the [gecholog](https://github.com/direktoren/gecholog) container.

From this directory, run

  export NATS_TOKEN=changeme
  export GUI_SECRET=changeme
  export AISERVICE_API_BASE=https://your.openai.azure.com/
  docker compose up -d

## Using

```sh
export AISERVICE_API_KEY=your_api_key
export DEPLOYMENT=your_deployment
```

Send the request to the `/service/standard/` router:

```sh
curl -sS -H "api-key: $AISERVICE_API_KEY" -H "Content-Type: application/json" -X POST -d '{
  "messages": [
    {
      "role": "system",
      "content": "Assistant is a large language model trained by OpenAI."
    },
    {
      "role": "user",
      "content": "Who were the founders of Microsoft?"
    }
  ],
  "max_tokens": 15
}' "http://localhost:5380/service/standard/openai/deployments/$DEPLOYMENT/chat/completions?api-version=2023-05-15"
```

Now try to make your requests to the mock router `/mock/service/standard/`:

```sh
curl -sS -H "Content-Type: application/json" -X POST -d '{
  "messages": [
    {
      "role": "system",
      "content": "Assistant is a large language model trained by OpenAI."
    },
    {
      "role": "user",
      "content": "Who were the founders of Microsoft?"
    }
  ],
  "max_tokens": 15
}' "http://localhost:5380/mock/service/standard/openai/deployments/any/chat/completions?api-version=2023-05-15"
```

And you should receive the same response back, without Gecholog forwarding your second request.

### Check the logs

Use the [Log Lister](http://localhost:8080/logs) to inspect the traffic

### How it works

`mock` will store the last response for each router. 

```sh
request1 to /service/standard/ returns answer1
request2 to /service/standard/ returns answer2
request3 to /service/standard/ returns answer3
request4 to /mock/service/standard/ returns answer3
```

`mock` will  separate the responses for each router.
 
```sh
request1 to /service/standard/ returns answer1
request2 to /service/capped/ returns answer2
request3 to /mock/service/standard/ returns answer1
request4 to /mock/service/capped/ returns answer2
```

### Change response time

`mock` will randomize response time using the [Exponential distribution](https://en.wikipedia.org/wiki/Exponential_distribution) with environment variable `LAMBDA`. Set `LAMBDA=0` to disable the latency which is the default value. The `docker-compose.yml` uses `LAMBDA=0.2` which gives a mean response time of 500 ms.
