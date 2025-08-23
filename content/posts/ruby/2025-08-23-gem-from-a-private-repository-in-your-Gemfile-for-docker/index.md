---
date: '2025-08-23'
lastmod: '2025-08-23'
draft: false
title: "Gem from a private Gitlab repository in your Gemfile(for Docker)"
slug: 'gem-from-private-repository-in-your-gemfile-for-Docker'
authors: ['Alex']
enableReadingTime: true
description: "Learn how to use private GitLab gems in your Ruby on Rails Gemfile when deploying with Docker and Kamal. This guide covers setting up deploy tokens, configuring environment variables, and updating your Dockerfile for secure gem access."
keywords: ["Ruby on Rails", "Docker", "GitLab", "private gem", "Kamal deployment", "deploy token", "Gemfile", "Dockerfile"]
tags: ["Ruby on Rails", "Docker", "GitLab", "Private Gem", "Kamal", "Deployment", "DevOps"]
categories: ['Post']
hiddenFromHomePage: false
featuredImagePreview: "Gemfile_and_Docker.jpg"
featuredImage: 'Gemfile_and_Docker.jpg'
---


# Gem from a private Gitlab repository in your Gemfile(for Docker)

If you want to deploy your Ruby on Rails project with Kamal, 
and your project depends on a Gem hosted in a private GitLab repository, 
```ruby
gem "mygem", git: "https://gitlab.com/username/mygem.git", branch: "develop"
```
you need to follow a few extra steps.

## Generate a Deploy token
Go to your GitLab project (or group) → Settings → Repository → Deploy Tokens.

Create a new deploy token with at least read_repository scope.

Note the username and token — you will need them later.

## Add Credentials to .env
Create or update your .env file:
```bash
GITLAB_USER=your-user-name
GITLAB_TOKEN=your-deploy-token
```

## Add Secrets to Kamal
Update .kamal/secrets:
```bash
GITLAB_USER=$GITLAB_USER
GITLAB_TOKEN=$GITLAB_TOKEN
```
This makes the variables available during the Docker build process.

## Add changes to config/deploy.yml
Add build arguments so Kamal can pass them into the Docker build:

```yaml
builder:
  args:
    GITLAB_USER: ${GITLAB_USER}
    GITLAB_TOKEN: ${GITLAB_TOKEN}
```

## Configure Dockerfile
In your Dockerfile, configure Git to use the credentials for GitLab:
```dockerfile
ARG GITLAB_USER
ARG GITLAB_TOKEN

RUN git config --global url."https://${GITLAB_USER}:${GITLAB_TOKEN}@gitlab.com/".insteadOf "https://gitlab.com/"
```

Now, when Docker runs bundle install, it will be able to fetch your private Gem.