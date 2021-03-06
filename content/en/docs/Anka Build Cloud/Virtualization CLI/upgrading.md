---
date: 2019-12-12T00:00:00-00:00
title: "Upgrading"
linkTitle: "Upgrading"
weight: 6
description: How to upgrade the Anka Build Virtualization CLI
---

> We follow [semantic versioning](https://semver.org/); minor and major version increases can have significant changes.

## Upgrade Procedure (without Anka Build Cloud Controller & Registry)

1) Install the new anka pkg. Follow the [install instructions]({{< relref "installation.md" >}})

2) Upgrade the guest addons inside the VM templates with `anka start -u`. Check the upgrade notes to see if this step is necessary.

## Upgrade Procedure (with Anka Build Cloud Controller & Registry)

When upgrading the entire Anka Build infrastructure (Controller/Registry and Nodes), execute the steps in the following sequence:

1) Run `sudo ankacluster disjoin` on your nodes

2) Install the new anka pkg. Follow the [install instructions]({{< relref "installation.md" >}})

3) Upgrade the guest addons inside the VM templates with `anka start -u`

4) Push the newly upgraded VM templates to registry with `anka registry push {template} --tag <tag>`

5) Go to the Controller & Registry and upgrade to the latest version.

6) On the nodes, run `sudo ankacluster join http://{controller address}`

## Anka Build Virtualization CLI upgrade note matrix

Existing Version | Target Version | Recommendation
--- | --- | ---
1.4.3 | 2.x.x | Requires upgrade of all existing VM templates with `anka start -u` and push to the registry
2.x | 2.1.2 | Only requires upgrade of existing Catalina VM templates with `anka start -u` and push to the registry
