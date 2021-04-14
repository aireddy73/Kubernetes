![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/configuration&empty)
## Configuration (18%)
### Practice questions based on these concepts

* Understand ConfigMaps
* Understand SecurityContexts
* Define an application’s resource requirements
* Create & Consume Secrets
* Understand ServiceAccounts

## ConfigMaps

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

### Create a configmap named config with values foo=lala,foo2=lolo

<details><summary>show</summary>
<p>

```bash
kubectl create configmap config --from-literal=foo=lala --from-literal=foo2=lolo
```

</p>
</details>

### Display its values

<details><summary>show</summary>
<p>

```bash
kubectl get cm config -o yaml
# or
kubectl describe cm config
```

</p>
</details>

### Create and display a configmap from a file

Create the file with

```bash
echo -e "foo3=lili\nfoo4=lele" > config.txt
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm configmap2 --from-file=config.txt
kubectl get cm configmap2 -o yaml
```

</p>
</details>

### Create and display a configmap from a .env file

Create the file with the command

```bash
echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config.env
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm configmap3 --from-env-file=config.env
kubectl get cm configmap3 -o yaml
```

</p>
</details>

### Create and display a configmap from a file, giving the key 'special'

Create the file with

```bash
echo -e "var3=val3\nvar4=val4" > config4.txt
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm configmap4 --from-file=special=config4.txt
kubectl describe cm configmap4
kubectl get cm configmap4 -o yaml
```

</p>
</details>

### Create a configMap called 'options' with the value var5=val5. Create a new nginx pod that loads the value from variable 'var5' in an env variable called 'option'

<details><summary>show</summary>
<p>

```bash
kubectl create cm options --from-literal=var5=val5
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    env:
    - name: option # name of the env variable
      valueFrom:
        configMapKeyRef:
          name: options # name of config map
          key: var5 # name of the entity in config map
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env | grep option # will show 'option=val5'
```

</p>
</details>

### Create a configMap 'anotherone' with values 'var6=val6', 'var7=val7'. Load this configMap as env variables into a new nginx pod

<details><summary>show</summary>
<p>

```bash
kubectl create configmap anotherone --from-literal=var6=val6 --from-literal=var7=val7
kubectl run --restart=Never nginx --image=nginx -o yaml --dry-run=client > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    envFrom: # different than previous one, that was 'env'
    - configMapRef: # different from the previous one, was 'configMapKeyRef'
        name: anotherone # the name of the config map
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env 
```

</p>
</details>

### Create a configMap 'cmvolume' with values 'var8=val8', 'var9=val9'. Load this as a volume inside an nginx pod on path '/etc/lala'. Create the pod and 'ls' into the '/etc/lala' directory.

<details><summary>show</summary>
<p>

```bash
kubectl create configmap cmvolume --from-literal=var8=val8 --from-literal=var9=val9
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  volumes: # add a volumes list
  - name: myvolume # just a name, you'll reference this in the pods
    configMap:
      name: cmvolume # name of your configmap
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # your volume mounts are listed here
    - name: myvolume # the name that you specified in pod.spec.volumes.name
      mountPath: /etc/lala # the path inside your container
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- /bin/sh
cd /etc/lala
ls # will show var8 var9
cat var8 # will show val8
```

</p>
</details>

## SecurityContext

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

### Create the YAML for an nginx pod that runs with the user ID 101. No need to create the pod

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  securityContext: # insert this line
    runAsUser: 101 # UID for the user
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

</p>
</details>


### Create the YAML for an nginx pod that has the capabilities "NET_ADMIN", "SYS_TIME" added on its single container

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    securityContext: # insert this line
      capabilities: # and this
        add: ["NET_ADMIN", "SYS_TIME"] # this as well
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

</p>
</details>

## Requests and limits

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Assign CPU Resources to Containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)

