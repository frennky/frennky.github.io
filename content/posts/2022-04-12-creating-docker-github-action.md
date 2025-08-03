---
layout: post
title:  "Creating Docker Github action"
date:   2022-04-12 20:19:30 +0100
---

# Creating Docker Github action

This is an example how to create a simple Docker Github action. To get more familiar with Github actions, check official [docs](https://docs.github.com/en/actions).

Start with an empty git repo:

```bash
mkdir docker-gha-example
cd docker-gha-example
git init
```

For repository to be considered a Github action, it is required to have a file `action.yml` or `action.yaml`, this metadata file will define inputs, outputs and run configuration.

```yaml
name: Docker Github action
description: This is an example of Docker Github action
branding:
  icon: star
  color: yellow

inputs:
  name:
    description: who to greet
    required: false
    default: 'world'

outputs:
  greeting:
    description: greeting message

runs:
  using: docker
  image: Dockerfile
  args:
    - ${{ inputs.name }}
```

For more details on `action.yml` check [docs](https://docs.github.com/en/actions/reference/workflows-and-actions/metadata-syntax).

Since docker image is referenced as `Dockerfile`, it is required to provide it inside repository.

```dockerfile
FROM alpine:latest

RUN apk update && apk add bash && rm -rf /var/cache/apk/*

WORKDIR /workspace

COPY . /workspace

ENTRYPOINT [ "/workspace/entrypoint.sh" ]
```

In this example, an entrypoint is a bash script, so it is required to provide it in repository as well.

```bash
#!/bin/bash

echo "Hello $1"
```

Finally, to test the action, create a Github workflow in this repository under `.github/workflows/main.yml`.

```yaml
name: Test action
on: push

jobs:
  test-action:
    runs-on: ubuntu-latest
    steps:
      - name: Run docker action
        id: docker-action
        uses: frennky/docker-gha-example@v1
        with:
          name: 'Github'

      - name: Print output
        run: echo ${{ steps.docker-action.outputs.greeting }}
```
