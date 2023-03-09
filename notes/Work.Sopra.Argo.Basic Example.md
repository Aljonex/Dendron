---
id: z1ftqz3cjlnl255lr8whi12
title: Basic Example
desc: 'For trainees who didn't complete the tutorial application a simple example using nginx'
updated: 1678370468049
created: 1678370095776
---
### K8s
Resources used:
- [Deploy nginx on k8s](https://www.nbtechsupport.co.in/2021/06/deploy-nginx-web-server-on-kubernetes.html)
- [Converting a k8s to helm](https://jhooq.com/convert-kubernetes-yaml-into-helm/)

The basic k8s example with nginx, defined in a `deployment.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx-app
    type: front-end
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30011
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-app
    type: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
      type: front-end
  template:
    metadata:
      labels:
        app: nginx-app
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
```

Then using helm in the directory with `helm create <chartName>` can be viewed after creation with `tree <chartName>` to display:
```bash
tree demochart
demochart
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```