### Create an nginx pod with requests cpu=100m,memory=256Mi and limits cpu=200m,memory=512Mi

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'
```

</p>
</details>

## Secrets

kubernetes.io > Documentation > Concepts > Configuration > [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

kubernetes.io > Documentation > Tasks > Inject Data Into Applications > [Distribute Credentials Securely Using Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)

### Create a secret called mysecret with the values password=mypass

<details><summary>show</summary>
<p>

```bash
kubectl create secret generic mysecret --from-literal=password=mypass
```

</p>
</details>

### Create a secret called mysecret2 that gets key/value from a file

Create a file called username with the value admin:

```bash
echo -n admin > username
```

<details><summary>show</summary>
<p>

```bash
kubectl create secret generic mysecret2 --from-file=username
```

</p>
</details>

### Get the value of mysecret2

<details><summary>show</summary>
<p>

```bash
kubectl get secret mysecret2 -o yaml
echo YWRtaW4K | base64 -d # on MAC it is -D, which decodes the value and shows 'admin'
```

Alternative:

```bash
kubectl get secret mysecret2 -o jsonpath='{.data.username}{"\n"}' | base64 -d  # on MAC it is -D
```

</p>
</details>

### Create an nginx pod that mounts the secret mysecret2 in a volume on path /etc/foo

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  volumes: # specify the volumes
  - name: foo # this name will be used for reference inside the container
    secret: # we want a secret
      secretName: mysecret2 # name of the secret - this must already exist on pod creation
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # our volume mounts
    - name: foo # name on pod.spec.volumes
      mountPath: /etc/foo #our mount path
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx /bin/bash
ls /etc/foo  # shows username
cat /etc/foo/username # shows admin
```

</p>
</details>

### Delete the pod you just created and mount the variable 'username' from secret mysecret2 onto a new nginx pod in env variable called 'USERNAME'

<details><summary>show</summary>
<p>

```bash
kubectl delete po nginx
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    env: # our env variables
    - name: USERNAME # asked name
      valueFrom:
        secretKeyRef: # secret reference
          name: mysecret2 # our secret's name
          key: username # the key of the data in the secret
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env | grep USERNAME | cut -d '=' -f 2 # will show 'admin'
```

</p>
</details>

## ServiceAccounts

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

### See all the service accounts of the cluster in all namespaces

<details><summary>show</summary>
<p>

```bash
kubectl get sa --all-namespaces
```
Alternatively 

```bash
kubectl get sa -A
```

</p>
</details>

### Create a new serviceaccount called 'myuser'

<details><summary>show</summary>
<p>

```bash
kubectl create sa myuser
```

Alternatively:

```bash
# let's get a template easily
kubectl get sa default -o yaml > sa.yaml
vim sa.yaml
```

```YAML
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myuser
```

```bash
kubectl create -f sa.yaml
```

</p>
</details>

### Create an nginx pod that uses 'myuser' as a service account

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --serviceaccount=myuser -o yaml --dry-run=client > pod.yaml
kubectl apply -f pod.yaml
```

or you can add manually:

```bash
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  serviceAccountName: myuser # we use pod.spec.serviceAccountName
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe pod nginx # will see that a new secret called myuser-token-***** has been mounted
```


</p>
</details>

### Additional Practice Questions



<details><summary>List all the configmaps in the cluster</summary>
<p>
   
```
kubectl get cm
     or
kubectl get configmap
```
</p>
</details>


<details><summary>Create a configmap called myconfigmap with literal value appname=myapp</summary>
<p>
   
```
kubectl create cm myconfigmap --from-literal=appname=myapp
```
</p>
</details>


<details><summary>Verify the configmap we just created has this data</summary>
<p>
   
```
// you will see under data
kubectl get cm -o yaml
         or
kubectl describe cm
```
</p>
</details>


<details><summary>delete the configmap myconfigmap we just created</summary>
<p>
   
```
kubectl delete cm myconfigmap
```
</p>
</details>


<details><summary>Create a file called config.txt with two values key1=value1 and key2=value2 and verify the file</summary>
<p>
   
```
cat >> config.txt << EOF
key1=value1
key2=value2
EOF

cat config.txt
```
</p>
</details>


<details><summary>Create a configmap named keyvalcfgmap and read data from the file config.txt and verify that configmap is created correctly</summary>
<p>
   
```
kubectl create cm keyvalcfgmap --from-file=config.txt

