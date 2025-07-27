---
title:  "Docker model runner"
date:   2025-07-27 16:27:00 +0100
---

Sometime in April this year, Docker added new feature called Docker Model Runner. It's meant to streamline the process of pulling, running, and serving large language models (LLMs) and other AI models directly from Docker Hub or OCI-compliant registries.

It integrates with Docker Desktop and Docker Engine, allows you to serve models via OpenAI-compatible APIs, package GGUF files as OCI, and interact with models from the command line.

## Features

- Pull and push models
- Serve models on OpenAI-compatible APIs
- Package and publish GGUF files as OCI
- Run AI models directly from the command line
- Manage local models and display logs

## Requirements

Docker Model Runner is supported on the following platforms:
- Windows (amd64) - NVIDIA GPUs
- Windows (arm64) - Qualcomm Adreno GPU
- MacOS (Apple Silicon)
- Linux - NVIDIA GPUs

## How it works

You pull the model, run it and you can interact with it from command line, or you can use OpenAI APIs:
- GET /engines/llama.cpp/v1/models
- GET /engines/llama.cpp/v1/models/{namespace}/{name}
- POST /engines/llama.cpp/v1/chat/completions
- POST /engines/llama.cpp/v1/completions
- POST /engines/llama.cpp/v1/embeddings

Note that models are loaded into memory only at runtime when a request is made, and unloaded when not in use. Also, keep in mind that pulling models can take some time, since thier OCIs tend to be large.

## Enable Docker Model Runner

### Docker Desktop

To enable it in Docker Desktop, just go to beta features and tick the checkbox. Make sure you have version 4.40+

### Docker Engine

- just install DMR as a package. For example, on ubuntu run:
```bash
$ sudo apt-get update
$ sudo apt-get install docker-model-plugin
```
- verify installation:
```bash
$ docker model version
```

## Pull a model

Run:
```bash
docker model pull ai/smollm3
# with specific tag
docker model pull ai/smollm2:360M-Q4_K_M
# pulling from hugging face
docker model pull hf.co/bartowski/Llama-3.2-1B-Instruct-GGUF
```

Version of the model is part of the name, while tags represent model quantization.

## Run a model

Run:
```bash
docker model run ai/smollm2
```

In case you have some issues running a model, they expose logs which can be viewed with:
```bash
docker model logs
```

## Calling OpenAI endpoints

```bash
# from within a container
curl -X POST -H "Content-Type: application/json" -d @data.json "http://model-runner.docker.internal/engines/llama.cpp/v1/chat/completions" 
 
# from the host
curl -X POST -H "Content-Type: application/json" -d @data.json "http://localhost:12434/engines/llama.cpp/v1/chat/completions"
```

An example payload in `data.json` file:
```json
{
    "model": "ai/smollm2",
    "messages": [
        {
            "role": "user",
            "content": "Hello there"
        }
    ]
}
```

## Compose

Docker Compose also has support for models. You can now define if a service depends on a model and docker engine will make sure that service can access the OpenAI endpoins that model runner exposes.

Here's a simple example:

```yaml
services:
  app:
    image: app:latest
    models:
      - llm

models:
  llm:
    model: ai/smollm2
```

## More

If you're interested to learn more, check the offical [docs](https://docs.docker.com/ai/model-runner/) or take a look at my example application [here](https://github.com/frennky/gochat).
