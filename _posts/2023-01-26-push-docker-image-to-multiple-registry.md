---
title: Push Docker image to multiple registry
date: 2023-01-26 20:00:00 +0700
categories: [DevOps, CICD]
tags: [docker]
---

## Problem with Docker Registry

I have managed a [GitOps workflow](https://www.weave.works/technologies/gitops/) with Github Action and FluxCD for 3 years. Erverything is fine until Docker announced [new rate limit](https://www.docker.com/blog/scaling-docker-to-serve-millions-more-developers-network-egress/) that will take effect November 1, 2020. There is the 2 important quotes:

- `Free plan – anonymous users: 100 pulls per 6 hours `
- `Free plan – authenticated users: 200 pulls per 6 hours`

Because FluxCD use pulling model that checks new image by an interval time, so it does not work anymore because of new rate limit. I need to figure out a solution that lets current CICD workflow works well again

## Dead simple solution

The solution is simple - push docker image to 2 diffrent registries: Docker Registry and another reigstry that has larger rate limit or no rate limit at all. So, I will push the stable build (tag by semantic version) to Docker Registry to let everybody use it easily. For another registry (I am using [ECR](https://aws.amazon.com/ecr/)), FluxCD will check new build every minute and deploy it if I have pushed the latest code on main branch `master`

## Implementation

What we need here is 2 pipelines that run 2 different jobs to push 2 separate registries and some flags that let us know which pipline should be run (or we will run both of them). Here is the architecture we need to accomplish

![multiple-registry-pipline](/assets/img/multiple-registry-pipline.png)

The `prepare` is the key step where we will check which pipline should we run (for example, I need to push image to DockerHub and ECR). The condition logic is simple:

- If we have set `DOCKERHUB_USERNAME`, enable pipeline that push image to Docker Hub
- If we have set `ECR_USERNAME`, enable ECR pipeline too
- If we have set something else, enable corresponding pipeline as well

{% raw %}
```yaml
prepare:
    runs-on: ubuntu-latest
    outputs:
        ENABLED_ECR: ${{ steps.CHECK_ECR.outputs.ENABLED }}
        ENABLED_DOCKERHUB: ${{ steps.CHECK_DOCKERHUB.outputs.ENABLED }}
    steps:
        - id: CHECK_DOCKERHUB
        env:
            DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
        if: "${{ env.DOCKERHUB_USERNAME != '' }}"
        run: echo "ENABLED=true" >> $GITHUB_OUTPUT

        - id: CHECK_ECR
        env:
            ECR_USERNAME: ${{ secrets.ECR_USERNAME }}
        if: "${{ env.ECR_USERNAME != '' }}"
        run: echo "ENABLED=true" >> $GITHUB_OUTPUT
```
{% endraw %}

In each pipeline, you need to add a condition to let the pipeline of enabled flag run or be skipped. With Github Action it is too simple with the `if` condition in each job

{% raw %}
```yaml
dockerhub:
    runs-on: ubuntu-latest
    needs: [prepare]
    if: ${{ needs.prepare.outputs.ENABLED_DOCKERHUB == 'true' }} # <- where you tell Github Action whether we we let it run or not
    steps:
        - uses: actions/checkout@v1
        - ...
        
ecr:
    runs-on: ubuntu-latest
    needs: [prepare]
    if: ${{ needs.prepare.outputs.ENABLED_ECR == 'true' }} # <- where you tell Github Action whether we we let it run or not
    steps:
        - uses: actions/checkout@v1
        - ...
```
{% endraw %}

## Limitations

If we need to run too many pipelines (> 2 pipelines), you can find there is resource wasting at the workflow I have showed you. Because we need to run seperated job for different registry, we end up with run docker build multiple time. You can improve it by run only one job, set the flag to environment variable and check it at the step of push image to registry. But you can not reuse pre-defined actions from Github Actions Marketplace. 

In my case, they are [docker/build-push-action](https://github.com/docker/build-push-action) and [aws-actions/amazon-ecr-login](https://github.com/aws-actions/amazon-ecr-login)


> Be careful when you decided to manage credentials of registry by yourself cause there is a risk to leak them at Github Action logs
{: .prompt-warning }


## Bonus

Full version of Github Action file is placed on my open source project. You can find it at [Scraphook - The fast, secure, and efficient webhook service ](https://github.com/scrapnode/scraphook/blob/master/.github/workflows/master.yaml)