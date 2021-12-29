---
title: "Weekly update runner toolchain"
date: 2021-12-30T09:00:01+02:00
tags: [ github actions, runner, workflows, github enterprise server, ghes ]
draft: false
---

When using GitHub Enterprise Server in a large organization, you might run into problems due to limited or no internet connection. Or you might have set-up your runners to be detroyed everytime a workflow is triggered. In either case, you can speed up the workflow by using cached versions of tools the workflow need. Node, go, python, etc. These tools are setup with [actions/node-setup@v2](https://github.com/actions/setup-node) or similar step.

The general idea is that you run `actions/node-setup@v2` with every version you need and then create a tarball of the `runnner.tool_cache` folder.

You can install these cached toolchains in the `RUNNER_DIR/_work/_tool` folder grouped by tool. Just extract the toolchain to the `RUNNER_DIR/_work/_tool` folder and it will be automatically installed. e.g. `tar -xzf node_tool_cache.tar.gz -C RUNNER_DIR/_work/_tool/node`.

I created some example workfslows for Node, Go and Python which might be useful for you.
[cloudcosmonaut/runner-toolchains](https://github.com/cloudcosmonaut/runner-toolchains)
