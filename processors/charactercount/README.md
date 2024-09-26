# Charactercount

## Purpose

The `charactercount` processor is a simple template processor that counts the characters in the incoming payload and writes a custom field in the log. `charactercount` illustrates how to accept an incoming message to a processor and send it back to `gecholog`.

## Building

Before you proceed, build the [gecholog](https://github.com/direktoren/gecholog) container.

From this directory, run

```sh
  export NATS_TOKEN=changeme
  export GUI_SECRET=changeme
  docker compose up -d
```

## Using

Make a request to generate a log

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
}' "http://localhost:5380/echo/"
```

### Check the logs

Use the [Log Lister](http://localhost:8080/logs) to inspect the traffic. Try to locate the `character_count` field in the request object.
