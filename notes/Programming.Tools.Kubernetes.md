---
id: hzpowz01h8f3fx6tvawz7kc
title: Kubernetes
desc: ''
updated: 1677058087650
created: 1675855568297
---
## Kubernetes
Kubernetes is an open source platform for managing containerized workloads and services, it is basically a way of managing the complexity of multi-container environments via orchestration.
It helps with *scaling, deployment, automation, and workload management*.
It is also known as *K8s*, it can scale containers across machines and can auto-heal when a container fails. 
A system deployed on Kubernetes is a *cluster* and the brain is called the *control plane* exposing API server handling internal and external requests.
It manages *nodes* (machines) which run *Kubelets* (a tiny application running on the machine to communicate with the main control plane), in nodes are *pods* (containers running together).
It has its own key-value database called *ETCD* which contains info about running the cluster.

## Basics
### Pods
Smallest deployable unit of computing you can create and manage in K8s, it is a group of one or more containers with shared storage and network resources and a specification on how to run them.
They are defined in *yaml* files and run with the command `kubectl apply -f <podFileName.yaml>`.
Generally, pods run a single container but sometimes there is strong coupling and pods can run multiple containers which form a single unit for service.

#### Intro 
1. Create a pod with command `kubectl run nginx-pod --image=nginx`
2. List all running pods `kubectl get pods`
3. Describe pod to get details about pod and its running containers (will also list events that happen at the bottom as a history) `kubectl describe pod nginx-pod`
4. Now expose pod outside the cluster - do this with a port-forward. This will stay running on window until stopped via `Ctrl+C`, and can now be accessed from the browser: `kubectl port-forward pod/nginx-pox 8080:80`
5. Then clean up the resources via `kubectl delete pod nginx-pod`

#### YAML
Its not the ideal solution to use all commands like above, so you can declare kubernetes resourcing through yaml files.
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: nginx-pod
spec:
    containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
