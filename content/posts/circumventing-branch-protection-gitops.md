---
title: "Circumventing Branch Protection for Fun and GitOps"
date: 2020-11-19
draft: false
toc: false
images:
tags:
  - GitOps
  - Github Actions
---

Over the last year, Quizlet has adopted GitOps for managing our deploys and infrastructure. During the migration we repeatedly found ourselves needing to work around Github’s branch protection features when writing GitOps tooling. Thanks to a happy accident, we found a solution to this problem which we are publishing as a Github Action ([quizlet/commit-changes-action](https://github.com/quizlet/commit-changes-action)) so that others can benefit.

## The Problem

[Protected branches](https://docs.github.com/en/free-pro-team@latest/github/administering-a-repository/about-protected-branches) are a Github feature that allows you to manage how changes are introduced to a branch, such as requiring reviews or successful checks. This is a critically important tool when you’re following GitOps and merging to master will change production. The problem arose as we wrote more GitOps automation that committed to protected branches.

For example, our deployment chatbot triggers a Github Actions workflow that updates the image tag value in our Helm charts. However, because you cannot push directly to a protected branch from the CLI, even if you are an administrator, we ended up with some very complex workarounds. The workflow would shell out to git to create a branch with the changes, curl the Github API to create a pull request, use administrative privileges to ignore the branch protection rules, and merge the changes. This created a lot of unnecessary pull requests and unactionable notifications for developers when [Code Owners](https://docs.github.com/en/free-pro-team@latest/github/creating-cloning-and-archiving-repositories/about-code-owners) assigned them as reviewers on those PRs.

## The Solution

We stumbled across the solution when an admin using Github’s online editor discovered they could commit their changes directly to master, ignoring our branch protection rules. After digging around in the API documentation, we found an [endpoint for updating a file](https://docs.github.com/en/free-pro-team@latest/rest/reference/repos#create-or-update-file-contents) that also ignores the protection rules if the authenticating user was an admin. While this endpoint wouldn’t have worked for us as it only works on a single file at a time, the documentation included a hint that pointed us to the perfect solution: the Git Database API.

## The Git Database API

The [Git Database API](https://docs.github.com/en/free-pro-team@latest/rest/guides/getting-started-with-the-git-database-api) exposes the raw Git objects of a repository and allows the manipulation of them through REST API calls. This post is going to avoid diving into Git’s internals, but if you’re curious, sections [10.2](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects) and [10.3](https://git-scm.com/book/en/v2/Git-Internals-Git-References) of the Git book explain the relevant concepts well. After some experimentation, we were able to use this API to perform all of the steps required to create and push a commit while circumventing the branch protection rules.

## commit-changes-action

The result is a generic Github Action for authoring commits using the Git Database API which is available for use in your projects right now! [quizlet/commit-changes-action](https://github.com/quizlet/commit-changes-action) is written in Typescript using Github’s official libraries and can match globs of files to commit, retry on errors, and avoid empty commits. Feel free to file feature requests and bugs if you find any ways we can improve it!
