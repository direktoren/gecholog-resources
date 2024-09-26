# Regex

## Purpose

The `regex` processor connects to the Gecholog LLM Gateway to use regular expressions to extract information from the LLM API response. Fields to extract from and regex patterns to use can be customized. The default behavior is as follows:

- regex attempts to extract TEXT within ```` ```markdown TEXT``` ```` from the response field `choices[0].message.content` response for all routers except the `/json/` router
- It adds a new field to the response indicating if match was successful and the extracted TEXT
- For the `/json/` router `regex` extracts JSON and deserializes the extracted data.

The environment variable `MATCH_JSON` toggles deserialization.

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

#### Markdown Extraction

Make a request to the `/markdown/` router and ask the LLM API for response in markdown. Test this with `GPT-4` for best results.

```sh
curl -sS -H "api-key: $AISERVICE_API_KEY" -H "Content-Type: application/json" -X POST -d' {
  "messages": [
    {
      "role": "system",
      "content": "Assistant is a large language model trained by OpenAI."
    },
    {
      "role": "user",
      "content": "Who are the founders of Microsoft? Please provide answer in 20 words in markdown"
    }
  ],
  "max_tokens": 150
}' "http://localhost:5380/markdown/openai/deployments/$DEPLOYMENT/chat/completions?api-version=2023-05-15"
```

Receive a response like this

```json
{
  "id": "chatcmpl-8gA57qCBPn7nTOMqhNDIiaFxoCRD7",
  "object": "chat.completion",
  "created": 1705059221,
  "model": "gpt-4",
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "```markdown\nMicrosoft was founded by Bill Gates and Paul Allen on April 4, 1975.\n```"
      }
    }
  ],
  "usage": {
    "prompt_tokens": 38,
    "completion_tokens": 22,
    "total_tokens": 60
  },
  "system_fingerprint": "fp_6d044fb900",
  "regex": {
    "match": true,
    "sections": [
      {
        "text": "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975.",
        "object": null
      }
    ]
  }
}
```

The `regex` processor has added the fields

```json
{
  "regex": {
    "match": true,
    "sections": [
      {
        "text": "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975.",
        "object": null
      }
    ]
  }
}
```

#### JSON Extraction

Make a request to the `/json/` router and ask the LLM API for response in JSON. Test this with `GPT-4` for best results.

```sh
curl -sS -H "api-key: $AISERVICE_API_KEY" -H "Content-Type: application/json" -X POST -d' {
  "messages": [
    {
      "role": "system",
      "content": "Assistant is a large language model trained by OpenAI."
    },
    {
      "role": "user",
      "content": "Who are the founders of Microsoft? Please provide answer in 20 words in json"
    }
  ],
  "max_tokens": 150
}' "http://localhost:5380/json/openai/deployments/$DEPLOYMENT/chat/completions?api-version=2023-05-15"
```

Receive a response like this

```json
{
  "id": "chatcmpl-8gA6hfW1QLmh2MaLTI8J55KraVyBq",
  "object": "chat.completion",
  "created": 1705059319,
  "model": "gpt-4",
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "```json\n{\n  \"founders_of_microsoft\": [\n    \"Bill Gates\",\n    \"Paul Allen\"\n  ]\n}\n```"
      }
    }
  ],
  "usage": {
    "prompt_tokens": 38,
    "completion_tokens": 27,
    "total_tokens": 65
  },
  "system_fingerprint": "fp_6d044fb900",
  "regex": {
    "match": true,
    "sections": [
      {
        "text": "{\n  \"founders_of_microsoft\": [\n    \"Bill Gates\",\n    \"Paul Allen\"\n  ]\n}",
        "object": {
          "founders_of_microsoft": [
            "Bill Gates",
            "Paul Allen"
          ]
        }
      }
    ]
  }
}
```

The `regex` processor has added the fields and deserialized the JSON object

```json
{
  "regex": {
    "match": true,
    "sections": [
      {
        "text": "{\n  \"founders_of_microsoft\": [\n    \"Bill Gates\",\n    \"Paul Allen\"\n  ]\n}",
        "object": {
          "founders_of_microsoft": [
            "Bill Gates",
            "Paul Allen"
          ]
        }
      }
    ]
  }
}
```

### Check the logs

Use the [Log Lister](http://localhost:8080/logs) to inspect the traffic

### Field Selection

The `regex` processor uses [gjson](https://github.com/tidwall/gjson) syntax to select response fields to extraction. This makes `regex` LLM API agnostic. The default field selection pattern is `choices[0].message.content`.

### Regular Expression

`regex` uses regular expressions to extract patterns from the response fields. `regex` is written in Go and uses the [Re2 library Syntax](https://github.com/google/re2/wiki/Syntax).
