---
layout: default
title: Push image to both private and public registry by same codebase
summary: How do we configure the CI that allow push image to different registry in different repository?
date: 2022-04-23
nav_order: 1
parent: DevOps
grand_parent: Random
---

# Push image to both private and public registry

## Problem

I has been developing an open source that is also used in our company for 3 months. I had been hosting both of them on Github: the first one is a public repository for open source and the second one is a private repository for company usage. I want to push the image to Docker Hub for open source project and ECR for my company project.

So, I want a configuration that is achieved a goal:

    - If the public repository got a push, Github Action should only build and push new image to Docker Hub
    - If I pushed code to the private repository, only ECR should got the new build

Everything is easy if you use separate code for public and private repository. But I want to push same code to both repositories

## Solution

The logic is simple, if a repository has a secret for ECR, CI should only run ECR task. Otherwise, Github Action should only run DockerHub task. Here is the diagram

![Github Action ECR - Dockerhub](/assets/img/github-action-ecr-dockerhub.png)

The main point is in `prepare` step. We have to do some check then decide which is the next action we should run. So my task is set the secret `DOCKER_HUB_USERNAME` on public repository and `ECR_USERNAME` on the private.

{% raw %}

```yaml
prepare:
  runs-on: ubuntu-latest
  outputs:
    IS_DOCKERHUB: ${{ steps.CHECK_DOCKERHUB.outputs.defined }}
    IS_ECR: ${{ steps.CHECK_ECR.outputs.defined }}
  steps:
    - id: CHECK_DOCKERHUB
      env:
        DOCKER_HUB_USERNAME: ${{ secrets.DOCKER_HUB_USERNAME }}
      if: "${{ env.DOCKER_HUB_USERNAME != '' }}"
      run: echo "::set-output name=defined::true"
    - id: CHECK_ECR
      env:
        ECR_USERNAME: ${{ secrets.ECR_USERNAME }}
      if: "${{ env.ECR_USERNAME != '' }}"
      run: echo "::set-output name=defined::true"
```

{% endraw %}

In another jobs, we have to make them depend on `preapre` step and check whether the check flag is true or not.

{% raw %}

```yaml
dockerhub:
  runs-on: ubuntu-latest
  needs: [prepare]
  if: ${{ needs.prepare.outputs.IS_DOCKERHUB == 'true' }}
  steps:
    - uses: actions/checkout@v1
ecr:
  runs-on: ubuntu-latest
  needs: [prepare]
  if: ${{ needs.prepare.outputs.IS_ECR == 'true' }}
  steps:
    - uses: actions/checkout@v1
```

{% endraw %}

If you set not only secrets `DOCKER_HUB_USERNAME` but `ECR_USERNAME`, both of your registry will received the update.
