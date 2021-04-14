---
title: "Running unit tests for every PR"
date: 2021-04-12T09:00:00+02:00
tags: [ github, actions, devops, swift ]
draft: false
---

A major part of the DevOps philosophy, is testing early in your development cycle and test often. To do that, you can run your 
test on every PR. Perferably on multiple SDK versions.


## When to run

First we need to specify when the Action should trigger. In this case I want to run the Action when a Pull Request is composed agains the `main` branch.

```yaml
on:
  pull_request:
    branches: [ main ]
```

## Check out the code

In most cases, this step doesn't require any configuration. When you want to access sub-modules, files from git-lfs or a remote GIT repository you might need to change some settings.

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

You might want to track changes in code coverage to get some idea on the quality of code. With Codecov
you need to add an Acces Token when your project is on a private repo.

```yaml
    - uses: codecov/codecov-action@v1
      name: Determine code coverage with CodeCov
      # For private repos, an access token for codecov.io is required
      # token: ${{ secrets.CODECOV_TOKEN }}
```

## Add testing matrix

But now you want to test them on different SDKs. Then you need to change the `Xcode Test` step and 
add a so called `matrix`. In this matrix you can define a multidemensional array, for which each
value which is used to combine all values of every attribute in that array.

In this case, we only need one deminsion: the Xcode version which is bound to the iOS SDK version.


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

When completed, you can see all checks in your PR:

![All checks green!](/20210413-validation-checks-on-pr.png)


### Complete pipeline yaml

Here you have the complete pipeline template which I use in one of my projects ([RFIBAN-Helper](https://github.com/readefries/IBAN-Helper/blob/main/.github/workflows/check-pr.yml)). 

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