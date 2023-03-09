---
id: 5dmvg7mnnkltzrvzih72xk1
title: Argo
desc: 'Objective to create a moodle page for Argo that future trainees can go through to learn about it'
updated: 1678108815074
created: 1675855409245
---
## Prerequisites
![[Programming.Tools.Docker#docker]]
> Confluence: 
<br> [[Docker Optional Tut | https://confluence.apak.com/live/pages/viewpage.action?pageId=88444148]]
<br> [[Docker Cloud Tut | https://confluence.apak.com/live/display/WIKI/Cloud+Training%3A+Docker]]

![[Programming.Tools.Kubernetes#kubernetes]]
> Confluence:
<br> [[Kubernetes Cloud Tut | https://confluence.apak.com/live/display/WIKI/Cloud+Training%3A+Kubernetes]]
<br> [[Kubernetes Training and Local Setup | https://confluence.apak.com/live/display/WIKI/Kubernetes+Training]]

![[Programming.Tools.Helm#helm]]
> Confluence:
<br> [[Helm Charts Cloud Tut | https://confluence.apak.com/live/display/WIKI/Cloud+Training%3A+Helm+Charts]]

> Confluence:
<br> [[Argo | https://confluence.apak.com/live/display/~rich.ellor/Argo]]

## Argo
Followed getting started from the *argocd* docs, changed password to *`welcome2022`*.

Manually added repo from https token created in bitbucket `argocd repo add https://bitbucket.apak.delivery/scm/train/training.git --username alex.jones3 --password <token>`

Port forwarding is `kubectl port-forward svc/argocd-server -n argocd 8080:443`

> The creation and making of Argo servers doesn't work connecting bitbucket to the local instance due to some timeout errors, in the end I used an HTTPS access token (no username) as authentication and when connected to non-corporate proxy it worked

```bash
 kubectl config set-context --current=true -
-namespace=default
Context "rancher-desktop" modified.
alexajones2@ITEM-S134843:/mnt/c/Users/alexajones2/app (feature/aj-cloud-training) $ kubectl get all
NAME                                      READY   STATUS    RESTARTS   AGE
pod/argo-test-test-chart-874957d6-7zfqt   2/2     Running   0          9m44s

NAME                           TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)          AGE
service/kubernetes             ClusterIP      10.43.0.1      <none>         443/TCP          21h
service/argo-test-test-chart   LoadBalancer   10.43.192.10   172.30.8.145   8080:31040/TCP   9m44s

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argo-test-test-chart   1/1     1            1           9m44s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/argo-test-test-chart-874957d6   1         1         1       9m44s
```
Then to port forward to check app is working
`kubectl port-forward <podname> <targetPort>:<containerPort>` and you can see the container port in `kubectl describe <deploymentName>`

![Image displaying app tree in GUI, after adding second replica](pics/details-tree-gui.png)

> NEED: known hosts file from windows to be added, from WSL didn't work as lacked correct formatting, an HTTPS or SSH token and then setup correctly with branch and helm chart included on it, then you can either create app by putting YAML in or following GUI.