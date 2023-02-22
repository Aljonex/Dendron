---
id: y01vmzp44eykbfjtp2vnuig
title: Getting-Started
desc: 'Page about the initial setup and issues around Kubernetes'
updated: 1677060754645
created: 1676025937352
---
### Initial stages
- Downloaded Rancher Desktop and set it up for *containerd* - which is a container runtime for **simplicity, robustness, and portability**

### Basics 
`kubectl` is the command-line tool which allows running of commands against Kubernetes clusters. It can be used for application deployment, inspection or management of cluster resources and the viewing of logs. <br>
[[kubectl Docs | https://kubernetes.io/docs/reference/kubectl/]]

General layout of syntax is: `kubectl [command] [TYPE] [NAME] [flags]`

### Setting up a local Repository
The way I understand is Kubernetes is almost like Git in the sense that you have a repository for your pod files and clusters, to do this you would do the command:
`docker run -d -p 5000:5000 --restart=always --name registry registry:2`
- `-d` runs this in a detached state so you don't need to see the logs (this can be ascertained by `docker logs -f <containerID>`)<br>
[[Medium article on this | https://medium.com/htc-research-engineering-blog/setup-local-docker-repository-for-local-kubernetes-cluster-354f0730ed3a]]<br>
[[Stackoverflow | https://stackoverflow.com/questions/36874880/kubernetes-cannot-pull-local-image/59269154#59269154]]

#### Useful commands
- Stop all pods: `kubectl delete all --all --namespace default`
- Check rollout status of deployment: `kubectl rollout status deployment/<deployment-name>`
- *initContainers* are containers that run before the app containers, for example I'm attempting to run postgres and pgadmin before the jsf app, to see the logs of those specifically do `kubectl logs <pod-name> -c <init-container-name>`

### Common Errors
`Unable to connect to the server: dial tcp 172.21.130.106:6443: i/o timeout` - this is due to not running Rancher Desktop<br>
Kubernetes doesn't allow the use of underscores in the naming of volume mounts.<br>
*initContainers* must have something that successfully has exit code of 0 before moving onto next *initContainer* so you should add the `command: [<commands>]` in the yaml and verify what you wanted to happen, happened

Had an issue with CSRF tokens on the login when using a *deployment, service, ingress* but it was because there were multiple replicas so when switching to 1 it fixed this error. Now running into the problem of `UnknownHostException: postgres_11_dev` coming from the use of `postgres_11_dev` as the db host in the app deployment yaml, should be simple wiring fix.