kubectl get cm keyvalcfgmap -o yaml
```
</p>
</details>



<details><summary>Create an nginx pod and load environment values from the above configmap keyvalcfgmap and exec into the pod and verify the environment variables and delete the pod</summary>
<p>
   
```
// first run this command to save the pod yml
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx-pod.yml

// edit the yml to below file and create
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
    envFrom:
    - configMapRef:
        name: keyvalcfgmap
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx-pod.yml

// verify
kubectl exec -it nginx -- env
kubectl delete po nginx
```
</p>
</details>


<details><summary>Create an env file file.env with var1=val1 and create a configmap envcfgmap from this env file and verify the configmap</summary>
<p>
   
```
echo var1=val1 > file.env
cat file.env

kubectl create cm envcfgmap --from-env-file=file.env
kubectl get cm envcfgmap -o yaml --export
```
</p>
</details>


<details><summary>Create an nginx pod and load environment values from the above configmap envcfgmap and exec into the pod and verify the environment variables and delete the pod</summary>
<p>
   
```
// first run this command to save the pod yml
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx-pod.yml

// edit the yml to below file and create
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
    env:
    - name: ENVIRONMENT
      valueFrom:
        configMapKeyRef:
          name: envcfgmap
          key: environment
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx-pod.yml

// verify
kubectl exec -it nginx -- env
kubectl delete po nginx
```
</p>
</details>


<details><summary>Create a configmap called cfgvolume with values var1=val1, var2=val2 and create an nginx pod with volume nginx-volume which reads data from this configmap cfgvolume and put it on the path /etc/cfg</summary>
<p>
   
```
// first create a configmap cfgvolume
kubectl create cm cfgvolume --from-literal=var1=val1 --from-literal=var2=val2

// verify the configmap
kubectl describe cm cfgvolume

// create the config map 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  volumes:
  - name: nginx-volume
    configMap:
      name: cfgvolume
  containers:
  - image: nginx
    name: nginx
    resources: {}
    volumeMounts:
    - name: nginx-volume
      mountPath: /etc/cfg
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx-volume.yml

// exec into the pod
kubectl exec -it nginx -- /bin/sh

// check the path
cd /etc/cfg
ls
```
</p>
</details>


<details><summary>Create a pod called secbusybox with the image busybox which executes command sleep 3600 and makes sure any Containers in the Pod, all processes run with user ID 1000 and with group id 2000 and verify.</summary>
<p>
   
```
// create yml file with dry-run
kubectl run secbusybox --image=busybox --restart=Never --dry-run -o yaml -- /bin/sh -c "sleep 3600;" > busybox.yml

// edit the pod like below and create
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secbusybox
  name: secbusybox
spec:
  securityContext: # add security context
    runAsUser: 1000
    runAsGroup: 2000
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600;
    image: busybox
    name: secbusybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f busybox.yml

// verify
kubectl exec -it secbusybox -- sh
id // it will show the id and group
```
</p>
</details>


<details><summary>Create the same pod as above this time set the securityContext for the container as well and verify that the securityContext of container overrides the Pod level securityContext.</summary>
<p>
   
```
// create yml file with dry-run
kubectl run secbusybox --image=busybox --restart=Never --dry-run -o yaml -- /bin/sh -c "sleep 3600;" > busybox.yml

// edit the pod like below and create
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secbusybox
  name: secbusybox
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600;
    image: busybox
    securityContext:
      runAsUser: 2000
    name: secbusybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f busybox.yml

// verify
kubectl exec -it secbusybox -- sh
id // you can see container securityContext overides the Pod level
```
</p>
</details>


<details><summary>Create pod with an nginx image and configure the pod with capabilities NET_ADMIN and SYS_TIME verify the capabilities</summary>
<p>
   
```
// create the yaml file
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// edit as below and create pod
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    securityContext:
      capabilities:
        add: ["SYS_TIME", "NET_ADMIN"]
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx.yml

// exec and verify
kubectl exec -it nginx -- sh
cd /proc/1
cat status

