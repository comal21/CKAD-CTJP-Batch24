## Multi-container

### Task 1: Sidecar container
```
vi sidecar.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
spec:
  containers:
  - name: main-container
    image: nginx:latest
    ports:
    - containerPort: 80
    # Main application container

  - name: sidecar-container
    image: debian:latest
    command: ['sh', '-c', 'apt update && apt install curl -y && while true; do echo "Sidecar Running"; sleep 10; done']
    # Sidecar container
```
```	
kubectl apply -f sidecar.yaml
```
```
kubectl get pod
```
```
kubectl exec -it sidecar-pod -c sidecar-container -- sh
```
```
curl localhost
```
```
kubectl exec -it sidecar-pod -c main-container -- sh
```
```
curl
```
```
kubectl delete -f sidecar.yaml
```
### Task 1.2: SideCar Container - SELF EXERCISE
```
vi sidecareg2.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: multi-pod
  name: multi-pod
spec:
  containers:
  - image: nginx
    name: container-1
    ports:
    - containerPort: 80
  - name: container-2
    image: busybox
    args: 
    - "/bin/sh"
    - "-c"
    - "sleep 3000"
```
```	
kubectl apply -f sidecareg2.yaml
```
```
kubectl get pod
```
```
kubectl exec -it multi-pod -c container-1 -- sh
```
```
kubectl exec -it multi-pod -c container-2 -- sh
```
### Task 2: Init container
```
vi init.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  containers:
  - name: main-container
    image: registry.access.redhat.com/ubi8/ubi:latest
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
    # Main application container

  initContainers:
  - name: init-container
    image: registry.access.redhat.com/ubi8/ubi:latest
    command: ['sh', '-c', 'until getent hosts myservice; do echo waiting for myservice; sleep 2; done;']
    # Init container

```
```	
kubectl apply -f init.yaml
```
```
kubectl get pod
```
```
kubectl describe pod init-pod
```
```
vi svc.yaml
```
```
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
    - protocol: TCP
      port: 80       # Port to expose
      targetPort: 8080  # Port on the pod that the service should forward to
```
```
kubectl apply -f svc.yaml
```
```
kubectl get pod
```
```
kubectl describe pod init-pod
```

### Task 2.1: Init container - SELF EXERCISE
```
vi initctr.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-container-pod
spec:
  containers:
  - name: main-container
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      mountPath: /app
    # Main application container

  initContainers:
  - name: init-container
    image: busybox:latest
    command: ['sh', '-c', 'echo "Init Container Completed" > /work-dir/completed.txt']
    volumeMounts:
    - name: workdir
      mountPath: /work-dir
    # Init container
  volumes:
  - name: workdir
    emptyDir: {}

```
```	
kubectl apply -f initctr.yaml
```
```
kubectl get pod
```
```
kubectl exec -it init-container-pod -c main-container -- sh
```
```
cd /app && ls
```
```
cat completed.txt
```


