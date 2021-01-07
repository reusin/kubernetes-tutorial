## Injecting Files into Pods using Volumes

Files can be injected into pods using Persistent Volumes and Persistent Volume Claims.
In the Image shown below, the contents inside <code>resources</code> are required to be injected into the pods. This tutorial explains how it can be achieved using Persistent Volume and Persistent Volume Claims.

<img src="https://github.com/reusin/kubernetes-tutorial/blob/main/images/folder%20structure.JPG"/>

### 1. Create a Persistent Volume

A Persistent Volume object must be created with a path which contains the files required to be injected.
For this demonstration, a Persistent Volume of <code>local</code> type will be used. 

<b>Note</b>: Persistent Volumes with <code>hostPath</code> shouldn't be used for production as it only works for a single node cluster, however a <code>local</code> Persistent Volume can be used together with the <code>nodeAffinity</code> option.

More about types of Persistent Volumes can be found <a href="https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes">here</a>

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: volume
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /home/sshfromputty/kpow-local/resources
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - hostname
  ```
  
 ### 2. Create a Persistent Volume Claim to bind with the Persistent Volume
 
A Persistent Volume Claim binds to the Persistent Volume, and can be accessed by multiple pods. If the cluster has dynamic volume provisioning enabled, assign the appropriate <code>storageClassName</code> as defined by the Cluster Administrator to dynamically have a volume bound to the Persistent Volume Claim. 
 
 ```
 apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: volume-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
 ```
 
### 3. Mount Persistent Volume Claim to Pods

For this demonstration, the Persistent Volume Claim is mounted as a volume to pods running Nginx image in the deployment manifest.
 
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
        - name: resources
          mountPath: /resources
      volumes:
      - name: resources
        persistentVolumeClaim:
          claimName: volume-claim
 ```
 
### 4. Just to be really sure if it works, let's take a look at <code>resources</code> inside the pod.
 
The interactive terminal in the Pod can be accessed, to take a look at the resources which were injected.
 
```
kubectl -it exec pod/nginx-deployment-pod -- bash
```
 
<img src="https://github.com/reusin/kubernetes-tutorial/blob/main/images/reallyverifyvolume.JPG"/>
  
### Reference
[1] https://kubernetes.io/docs/concepts/storage/persistent-volumes
