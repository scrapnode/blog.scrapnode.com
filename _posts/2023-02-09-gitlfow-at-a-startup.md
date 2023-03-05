---
title: Gitflow at a startup
date: 2023-02-09 21:15:00 +0700
categories: [Git, Mangement]
tags: [gitflow]
---

[Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) is a way you manage your Git branches. As a Princial Engineer in a startup, I have responsibility to make the flow suitable for business goals, straightforward for teammates (both tech and non-tech guys) and easy to maintain. It is hard but fortunately we figured it out.

## Requirement

Simpler requirement often leads to more complex implementations, which we have unfortunately experienced. And the requirement that our Project Manager gave me is

```
We want to develop multiple features simultaneously and then chose which ones to release, while being able to change the release priority at any time.
```

As a fast-growing startup, the developer team frequently receives change requests to align with our business goals. For example, we want to do R&D for feature A, implement a feature for one of our big merchants and fix a bug on production environment simultaneously. Initially, We decided prioritize the bug fix, followed by the R&D works, and end with merchant feature. However, we suddenly received a request to deliver the merchant feature first (or we die). Unfortunately, our code structure was organized to deliver the features in the previously defined order. Now we have to change it and that is a frustrating and painful process. 

To address these challenges, I have suggested a workflow that allows us develop and review our changes in multiple environments. This way, we can prioritize the order of feature releases when we are close to the deadline, and make any necessary changes without compromising the quality of out work.

## Definitions

We need to take a look at some definitions to understand what we are working with. They are entities of Git Workflows and the naming convention of those entities

### Entities

We need to clarify the types of git branches we have

1. Tags are used to identify releases
2. The `master` or `main` branches contain **STABLE** code,meaning that a new version can be released by tagging that branch **anytime**. These branches are also the main working branches for QA team to verify builds before release
3. The develop branches, with the prefix `develop-`, such as `develop-zero`, `develop-one`, `develop-two`, contain new feature code and bug fixes. The branches are the main working branches for developer, where they can test their code on these environments
4. Other branches for new feature have the prefix `feature/` and those for bug fixes have the prefix `fix/` or ` hotfix/`

### Naming convention

- **Tag:** We follow the [Semantic Versioning](https://semver.org/), we use the release datetime as the tag name, formatted as vYYYY.MM.DD. For example: `v2022.1201.2030`
- **Feature branches:** Each feature has its own ticket, so we will name the branch using the ticket ID. For example: `feature/SH-147`
- **Bug fix branches:** Similarly, we name bug fix branches using the ticket ID. For example: `fix/SH-223` or `hotfix/SH-371`

## Workflows

![gitflow](/assets/img/2023-02-09-gitflow.png)

Typically, we have three main workflows: one for new features, another for bug fixes and the last one for hotfix

### Feature branches

![feature-branches](/assets/img/2023-02-09-feature-branches.png)

Developers need to select an unuse branch, make it their working branch, and synchronize it with the `master` branch. Next, they should create a new branch specific to their assigned ticket, using the ticket ID as the branch name. For example, if a developer want to implement new feature using the `develop-one` branch and they have ticket ID `SH-147`, they have to create a branch called `feature/sh-147` from `develop-one`. After complete their implementation on `feature/sh-147`, they have to merge it back into `develop-one` and continue working on other tickets.

### Bug fixes

![fix-branches](/assets/img/2023-02-09-fix-branches.png)

Similarly, developers must first select a working branch (lets say it is `develop-two`), then create a new branch specificlly for the bug they are fixing. The branch name should be based on the ticket ID, such as `fix/sh-223`. After completing their work on the bug, the developers should merge the fix branch (`fix/sh-223`) back into the working branch and move on to other tickets.

### Hotfix

![hotfix-branches](/assets/img/2023-02-09-hotfix-branches.png)

Hotfix branches are different story. They are using for critical problems that cannot be solved using the standard bug fixing workflow, as they require immediate action due to the bug being in production. Furthermore, the version contains the bug in production is often outdated compared to the `master` branch. Because of all of those reason, we need to create hotfix branches from the tag contains bug instead of from the `master` branch. For example, if there is a bug at version `v2022.1201.2030` and we have ticket `SH-371` for it, we must create a branch `hotfix/sh-371` from that version, fix the bug then deploy a new version `v2022.1202.330` by tag the branch `hotfix/sh-371`. If everything works well after the patch, we can merge the branch `hotfix/sh-371` into the `master` branch to keep the fix in main stream.

## Operations

All of the workflows we have defined allow developers to work freely with their own branch and enviroment without worrying about messing up the main branch. Once they have finised their works, they can make a Pull Request to staging branch and wait for feedback from reviewers. The PR is kept open until we finalize the list of features or bug fixes for the next release. Then, Teach Leads for those items can merge corresponding PR in the order of list features or bug fixes, and the QA team can start testing the changes on the staging environment

## Automation

With various developer branches as previously mentioned, manual management can become challenging, especially when deploying new code to multiple environments every time the developers push their changes. To make it easier, you need a robust CI/CD workflow to automate all deployment operations. Here is the workflow that we are use to manage our Gitflow processes

1. When the developer commits their code with a predefined hashtag to a development branch (e.g. `develop-one`), a GitHub Action workflow is triggered for the corresponding branch. This workflow builds and pushes a new Docker image to the Image Registry
2. Our CD platform checks for new images every five minutes and deploys them if a new image is found
3. Upon successful deployment, the CD platform sends a notification to our Slack channel, letting team members know their deployment is completed. They can then test their features or bug fixes in the development environment

Following my team usage, the CI/CD wokflow helps us deploy developer changes in only 15 minutes from the time they commit their changes.

## Results

Prior to implementing our new workflow, we are only able to deliver a limited set of feature along with a few small bug fixes each week due to the constraints of our environments. It was frustrating not being able to test our feature early and make necessary changes promptly. However, that is all in the past now. With new workflow in place, we can deliver at a much faster pace, developing multiple features or patching bugs simultaneously across multiple environments. According to our report, we are now able to ship up to three features and eight bug fixes per week. This is truly remarkable progress of a small team like us.