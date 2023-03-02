---
id: 31fj8g3odol70g5gf879mkt
title: Helm Charts
desc: ''
updated: 1677584836968
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
Once the basics of your charts are defined and the relative docker image is selected and the port to be forwarded to is chosen, enter the directory above the `helm create <chart-name>` *chart-name* directory and run `helm install <chosen-name> <chart-name>` but if you don't want to choose a name for the k8s cluster you can add the `--generate-name` flag.

If you want to test that a deployment with helm should work without initialising everything you can run from inside the directory with `helm install <NAME> . --dry-run --debug` and if a level above the directory simply replace `.` with `<directory-name>`

To lint and check files `helm lint <chart-name>`