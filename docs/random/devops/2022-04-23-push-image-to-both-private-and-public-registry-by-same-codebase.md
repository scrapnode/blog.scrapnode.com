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

I has been developing an open source that is also used in our company for 3 months. I had been hosting both of them on Github: the first one is a public repository for open source and the second one is a private repository for company usage. With open source repository, I am pushing the image to Docker Hub and for private repository I am pushing it to ECR on AWS.

So, I want a configuration that is achieved a goal:

    - If the public repository got a push, Github Action should only build and push new image to Docker Hub
    - If I pushed code to the private repository, only ECR should be got the new build image

Everything is easy if you used separate code for public and private repository. But I want to push same code to both repositories

## Solution

The logic is simple, if a repository has a secret for ECR, it should only run ECR Action. Otherwise, Github Action should only run Github Action. Here is the diagram

![Github Action ECR - Dockerhub](/assets/img/github-action-ecr-dockerhub.png)

The main point is in `prepare` step. We have to do some check then decide which is the next action we should run.

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

We have to run two steps, the first one will check whether the secret `DOCKER_HUB_USERNAME` is exist or not in our repository. If yes, the output `IS_DOCKERHUB` will be true. The same logic is applied to `IS_ECR`. In another jobs, we have to make them depend on `preapre` step and check whether the check flag is true or not.

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

If you set not only secrets `DOCKER_HUB_USERNAME` but `ECR_USERNAME`, both of your registry will received the update.
