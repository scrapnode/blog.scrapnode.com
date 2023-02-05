---
title: Gitflow at a startup
date: 2023-02-03 17:00:00 +0700
categories: [Tools, Git]
tags: [gitflow]
---

[Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) is a way you manage your Git branches. As a Princial Engineer in a startup, I have responsibility to make the flow suitable for business goals, straightforward for teammates (both tech and non-tech guys) and easy to maintain. It is hard but fortunately we figured it out.

## Requirement

Simpler requirement often leads to more complex implementations, which we have unfortunately experienced. And the requirement that our Project Manager gave me is

```
We want to develop multiple features simultaneously and then chose which ones to release, while being able to change the release priority at any time.
```

As a fast-growing startup, the developer team often receive change requests to align with our business goals. For example, we want to do R&D for feature A and implement a feature for one of our merchant and do a fix for a bug on production environment at the same time. We decide ship the fix first, then R&D feature then merchant feature at the end. Suddenly we need to deliver merchant feature first (or we dead) but the code structure is organize to deliver as the way we define before. Now we have to change it and that is the frustrating and painful part 

## How did we do it?

### Entities definition

We need to clarify the types of git branches we have

1. Tags are used to identify releases
2. The `master` or `main` branches contain **STABLE** code,meaning that a new version can be released by tagging that branch **anytime**. These branches are also the main working branches for QA team to verify builds before release
3. The develop branches, with the prefix `develop-`, such as `develop-zero`, `develop-one`, `develop-two`, contain new feature code and bug fixes. The branches are the main working branches for developer, where they can test their code on these environments
4. Other branches for new feature have the prefix `feature/` and those for bug fixes have the prefix `fix/` or ` hotfix/`