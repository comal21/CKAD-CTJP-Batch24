## ConfigMap & Secrets

### Task 1: Directly inject variables - Traditional Method
```
vi env.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: ws
  name: env-pod
spec:
  containers:
  - image: nginx
    name: ng-ctr
    ports:
    - containerPort: 80
    env:
    - name: db_user   #key
      value: admin
    - name: db_pwd
      value: "1234"
```
```
kubectl apply -f env.yaml
```
```
kubectl describe pod env-pod
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it env-pod -- sh
```
```
echo $db_user
```
```
echo $db_pwd
```
```
env | grep db_
```

### Task 2: Inject `ALL` variables from ConfigMaps(FromLiteral) into POD.
Create a ConfigMap
```
kubectl create cm cm-1 --from-literal=db_user=admin --from-literal=db_pwd=1234
```
```
kubectl get cm
```
```
kubectl describe cm cm-1
```
Inject the ConfigMap into the Pod Yaml File
```
vi cm-env.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web-pod
spec:
  containers:
  - image: httpd
    name: ctr-1
    ports:
    - containerPort: 80
    envFrom:
    - configMapRef:
        name: cm-1
```
```
kubectl apply -f cm-env.yaml
```
```
kubectl describe pod web-pod
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it web-pod -- sh
```
```
echo $db_user
```
```
echo $db_pwd
```
```
env | grep db_
```

### Task 3: Inject `PARTICULAR` variables from ConfigMaps(FromLiteral) into POD.
Create a ConfigMap (If already perform then ignore the below step)
```
kubectl create cm cm-1 --from-literal=db_user=admin --from-literal=db_pwd=1234
```
```
kubectl get cm
```
```
kubectl describe cm cm-1
```
Inject particular variable from the ConfigMap into the Pod Yaml File
```
vi cm2-env.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web-pod-2
spec:
  containers:
  - image: httpd
    name: ctr-1
    ports:
    - containerPort: 80
    env:
    - name: db_password
      valueFrom:
        configMapKeyRef:
          name: cm-1
          key: db_pwd
```
```
kubectl apply -f cm2-env.yaml 
```
```
kubectl describe pod web-pod-2
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it web-pod-2 --  bash
```
```
echo $db_user
```
```
echo $db_pwd
```
```
echo $db_password
```
```
env | grep db_
```

### Task 4 : Injecting ConfigMap as volume mount from file

Create a file
```
vi token
```
```
This is CKAD Training. We are practicing Injecting variables from ConfigMaps(FromFile) into POD.
```
Create a ConfigMap
```
kubectl create cm file-cm --from-file=token        
```
```
kubectl get cm
```
```
kubectl describe cm file-cm
```
Inject as volume mount
```
vi volume-env.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web-pod-4
spec:
  volumes:
  - name: cm-volume
    configMap:
      name: file-cm
  containers:
  - image: httpd
    name: ctr-1
    volumeMounts:
    - name: cm-volume
      mountPath: /app
    ports:
    - containerPort: 80

```
```
kubectl apply -f volume-env.yaml 
```
```
kubectl describe pod web-pod-4
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it web-pod-4 -- bash
```
```
cd /app
```
```
cat token
```
```
exit
```

###  Secret
### Task 1:Imperative
```
kubectl create secret generic secret-1 --from-literal=db_user=admin --from-literal=db_pwd=123
```
```
kubectl get secret
```
```
kubectl describe secret secret-1
```
Declrative
```
vi secret.yaml
```
```
apiVersion: v1
kind: Secret
metadata:
  name: mysql-credentials
type: Opaque
data:
  ##all below values are base64 encoded

  ## To encode and print
  ## echo -n '<value-to-be-encoded>' | base64

  ## To decode and print
  ## echo '<value-to-be-decoded>' | base64 -d

  ##rootpw is root
  rootpw: cm9vdAo=

  ##user is user
  user: dXNlcgo=

  ##password is mypwd
  password: bXlwd2QK
```
```
kubectl apply -f secret.yaml
```
```
kubectl get secrets
```
```
kubectl describe secrets mysql-credentials
```
You can inject the secret in all the three ways as above.

### Task 2: Injecting all values
```
vi sc-pod.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: sc-pod
  name: sc-pod
spec:
  containers:
  - image: nginx
    name: sc-ctr
    envFrom:
    - secretRef:
        name: secret-1
    - secretRef:
        name: mysql-credentials
```
```
kubectl apply -f sc-pod.yaml
```
```
kubectl get po
```
```
kubectl exec -it sc-pod -- sh
```
```
echo $db_user
```
```
echo $db_pwd
```
```
echo $rootpw
```
```
echo $user
```
```
echo $password
```
```
env 
```
### Create a Secret
```
kubectl create secret generic secret-1 --from-literal=db_user=admin --from-literal=db_pwd=123
```
```
kubectl get secret
```
```
kubectl describe secret secret-1
```
### Task 3: Inject PARTICULAR variables from Secret(FromLiteral) into POD
Create secret as shown below
```
vi sc-pod-2.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: sc-pod-2
spec:
  containers:
  - image: httpd
    name: ctr-1
    ports:
    - containerPort: 80
    env:
    - name: db_password   #new-key
      valueFrom:
        secretKeyRef:
          name: secret-1
          key: db_pwd     #old=key
```
Apply the Pod configuration to your 

```
kubectl apply -f sc-pod-2.yaml
```
check the status of pod
```
kubectl get pod
```
Enter into the pod and check if the variable has been passed correctly or not
```
kubectl exec -it sc-pod-2 -- bash
```
```
env | grep db_
```
```
exit
```
### Task 4: Injecting ConfigMap as volume mount
Create a file
```
vi token
```
```
This is CKAD Training. We are practicing Injecting variables from ConfigMaps(FromFile) into POD.
Create a ConfigMap
```
save the file

```
kubectl create secret generic file-secret --from-file=token
```
```
kubectl get secret
```
```
kubectl describe secret file-secret
```
```
kubectl get secret file-secret -o yaml
```

```
vi sc-pod-3.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: sc-pod-3
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: file-secret
  containers:
  - image: httpd
    name: ctr-1
    ports:
    - containerPort: 80
    volumeMounts:
    - name: secret-volume
      mountPath: /tmp/credential/
```                               
Apply the Pod configuration to your 

```
kubectl apply -f sc-pod-3.yaml
```
check the status of pod
```
kubectl get pod
```
Enter into the pod and check if the variable has been passed correctly or not
```
kubectl exec -it sc-pod-3 -- bash
```
```
cd /tmp/credential/
```
```
ls
cat token
```
```
exit
```
