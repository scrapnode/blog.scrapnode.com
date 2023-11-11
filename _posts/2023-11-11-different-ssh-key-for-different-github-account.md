---
title: Different SSH key for different Github account
date: 2023-05-20 9:30:00 +0700
categories: [Git]
tags: []
---

## The problem

As you know, we can specify the SSH key to be used when interacting with a remote repository. The configuration file is located at `~/.ssh/config`, with a format like this:

```
Host github.com
    Hostname github.com
    User scrapnode
    IdentityFile ~/.ssh/scrapnode
```

This setup works seamlessly if you have only one user per host. However, in my case, I have a GitHub account created with my work email and another GitHub account created with my personal email. Unfortunately, on the same machine, I cannot use both accounts to interact with GitHub remote repositories because the SSH agent only selects the configuration of the first match."

## The solution

If you carefully examine the configuration file at `~/.ssh/config`, you'll notice that the Hostname represents the actual address of the remote server, while the Host property functions more like an alias or a user-defined name for your configuration. To illustrate this point, consider the following example from my `~/.ssh/config` file, which I use to connect to an EC2 instance:

```
Host database-postgres
    Hostname 123.21.35.174
    User ubuntu
    IdentityFile ~/.ssh/database
```

With this setup, when you want to connect to the host `database-postgres` you simply use the command `ssh database-postgres` instead of the more complex `ssh -i ~/.ssh/database ubuntu@123.21.35.174`. This streamlined approach works smoothly because the SSH agent intelligently selects the appropriate configuration, facilitating a seamless connection to your remote host.

As a continuation of this approach, we can modify the file `~/.ssh/config` to enable SSH connections to the same host but with different users and identity files. The following configuration should work:

```
Host personal.github.com
    Hostname github.com
    User scrapnode
    IdentityFile ~/.ssh/scrapnode

Host company.github.com
    Hostname github.com
    User scrapnode
    IdentityFile ~/.ssh/company
```

After updating the `~/.ssh/config` file, you also need to adjust the interaction address with your GitHub repositories when configuring the remote server connection. Instead of using the format `git@github.com:USERNAME/REPOSITORY.git` (for example: `git@github.com:scrapnode/blog.scrapnode.com.git`), use the formats `git@personal.github.com:USERNAME/REPOSITORY.git` and` git@company.github.com:USERNAME/REPOSITORY.git` for personal and company repositories, respectively.

- Clone a repo

    ```bash
    # personal repository
    git clone git@personal.github.com:scrapnode/blog.scrapnode.com.git
    # company repository
    git clone git@company.github.com:COMPANY_NAME/COMPANY_REPO.git
    ```

- Update the existing repository remote url

    ```bash
    # personal repository
    git remote set-url origin git@personal.github.com:scrapnode/blog.scrapnode.com.git
    # company repository
    git remote set-url origin git@company.github.com:COMPANY_NAME/COMPANY_REPO.git
    ```