```
Key notes: the name is `nginx-pod`, name of the pod and you'll see it when running `get pods` command. The `containers`, the *container for the pod to startup with* (this is a list so there can be multiple containers, not generally used - generally its 1-1).

To run the yaml into the cluster: `kubectl apply -f <podFile>.yaml`
### Deployments
Deployment provides declarative updates for Pods and ReplicaSets.
Can describe a *desired state* in a Deployment, and Deployment Controller changes the actual state to the desired state at controlled rate, can define Deployments to create new ReplicaSets or remove existing Deployments and adopt all resources.<br>
Example deployment:
```yaml
piVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
Similar to pods, its `kubectl apply -f <nameOfDeployment.yaml>`<br>
And then `kubectl get deployments` to see the running deployments<br>
[[More on deployment spec | https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#writing-a-deployment-spec]]


```Bash
alexajones2@ITEM-S134843:/mnt/c/Users/alexajones2/Training/tutorial-2/training (Alex-Jones/dev) $ kubectl get pods
NAME      READY   STATUS         RESTARTS   AGE
app-pod   0/1     ErrImagePull   0          12s
alexajones2@ITEM-S134843:/mnt/c/Users/alexajones2/Training/tutorial-2/training (Alex-Jones/dev) $ docker tag jsf-app:latest localhost:5000/jsf-app
alexajones2@ITEM-S134843:/mnt/c/Users/alexajones2/Training/tutorial-2/training (Alex-Jones/dev) $ docker push localhost:5000/jsf-app
Using default tag: latest
docker-credential-secretservice: error while loading shared libraries: libsecret-1.so.0: cannot open shared object file: No such file or directory
The push refers to repository [localhost:5000/jsf-app]
9d18b9f0f029: Pushed
487bd68d091a: Pushed
9ccda1f6e03d: Pushed
694630803427: Pushed
25f4ff8a1284: Pushed
fa3b6a1ea5c4: Pushed
51566e3f832b: Pushed
51774d97c868: Pushed
ea20c4bf3aae: Pushed
2c8d31157b81: Pushed
7b76d801397d: Pushed
f32868cde90b: Pushed
0db06dff9d9a: Pushed
latest: digest: sha256:0e86d9afca2285e5664c5658c091f6ca52f337e39db013c15eedba67df75deaf size: 3054
alexajones2@ITEM-S134843:/mnt/c/Users/alexajones2/Training/tutorial-2/training (Alex-Jones/dev) $ kubectl get pods
NAME      READY   STATUS         RESTARTS   AGE
app-pod   0/1     ErrImagePull   0          97s
alexajones2@ITEM-S134843:/mnt/c/Users/alexajones2/Training/tutorial-2/training (Alex-Jones/dev) $ kubectl delete pod app-pod
pod "app-pod" deleted
alexajones2@ITEM-S134843:/mnt/c/Users/alexajones2/Training/tutorial-2/training (Alex-Jones/dev) $ kubectl apply -f pod-app.yaml
pod/app-pod created
alexajones2@ITEM-S134843:/mnt/c/Users/alexajones2/Training/tutorial-2/training (Alex-Jones/dev) $ kubectl get pods
NAME      READY   STATUS              RESTARTS   AGE
app-pod   0/1     ContainerCreating   0          4s
alexajones2@ITEM-S134843:/mnt/c/Users/alexajones2/Training/tutorial-2/training (Alex-Jones/dev) $ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
app-pod   1/1     Running   0          10s
```

### Services

Every Pod in a cluster gets its own unique cluster-wide IP address, so you don't need to explicitly link Pods and almost never deal with mapping container ports to host ports.
Creating a clean, backwards-compatible model where Pods can be treated much like VMs or physical hosts from perspectives of port allocation, naming, service discovery, load balancing... etc.

A *service* is an abstract way to expose an application running over a set of Pods as a network service. 
Pods are non-permanent resources so are created and destroyed to match desired cluster state.
In *Kubernetes* a service is a **REST** object like a Pod, you can `POST` a service definition to the API server to create a new instance.


### Ingress
Is effectively meant to do the `kubectl port-forward <resource>/<resource-name> <localport>:<containerPort>` legwork, but doesn't for some stupid reason.

![](pics/deployment-service-ingress-linkup.jpg)
[[Relation of deployment service and ingress | https://dwdraju.medium.com/how-deployment-service-ingress-are-related-in-their-manifest-a2e553cf0ffb]]

Ingress resources MUST always have a controller, with Rancher desktop this defaults to *traefik* [[traefik ingress docs | https://doc.traefik.io/traefik/providers/kubernetes-ingress/]]

### Namespaces
Are a great way to organise clumps of clustered things together so you don't get cluttered kubectl outputs.
To create a new namespace `kubectl create namespace <name>` and can see all namespaces with `kubectl get namespaces`.
These are useful as you can limit resources on given namespaces, but when trying to find any of the resources you must add the flag `-n=<namespace-name>`.

[[Useful google resource | https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-organizing-with-namespaces]]

```bash
alexajones2@ITEM-S134843:/mnt/c/Users/alexajones2/app (Alex-Jones/dev) $ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.184.120:6443
  name: rancher-desktop
contexts:
- context:
    cluster: rancher-desktop
    user: rancher-desktop
  name: rancher-desktop
current-context: rancher-desktop
kind: Config
preferences: {}
users:
- name: rancher-desktop
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
alexajones2@ITEM-S134843:~$ kubectl config get-contexts
CURRENT   NAME              CLUSTER           AUTHINFO          NAMESPACE
*         rancher-desktop   rancher-desktop   rancher-desktop
alexajones2@ITEM-S134843:~$ kubectl config set-context --current --namespace=tomcat
Context "rancher-desktop" modified.
alexajones2@ITEM-S134843:~$ kubectl config get-contexts
CURRENT   NAME              CLUSTER           AUTHINFO          NAMESPACE
*         rancher-desktop   rancher-desktop   rancher-desktop   tomcat
```

To get the shell of a kubernetes pod:
```
` kubectl exec -it <pod-name> -c <container-name> -- /bin/bash`

And to see data in a postgres database (for example):
root@deployment-aj-6549644b7f-wzpw6:/# psql -U postgres -d postgres
psql (11.8 (Debian 11.8-1.pgdg90+1))
Type "help" for help.

postgres=# SLECT * from vehicle
postgres-# SELECT * from vehicle
postgres-# SELECT COUNT(*) FROM vehicle
postgres-# \dt
                 List of relations
 Schema |         Name          | Type  |  Owner
--------+-----------------------+-------+----------
 public | priceband             | table | postgres
 public | pricerecord           | table | postgres
 public | pricerecord_priceband | table | postgres
 public | vehicle               | table | postgres
(4 rows)

postgres-# SELECT * from vehicle
postgres-# \q
```

To convert a PostgreSQL database from a Docker container to an import.sql file, you can use the pg_dump utility inside the container to generate a SQL dump file of your database, and then copy it from the container to your local machine.

Here's how you can do it:

Start a temporary container to generate the SQL dump file:

`docker exec -it <postgresContainer> sh`

`pg_dump -U <username> <database> > dump.sql`

Replace <network> with the name of the Docker network that your PostgreSQL container is running on, <username> with the username for the PostgreSQL database, and <database> with the name of the database you want to convert.

Copy the SQL dump file from the container to your local machine:

`docker cp <container_id>:/dump.sql /path/to/local/directory`

```bash
alexajones2@ITEM-S134843:~$ docker ps
CONTAINER ID   IMAGE                   COMMAND                  CREATED         STATUS         PORTS                                                 NAMES
e65ddc0f9b6d   jsf-app                 "catalina.sh run"        3 minutes ago   Up 3 minutes   8081/tcp, 0.0.0.0:8081->8080/tcp, :::8081->8080/tcp   app_tomcat_1
0dcae9497a21   postgres:11.8           "docker-entrypoint.s…"   3 minutes ago   Up 3 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp             postgres_11_dev
5da85ddd41cd   dpage/pgadmin4:latest   "/entrypoint.sh"         3 minutes ago   Up 3 minutes   443/tcp, 0.0.0.0:9000->80/tcp, :::9000->80/tcp        pgadmin
1cc04faaac57   registry:2              "/entrypoint.sh /etc…"   9 days ago      Up 4 hours     0.0.0.0:5000->5000/tcp, :::5000->5000/tcp             registry
alexajones2@ITEM-S134843:~$ docker cp 0dcae9497a21:/dump.sql /mnt/c/Users/alexajones2/app
```

Replace `<container_id>` with the ID of the temporary container you started in step 1, and `/path/to/local/directory` with the path to the local directory where you want to save the dump.sql file.

Convert the SQL dump file to the import.sql file format. You can use a text editor like Notepad or Sublime Text to do this.

Open the SQL dump file in your text editor and replace the first line (CREATE DATABASE) with the following:

`USE <database>;`

Replace `<database>` with the name of your database.

Save the file as import.sql.


Set up the import.sql as a configmap on its own so it can be used in a deployment
`kubectl create configmap <nameOfMap> --from-file=sql-file=<path_to_sql_file>`
and then from here you can simply reference as needed:
```yaml
    spec:
      restartPolicy: Always
      volumes:
        - name: postgres-11-vol
          configMap:
            name: import.sql
```
But you still have to go into PgAdmin and register the server:
![](2023-02-20-17-39-42.png)<br>
It can be named whatever you want, but you'll have to know this name to call it later, I called I postgres, and the host changed because of a service defined in the deployment yaml, shown below with hostname:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
data:
  DB_HOST: postgres-svc
  DB_USER: postgres
  DB_PASSWORD: welcome
  PGADMIN_DEFAULT_EMAIL: postgres@soprabanking.com
  PGADMIN_DEFAULT_PASSWORD: welcome
  import.sql: |-
    {{ .Files.Get "import.sql" | indent 4 }}
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-svc
  namespace: tomcat
  labels:
    app: app-aj
spec:
  ports:
    - port: 5432
      targetPort: 5432
      protocol: TCP
  selector:
    app: app-aj
---
### postgres ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: postgres-ingress
  namespace: tomcat
  annotations:
    ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: ImplementationSpecific
            backend:
              service:
                name: postgres-svc
                port:
                  number: 5432
```

### Data Storage
Kubernetes supports many types of volumes, a pod can use any number of volume types at once and each container in a pod can access them.
Basically a volume is a directory, which may have data in it and the medium and its contents are determined by the volume type used.

To use a volume, specify the volumes to provide for the Pod in `.spec.volumes` array of the yaml and declare where to mount the volumes into containers in `.spec.containers[*].volumeMounts`.
While you're not able to have volumes within other volumes, you can use the `volumeMounts.subPath` entry in a yaml to specify that data coming from this mount gets stored in a subdirectory of the volume and not the root.

There are also the *PersistentVolume* subsystem which provides an API for users and administrators to abstract details of how storage is provided from how it is used, you must make use of *PersistentVolume* and *PersistentVolumeClaim* which respectfully are:
- a piece of storage in the cluster that has been provisioned, and have a lifecycle independent of any Pod that uses the PV
- a request for storage by a user, PVCs consume PV resources and can be mounted with varying *AccessModes*.

### Configuration and Secrets
#### Best Practices 
- Specify latest stable API version
- Config files should be stored in version control before pushing to the cluster, it allows quick roll backs if necessary
- Write config files in YAML instead of JSON, YAML is more user friendly
- Group related objects into single file, remember can separate with `---`
- Many `kubectl` commands can be called on a directory, you can call `kubectl apply` on a directory of files! (`kubectl apply -f <directory>`)
- Don't use naked Pods (not bound to ReplicaSets or Deployments), they won't be rescheduled in the event of node failure
- Create *Service* before its corresponding backend workloads (Deployments or ReplicaSets)

#### ConfigMaps
API object used to store non-confidential data in key-value pairs, Pods can consume these as *environment variables, command-line args, or config files in a volume*. They allow the decoupling of environment-specific configuration from your container images, allowing portability of applications.
> If you want to store confidential data, make use of a **Secret** rather than a ConfigMap.



## Rancher Desktop
- [[Confluence Practical example | https://confluence.apak.com/live/display/~rich.ellor/Docker+and+Kubernetes+Practical+Example#DockerandKubernetesPracticalExample-Kubernetes(RancherDesktop)]]
- [[Kubernete and Helm | https://confluence.apak.com/live/display/~rich.ellor/Kubernetes]]