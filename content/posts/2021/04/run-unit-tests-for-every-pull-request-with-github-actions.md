---
title: "Run unit tests for every Pull Request with GitHub Actions"
date: 2021-04-12T09:00:00+02:00
tags: [ github actions, devops, swift, pull request, pr, unit test ]
draft: false
---

Testing early and often in your software development cycle is a major part of the DevOps philosophy. With GitHub actions, you can run your test on every Pull Request (PR). Testing as early as possible will detect errors in an early stage so errors can be fixed before releasing.
 
## What are GitHub Actions?

You might know GitHub as the open source developers platform. GitHub now contains GitHub Actions. You can compare Github Actions with an Azure Devops Pipeline or an AWS CodePipeline. With these Actions you can create a workflow. A workflow is a way of building and deploying your software automatically, often triggered by a PR. 
 
You define a GitHub Action in a yaml file. In the next section I describe in yaml when and what the Github Action is going to do. 


## When to trigger

To start a GitHub Action you first need to specify when the Action should trigger. The trigger determines when the Action is executed. In this case the Action needs to trigger when a Pull Request is composed against the `main` branch.

```yaml
on:
  pull_request:
    branches: [ main ]
```

## Check out the code

Your Action needs to have access to your code, so you need to add a step to checkout your code.

```yaml
 steps:
    - uses: actions/checkout@v2
```

## Run the unit tests

This step requires you to pass the relative location of the Xcode Project or Workspace file, the scheme 
in your project to test and the destination (or target if that's more in your vocabulary)

```yaml
- name: Xcode Test
      uses: devbotsxyz/xcode-test@v1.1.0
      with:
        project: 'project.xsproj' # `project` and `workspace` cannot be used at the same time
        workspace: 'workspace.xcworkspace' # `project` and `workspace` cannot be used at the same time
        scheme: 'scheme'
        configuration: Debug
        destination: platform=iOS Simulator,OS=latest,name=iPhone 11
```

## Code coverage

When your code changes over time, you might want to know how your code coverage changes over time. You can do this with Codecov. If you use a private repo, you need to add a Codecov Access Token.

```yaml
    - uses: codecov/codecov-action@v1
      name: Determine code coverage with CodeCov
      # For private repos, an access token for codecov.io is required
      # token: ${{ secrets.CODECOV_TOKEN }}
```

## Add testing matrix

As I want to know that my library works successfully on different versions of iOS, I want to run my Unit Tests on several SDK versions.
 
To do this, you need to change the Test step, in my case `Xcode Test`, and add a so-called `matrix`. In my case I only need one dimension in which I put four SDK configurations.


```yaml
env:
  PROJECT_FILE: # Relative path to Xcode project file
  SCHEME: # Name of scheme to test

jobs:
  build:
  runs-on: macos-latest

    strategy:
      matrix: 
        xcode:
          - destination: platform=iOS Simulator,OS=latest,name=iPhone 11
            version : latest-stable
          - destination: platform=iOS Simulator,OS=14.4,name=iPhone 11
            version: 12.4
          - destination: platform=iOS Simulator,OS=13.7,name=iPhone 11
            version: 11.7
          - destination: platform=iOS Simulator,OS=12.4,name=iPhone 7
            version: 10.3
    - ...
    - name: Xcode Test
        uses: devbotsxyz/xcode-test@v1.1.0
        with:
        project: ${{ env.PROJECT_FILE }}
        scheme: ${{ env.PROJECT_SCHEME }}
        configuration: Debug
        destination: ${{ matrix.xcode.destination }}
```

## Result

When you completed all steps and committed the GitHub Action as a file in `.github/workflows/`, you will see the results of the checks when you create a PR.

![All checks green!](/20210413-validation-checks-on-pr.png)


### Complete pipeline yaml

Here you have the complete pipeline template which I used in one of my projects ([RFIBAN-Helper](https://github.com/readefries/IBAN-Helper/blob/main/.github/workflows/check-pr.yml)).

```yaml
name: Check PR

on:
  pull_request:
    branches: [ main ]

env:
  PROJECT_FILE: # Relative path to Xcode project file
  SCHEME: # Name of scheme to test

jobs:
  build:
    runs-on: macos-latest

    strategy:
      matrix: 
        xcode:
          - destination: platform=iOS Simulator,OS=latest,name=iPhone 11
            version : latest-stable
          - destination: platform=iOS Simulator,OS=14.4,name=iPhone 11
            version: 12.4
          - destination: platform=iOS Simulator,OS=13.7,name=iPhone 11
            version: 11.7
          - destination: platform=iOS Simulator,OS=12.4,name=iPhone 7
            version: 10.3
    
    steps:
    - uses: actions/checkout@v2
    - uses: maxim-lobanov/setup-xcode@v1
      with:
        xcode-version: ${{ matrix.xcode.version }}
    - uses: ruby/setup-ruby@v1
      with:
        bundler-cache: true      
    - name: Xcode Test
      uses: devbotsxyz/xcode-test@v1.1.0
      with:
        project: ${{ env.PROJECT_FILE }}
        scheme: ${{ env.PROJECT_SCHEME }}
        configuration: Debug
        destination: ${{ matrix.xcode.destination }}
    - name: Perform Cocoapod lib lint validation
      run: make validate
    - uses: codecov/codecov-action@v1
      name: Determine code coverage with CodeCov
```