// you should see these values
CapPrm: 00000000aa0435fb
CapEff: 00000000aa0435fb
```
</p>
</details>


<details><summary>Create a Pod nginx and specify a memory request and a memory limit of 100Mi and 200Mi respectively.</summary>
<p>
   
```
// create a yml file
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add the resources section and create
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: 
      requests:
        memory: "100Mi"
      limits:
        memory: "200Mi"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx.yml

// verify
kubectl top pod
```
</p>
</details>


<details><summary>Create a Pod nginx and specify a CPU request and a CPU limit of 0.5 and 1 respectively.</summary>
<p>
   
```
// create a yml file
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add the resources section and create
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      requests:
        cpu: "0.5"
      limits:
        cpu: "1"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx.yml

// verify
kubectl top pod
```
</p>
</details>


<details><summary>Create a Pod nginx and specify both CPU, memory requests and limits together and verify.</summary>
<p>
   
```
// create a yml file
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add the resources section and create
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      requests:
        memory: "100Mi"
        cpu: "0.5"
      limits:
        memory: "200Mi"
        cpu: "1"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx.yml

// verify
kubectl top pod
```
</p>
</details>


<details><summary>Create a Pod nginx and specify a memory request and a memory limit of 100Gi and 200Gi respectively which is too big for the nodes and verify pod fails to start because of insufficient memory</summary>
<p>
   
```
// create a yml file
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add the resources section and create
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      requests:
        memory: "100Gi"
        cpu: "0.5"
      limits:
        memory: "200Gi"
        cpu: "1"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx.yml

// verify
kubectl describe po nginx // you can see pending state
```
</p>
</details>


<details><summary>Create a secret mysecret with values user=myuser and password=mypassword</summary>
<p>
   
```
kubectl create secret generic my-secret --from-literal=username=user --from-literal=password=mypassword
```
</p>
</details>


<details><summary>List the secrets in all namespaces</summary>
<p>
   
```
kubectl get secret --all-namespaces
```
</p>
</details>


<details><summary>Output the yaml of the secret created above</summary>
<p>
   
```
kubectl get secret my-secret -o yaml
```
</p>
</details>


<details><summary>Create an nginx pod which reads username as the environment variable</summary>
<p>
   
```
// create a yml file
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add env section below and create
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    env:
    - name: USER_NAME
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: username
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx.yml

//verify
kubectl exec -it nginx -- env
```
</p>
</details>


<details><summary>Create an nginx pod which loads the secret as environment variables</summary>
<p>
   
```
// create a yml file
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add env section below and create
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    envFrom:
    - secretRef:
        name: my-secret
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx.yml

//verify
kubectl exec -it nginx -- env
```
</p>
</details>


<details><summary>List all the service accounts in the default namespace</summary>
<p>
   
```
kubectl get sa
```
</p>
</details>


<details><summary>List all the service accounts in all namespaces</summary>
<p>
   
```
kubectl get sa --all-namespaces
```
</p>
</details>


<details><summary>Create a service account called admin</summary>
<p>
   
```
kubectl create sa admin
```
</p>
</details>


<details><summary>Output the YAML file for the service account we just created</summary>
<p>
   
```
kubectl get sa admin -o yaml
```
</p>
</details>


<details><summary>Create a busybox pod which executes this command sleep 3600 with the service account admin and verify</summary>
<p>
   
```
kubectl run busybox --image=busybox --restart=Never --dry-run -o yaml -- /bin/sh -c "sleep 3600" > busybox.yml

kubectl create -f busybox.yml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  serviceAccountName: admin
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

