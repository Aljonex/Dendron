---
id: 31fj8g3odol70g5gf879mkt
title: Helm Charts
desc: ''
updated: 1677067416370
created: 1677057891959
---
## Helm
Helm is effectively a package manager for Kubernetes, it solves the problem of allowing us to make templates for Kubernetes cluster set up.
Where Kubernetes is a container orchestration tool to allow the cumulative running of multiple docker containers across multiple machines with ease of replacement and consistent up time, **Helm** is a deployment tool for automating *creation, packaging, configuration, and deployment* of apps & configuration to the Kubernetes clusters.

To being the process navigate to your respective directory and type:<br>
`helm create <chart name>`

This will create a directory full of files required for helm to make a chart and the file tree is laid out with things like:
- `chart.yaml` - information related to the chart, *version, name, description* so you can find it if you publish it on an open repo, you can set external *dependencies* with the dependency key.
- `values.yaml` - this contains defaults for variables
- `templates (dir)` - where all the manifest files go, everything here gets passed on and created in kubernetes
- `charts` - if your chart depends on another chart, you can bring this same structure into this directory, and then chart dependencies are installed from bottom to top

## Basics