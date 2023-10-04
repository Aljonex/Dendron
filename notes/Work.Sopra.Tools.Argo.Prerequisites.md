---
id: pjtqhn7avgik4z9srh9html
title: Prerequisites to Argo Tutorial
desc: >-
  A documennt covering the prerequisite tutorials and everything good and bad
  about them and the volume spread across moodle and confluence
updated: 1677684406296
created: 1677668838389
---
**Notes**: I didn't realise some stuff are people's personal notes so ignore comments if that is true, just something of note as I could get to them as a trainee and am just commenting on potential confusion.

# Docker
- [[Docker Notes in Aurius Core Home | https://confluence.apak.com/live/display/AUR/Docker+Notes]] | the major issue is the instructions involved in downloading Docker Desktop, if this was found by trainees they wouldn't know that docker desktop isn't used due to the lack of the required of licensing for it, from our WSL install on terminal its to my understanding
- [[Docker Usage Policy by Rich Ellor | https://confluence.apak.com/live/display/~rich.ellor/Docker]] | can be deleted, only 3 links (Justin's tutorial, pipeline as above, and Docker usage policy - last update **2015**)
- [[Optional Tutorial J (Docker) | https://confluence.apak.com/live/pages/viewpage.action?pageId=88444148]] | Written by Justin and overall very useful, **ESPECIALLY** as a post-app tutorial, qualms only lie in this getting lost when the training app eventually get updated and so the key points of importance are:
    - [[WSL2 setup instructions included as prerequisite | https://confluence.apak.com/live/pages/viewpage.action?pageId=96835920]]
    - Clear end goals for tutorial, well laid out aswell as key points needed to understand, good basics to be covered, would like to retain these for moodle tutorial
    - The *Concise Docker Tutorial* may be the incorrect link as links to a Fireship video which, while useful, doesn't provide information on docker-compose really, but the links to the docker-compose docs are good
    - Step 3 provides useful insight and forces a new understanding of the fact that docker is effectively a self hosted environment and almost its own machine and so makes you understand that the host slightly changes and therefore hibernateContext needs updating to reflect the host in the environment variables of the docker-compose file
- [[Cloud Topics: Docker | https://confluence.apak.com/live/display/WIKI/Cloud+Training%3A+Docker#CloudTraining:Docker-WhatisDocker?]] | *minor* - hyperlinks at the top are superfluous as page is so short, otherwise quite good and adds a little bit to the above optional tutorial
- [[Docker and K8s Practical by Rich Ellor | https://confluence.apak.com/live/display/~rich.ellor/Docker+and+Kubernetes+Practical+Example]] | good comment on common error, well mentioned on *Rancher* being the preferred for Kubernetes (covered below), nice comments on persistent volumes and persistent volume claims

> Not covered: The DevOps tutorial on *Build Pipeline for Docker Images*

# Kubernetes
- [[DevOps Kubernetes | https://confluence.apak.com/live/display/WIKI/Kubernetes]] | generally just seems outdated and could cause confusion, not sure if anything here is useful
- [[Cloud Training: Kubernetes | https://confluence.apak.com/live/display/WIKI/Cloud+Training%3A+Kubernetes#CloudTraining:Kubernetes-KubernetesArchitecture(Highleveloverview)]] | again *minor* links at the top irrelevant for page so small, diagram good to bear in mind
- [[Setup a Local Kubernetes Cluster | https://confluence.apak.com/live/display/WIKI/Setup+a+Local+Kubernetes+Cluster]] | WSL2 setup should be prerequisite to this whole block, don't use minikube we use Rancher (or I've been told its preferred)
- [[Kubernetes Training | https://confluence.apak.com/live/display/WIKI/Kubernetes+Training]] | prerequisits, setup, content links are superfluous. The Training resources are good but don't add much to the documentation outside of bringing the example into a smaller size and a bit more manageable, minor would say that documentation link would be better after these examples as if, like me, you looked at the documentation page and relevant docs where the link is you would have completed the examples provided anyway. Would be useful to consider the addition of ingress' as they make `port-forward` command unecessary
- Kubernetes moodle page: should be setup like the Docker moodle which links to Justin's Optional Tutorial. All of the *"Introduction to X"* in the moodle section for Kubernetes simply takes you to a page that links documentation, and more of them take you to links on Confluence, Kubernetes is more complex than Docker but think it could be brought into one page with slightly decreased complexity for trainees (still fine to link to Nick Bithrey's resources and examples).

> May be useful to have a list of most common commands, maybe some of the more common errors (like not having a PV and PVC and this causing a mixup, or the NEED for a ingress controller with an ingress, and the need for a service when using ingress)

# Helm
The Helm moodle page is quite good and like Kubernetes, due to the substantial increase in complexity from Docker it may be worth splitting it across other pages.

Under the Kubernetes Training page, listed above, there is a section called Helm Package Manager Manifests, this is an amazing startup point for Helm and proves more useful than the documentation on the whole. No issues with this.

# Final Comments
It seems that resources are spread quite far across Confluence (with the exception of the Optional Docker tutorial which covers most, if not all of what you need whilst still providing insight in how to learn and research further). Taking the Docker example into consideration it may be better to condense both Kubernetes and Helm into fewer pages for the purpose of simplicity for a trainee gathering the basics, we need to bear in mind if the purpose of this is post application completion then it is a good idea to emphasise early that the end goal of each stage is to Dockerise, Kubernetes-ise, Helm-ise the application. This is obvious but needs to be maintained on condensation of resources.