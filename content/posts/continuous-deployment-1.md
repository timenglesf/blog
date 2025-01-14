---
date: "2024-08-14T22:06:02-08:00"
draft: true
title: "Continuous Deployment: Getting Started with CD for Containerized Go Apps Using GitHub Actions & GCP"
---

## Introduction to This Series

*The references to a blog in this post are for my older blog that was built from
scratch using Go. My new blog is built using Hugo and the Papermod.*

This post serves as an introduction to this series on creating a CD pipeline
for containerized Go apps using GitHub Actions and GCP.

## The Goal

Currently this blog is ready to host, but still needs some features that
I would like to add in order for it to make it more engaging for user, easier
for my to add posts, simpler to update posts, and add the ability to view
metrics on posts.

I do not want to have to deal with manually testing the code, and then building
and deploying Docker containers during each added feature and bug fix.

---

## Continuous Deployment

Continuous Deployment is the process of automatically deploying an updated
codebase to production after the updated code has passed all of the checks
that occur during Continuous Integration.

This series of blog posts will follow the journey of creating a Continuous
Deployment Pipeline for this blogging platform as a container using Docker,
GitHub Actions, and Google Cloud Platform.

- [Go](https://go.dev/): An open-source programming language that is great
  for web development.
- [Docker](https://www.docker.com/): Containerization software.
- [GitHub Actions](https://docs.github.com/en/actions): To automate our
  container build and deployment process.
- [Google Cloud Platform](https://cloud.google.com/): A cloud hosting provider.

### Why every project should be set up with continuous deployment

Continuous deployment eliminates the need for manual intervention when
deploying code updates to production servers. The process is triggered
automatically when certain events occur, like pushing new code to the `main`
branch of a GitHub repository. Various tools can manage continuous deployment,
and in this series, we’ll focus on using GitHub Actions.

With a well-structured CD pipeline, teams save countless hours by automating
repetitive tasks, allowing them to focus on building new features and fixing
bugs. Meanwhile, those updates are tested and deployed far more efficiently
than any manual process could achieve.