// verify
kubectl describe po busybox
```
</p>
</details>

## Configuring a Pod to Use a ConfigMap

1. Create a new file named `config.txt` with the following environment variables as key/value pairs on each line.

- `DB_URL` equates to `localhost:3306`
- `DB_USERNAME` equates to `postgres`

2. Create a new ConfigMap named `db-config` from that file.
3. Create a Pod named `backend` that uses the environment variables from the ConfigMap and runs the container with the image `nginx`.
4. Shell into the Pod and print out the created environment variables. You should find `DB_URL` and `DB_USERNAME` with their appropriate values.

<details><summary>Show Solution</summary>
<p>

Create the environment variables in the text file.

```bash
$ echo -e "DB_URL=localhost:3306\nDB_USERNAME=postgres" > config.txt
```

Create the ConfigMap and point to the text file upon creation.

```bash
$ kubectl create configmap db-config --from-env-file=config.txt
configmap/db-config created
$ kubectl run backend --image=nginx --restart=Never -o yaml --dry-run > pod.yaml
```

The final YAML file should look similar to the following code snippet.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: backend
  name: backend
spec:
  containers:
  - image: nginx
    name: backend
    envFrom:
      - configMapRef:
          name: db-config
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

Create the Pod by pointing the `create` command to the YAML file.

```bash
$ kubectl create -f pod.yaml
```

Log into the Pod and run the `env` command.

```bash
$ kubectl exec backend -it -- /bin/sh
/ # env
DB_URL=localhost:3306
DB_USERNAME=postgres
...
/ # exit
```

</p>
</details>

## Configuring a Pod to Use a Secret

1. Create a new Secret named `db-credentials` with the key/value pair `db-password=passwd`.
2. Create a Pod named `backend` that defines uses the Secret as environment variable named `DB_PASSWORD` and runs the container with the image `nginx`.
3. Shell into the Pod and print out the created environment variables. You should find `DB_PASSWORD` variable.

<details><summary>Show Solution</summary>
<p>

It's easy to create the secret from the command line. Furthermore, execute the `run` command to generate the YAML file for the Pod.

```bash
$ kubectl create secret generic db-credentials --from-literal=db-password=passwd
secret/db-credentials created
$ kubectl get secrets
NAME              TYPE      DATA   AGE
db-credentials    Opaque    1      26s
$ kubectl run backend --image=nginx --restart=Never -o yaml --dry-run > pod.yaml
```

Edit the YAML file and create an environment that reads the relevant key from the secret.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: backend
  name: backend
spec:
  containers:
  - image: nginx
    name: backend
    env:
      - name: DB_PASSWORD
        valueFrom:
          secretKeyRef:
            name: db-credentials
            key: db-password
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

Create the Pod by pointing the `create` command to the YAML file.

```bash
$ kubectl create -f pod.yaml
```

You can find the environment variable by shelling into the container and running the `env` command.

```
$ kubectl exec -it backend -- /bin/sh
/ # env
DB_PASSWORD=passwd
/ # exit
```

</p>
</details>

## Creating a Security Context for a Pod

1. Create a Pod named `secured` that uses the image `nginx` for a single container. Mount an `emptyDir` volume to the directory `/data/app`.
2. Files created on the volume should use the filesystem group ID 3000.
3. Get a shell to the running container and create a new file named `logs.txt` in the directory `/data/app`. List the contents of the directory and write them down.

<details><summary>Show Solution</summary>
<p>

Start by creating the Pod definition as YAML file.

```bash
$ kubectl run secured --image=nginx --restart=Never -o yaml --dry-run > secured.yaml
```

Edit the YAML file, add a volume and a volume mount. Add a security context with the relevant group ID.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secured
  name: secured
spec:
  securityContext:
    fsGroup: 3000
  containers:
  - image: nginx
    name: secured
    volumeMounts:
    - name: data-vol
      mountPath: /data/app
    resources: {}
  volumes:
  - name: data-vol
    emptyDir: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

Create the Pod and log into the container. Create the file in the directory of the volume mount. The group ID should be 3000 as defined by the security context.

```bash
$ kubectl create -f secured.yaml
pod/secured created
$ kubectl exec -it secured -- sh
/ # cd /data/app
/ # touch logs.txt
/ # ls -l
-rw-r--r-- 1 root 3000 0 Mar 11 15:56 logs.txt
/ # exit
```

</p>
</details>

## Defining a Pod’s Resource Requirements

Create a resource quota named `apps` under the namespace `rq-demo` using the following YAML definition in the file `rq.yaml`.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: app
spec:
  hard:
    pods: "2"
    requests.cpu: "2"
    requests.memory: 500m
```

