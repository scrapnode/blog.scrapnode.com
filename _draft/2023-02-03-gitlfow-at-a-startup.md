---
title: Gitflow at a startup
date: 2023-02-03 17:00:00 +0700
categories: [Tools, Git]
tags: [gitflow]
---

[Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) is a way you manage your Git branches. As a Princial Engineer in a startup, I have responsibility for make that flow becomes suitable for business goals, make them straightforward to teammates (both tech and non-tech guys) and easy to maintain. It is hard but fortunately we figured it out.

## Requirement

The more simple requirement is, the more complicated to implement. I am sad to say we got it and had to deal with it. The requirement is

```
We want to develop X features at the same time then we can select which one should be released. Btw we want able to change the priority of release feature at any time we want.
```

You see? It's simple. We are fast growing startup that need push more features to our customers to achieve business goals. That why in requirement we need to develop X features at the same time. Those feature could be a request feature from our merchant. Or it is improvement of an existing feature. Or it is just a research feature that need to do A/B testing. All of them will be planned to release later when we can finalized the list of release candidates. And you know, change request is make frequently at startup. Nothing guarantees a feature is deloyed as the plan said

## How did we do it?

### Entities definition

We need to clarify how many type of git branch we have

1. Tags are release entity. All of release must use tag as source of release. Example format: `v1.0.1`
2. `master` / `main` are branches contain **STABLE** code. That means we can release new version by tag this branch at **anytime** we want. They also are main working branches of QA team to verify those build before release
3. Develop branches are branches with `develop-` prefix. For example: `develop-zero`, `develop-one`, `develop-two` and so on. Those branches contain new feature code and bug fixes. They are main working branches of developer so they can test their code on those environments
4. Other branch for new feature (prefix `feature/`) or bug fixes (`fix/`,` hotfix/`)