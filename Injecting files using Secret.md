## Injecting Files into Pods using Secrets

In the Image shown below, the contents inside <code>resources</code> are supposed to be injected into the pod. This can be performed using configMap and Volumes, however contents of <code>resources/certs</code> is sensitive and mounting it as a Kubernetes Secret object would be appropriate. This tutorial explains how it can be achieved.

<img src="https://github.com/reusin/kubernetes-tutorial/blob/main/images/folder%20structure.JPG"/>

### 1. Create the kubernetes secret object

Create Secret for <code>my-key-store.jks</code> using <code>--from-file</code> option, which allows creation of secrets from files nad directories.

**Note** : A kubernetes Secret object will have its content encoded in the base64 scheme. However, within the pod, the contents will appear decoded.

```
$ kubectl create secret generic certs --from-file resources/certs
```

### 2. Mounting Secret into a pod

The secrets and configmaps are mounted as volumes into pods as shown in the manifest file, below.

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
        - name: certs-secret
          mountPath: /resources/certs/
        - name: jaas-config
          mountPath: /resources/jaas
        - name: rbac-config
          mountPath: /resources/rbac
      volumes:
      - name: certs-secret
        secret:
          secretName: certs
      - name: jaas-config
        configMap:
          name: jaas
      - name: rbac-config
        configMap:
          name: rbac

```

### 3. Just to be really sure if it works, let's take a look at <code>resources</code> inside the pod.

```
kubectl -it exec pod/<nginx-deployment-pod> -- bash
```

And as expected, we have the configuration files injected into the pod.

<img src="https://github.com/reusin/kubernetes-tutorial/blob/main/images/reallyverifysecret.JPG"/>

<h3>Reference</h3>

[1] https://kubernetes.io/docs/concepts/configuration/secret

