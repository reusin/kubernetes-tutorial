## Injecting Files into Pods using Configmaps

In the Image shown below, the contents inside <code>resources</code> are supposed to be injected into the pod. This tutorial explains how it can be achieved using Configmaps

<img src="https://github.com/reusin/kubernetes-tutorial/blob/main/images/folder%20structure.JPG"/>

### 1. Create the configmaps

Create Configmaps for each base folder using <code>--from-file</code> option, which allows creation of configmaps from files and directories.

```
$ kubectl create configmap certs --from-file resources/certs/
$ kubectl create configmap jaas --from-file resources/jaas/
$ kubectl create configmap rbac --from-file resources/rbac/
```

### 2. Mounting configmaps into a pod

The configmaps are mounted as volumes into pods as shown in the manifest file, below.

```
apiVersion: apps/v1
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
        volumeMounts:
        - name: certs-config
          mountPath: /resources/certs
        - name: jaas-config
          mountPath: /resources/jaas
        - name: rbac-config
          mountPath: /resources/rbac
      volumes:
      - name: certs-config
        configMap:
          name: certs
      - name: jaas-config
        configMap:
          name: jaas
      - name: rbac-config
        configMap:
          name: rbac
```

### 3. Run the manifest file

```
kubectl apply -f deployment.yaml
```

### 4. Just to be really sure if it works, let's take a look at <code>resources</code> inside the pod.

```
kubectl -it exec pod/<nginx-deployment-pod> -- bash
```

And as expected, we have the configuration files injected into the pod.

<img src="https://github.com/reusin/kubernetes-tutorial/blob/main/images/reallyverify.JPG"/>

<h3>Reference</h3>

[1] https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap

