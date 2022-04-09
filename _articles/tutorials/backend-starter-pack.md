---
layout: default
title: Backend Starter Pack
parent: Tutorials
nav_order: 1
---

# Backend Starter Pack

{: .no_toc }

Control your tools to get better development experiences and work faster.

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Terminal

- [Iterm2](https://iterm2.com/) There are a lot of terminal app you can find in the internet but Iterm2 is the first terminal that sastified me. So I keep using it until now.
- [Oh-My-Zsh](https://ohmyz.sh/) The main reason brought me to Oh-My-Zsh is they have the powerful prompt feature and it can remember my command history =))
- [powerlevel10k](https://github.com/romkatv/powerlevel10k) The ultimate theme for my terminal. Easy to setup through their wizard instruction
- [httpie](https://httpie.io/) Replacement of cURL with awesome arguments configuration and pretty response.

## SSH management

Make sure you know how to create and use SSH. Check [Github Tutorial](https://docs.github.com/es/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) if you don't know it.

When you are working in this industry, almost time you have to manage more than one SSH connection in your machine. Some platform does not allow you to have more than one SSH key in their platform and that is cause of all your painful every single time you want to connect to them.

Fortunately, we have some configuration that can rescue your day. Here is an example that help you manage your SSH connection to _Github_

```bash
# cat ~/.ssh/config

Host github.com
  Hostname ssh.github.com
  Port 443
  User scrapnode
  IdentityFile ~/.ssh/id_tuannguyen

Host company.github.com
  Hostname ssh.github.com
  Port 443
  User tuanatcompany
  IdentityFile ~/.ssh/id_tuanatcompany
```

The key here is you name your connection by the `Host` property. Whenever you connect to _Github_ by the `Host` instead of original url, SSH will use all the block configurations to your target.

Let's say I want to clone a personal project that is under my username. I can clone it with the url `git@github.com:scrapnode/blog.scrapnode.com.git` as the way I used to did.

But whenever I need to clone/pull/push to my company repository, I have to replace a part `github.com` with `company.github.com`. Please look at the example bellow to understand the idea clearer

```bash
# Clone project
# Before
git clone git@github.com:company/project.git
{: .text-red-300}
# After
git clone git@company.github.com:company/project.git

# Or add remote conf
# Before
git remote add origin git@github.com:company/project.git
# After
git remote add origin git@company.github.com:company/project.git
```

## Container engine

Both [Docker Desktop](https://www.docker.com/products/docker-desktop/) and [Podman](https://podman.io/) is fine. But I had been using Docker Desktop for more than 6 years so that why I will recommend it to my team.

## VIM

Yes! [VIM](https://www.vim.org/) will help you a lots in any task that need performing on your server. At least you should know which I list bellow

- Move a line up or down
- Copy/pass a line
- Select/delete multiple words
- Copy/pass a word
- Go to the top/bottom of file
- Delete all content of file
- Search a word