1. Create a new Pod that exceeds the limits of the resource quota requirements. Write down the error message.
2. Change the request limits to fulfill the requirements to ensure that the Pod could be created successfully. Write down the output of the command that renders the used amount of resources for the namespace.

<details><summary>Show Solution</summary>
<p>

First create the namespace and the resource quota in the namespace.

```bash
$ kubectl create namespace rq-demo
$ kubectl create -f rq.yaml --namespace=rq-demo
resourcequota/app created
$ kubectl describe quota --namespace=rq-demo
Name:            app
Namespace:       rq-demo
Resource         Used  Hard
--------         ----  ----
pods             0     2
requests.cpu     0     2
requests.memory  0     500m
```

Next, create the YAML file named `pod.yaml` with more requested memory than available in the quota.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mypod
  name: mypod
spec:
  containers:
  - image: nginx
    name: mypod
    resources:
      requests:
        memory: "1G"
        cpu: "400m"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

Create the Pod and observe the error message.

```bash
$ kubectl create -f pod.yaml --namespace=rq-demo
Error from server (Forbidden): error when creating "pod.yaml": pods "mypod" is forbidden: exceeded quota: app, requested: requests.memory=1G, used: requests.memory=0, limited: requests.memory=500m
```

Lower the memory settings to less than `500m` (e.g. `200m`) and create the Pod.

```bash
$ kubectl create -f pod.yaml --namespace=rq-demo
pod/mypod created
$ kubectl describe quota --namespace=rq-demo
Name:            app
Namespace:       rq-demo
Resource         Used  Hard
--------         ----  ----
pods             1     2
requests.cpu     400m  2
requests.memory  200m  500m
```

</p>
</details>

## Using a Service Account

1. Create a new service account named `backend-team`.
2. Print out the token for the service account in YAML format.
3. Create a Pod named `backend` that uses the image `nginx` and the identity `backend-team` for running processes.
4. Get a shell to the running container and print out the token of the service account.

<details><summary>Show Solution</summary>
<p>

First, create the service acccount and inspect it.

```bash
$ kubectl create serviceaccount backend-team
serviceaccount/backend-team created
$ kubectl get serviceaccount backend-team -o yaml --export
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: 2019-05-09T22:43:54Z
  name: backend-team
  namespace: default
  resourceVersion: "1888067"
  selfLink: /api/v1/namespaces/default/serviceaccounts/backend-team
  uid: ecd3b7ea-72ab-11e9-96c5-025000000001
secrets:
- name: backend-team-token-hskch
```

Next, you can create a new Pod and assign the service account to it.

```bash
$ kubectl run backend --image=nginx --restart=Never --serviceaccount=backend-team
```

You can print out the token from the volume source at `/var/run/secrets/kubernetes.io/serviceaccount`.

```bash
$ kubectl exec -it backend -- /bin/sh
/ # cat /var/run/secrets/kubernetes.io/serviceaccount/token
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImJhY2tlbmQtdGVhbS10b2tlbi1kbTJmZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJiYWNrZW5kLXRlYW0iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIxNzM0MzVjMS00NDJmLTExZTktOGRjMy0wMjUwMDAwMDAwMDEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpiYWNrZW5kLXRlYW0ifQ.DjWUxEMNUmQVoXd4b-eIjxboj3w3k7hS5hfV8mm8eoEPz3HJJMgjIpAaurcvo1pp2Ggpd1kIhQvfRqI6-u57f80N5UqXt_qATJfonat2NNXX8pXmFNoPig9LB-pbo8TN_pYGWNworXsxmK9w6V9eaRosIinRp0u-cvijQbsBw3lxWgGo9S4G-7f19mMKN1Pg2xS2J6fKX9IKvhHrUkM91nwcwmsO0use5B4TGbuRa9METiGsfEpegvzMPBbPl0B_T1ANH_pck0LFNtvKe0g1v5zpKx2lRF9WdFAqPsG7BJ1dEH88JtBHzD59OhxIPqtyT4sXKjACBN_ka5ZADMzPJg
```

</p>
</details>
