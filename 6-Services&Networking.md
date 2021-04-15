![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/services&empty)
## Services and Networking (13%)
### Practice questions based on these concepts

* Understand Services
* Demonstrate a basic understanding of NetworkPolicies

## Questions
### Create a pod with image nginx called nginx and expose its port 80

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --port=80 --expose
# observe that a pod as well as a service are created
```

</p>
</details>


### Confirm that ClusterIP has been created. Also check endpoints

<details><summary>show</summary>
<p>

```bash
kubectl get svc nginx # services
kubectl get ep # endpoints
```
</p>
</details>

### Get service's ClusterIP, create a temp busybox pod and 'hit' that IP with wget

<details><summary>show</summary>
<p>

```bash
kubectl get svc nginx # get the IP (something like 10.108.93.130)
kubectl run busybox --rm --image=busybox -it --restart=Never -- sh
wget -O- IP:80
exit
```

</p>
or
<p>

```bash
IP=$(kubectl get svc nginx --template={{.spec.clusterIP}}) # get the IP (something like 10.108.93.130)
kubectl run busybox --rm --image=busybox -it --restart=Never --env="IP=$IP" -- wget -O- $IP:80 --timeout 2
# Tip: --timeout is optional, but it helps to get answer more quickly when connection fails (in seconds vs minutes)
```

</p>
</details>

### Convert the ClusterIP to NodePort for the same service and find the NodePort port. Hit service using Node's IP. Delete the service and the pod at the end.

<details><summary>show</summary>
<p>

```bash
kubectl edit svc nginx
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: 2018-06-25T07:55:16Z
  name: nginx
  namespace: default
  resourceVersion: "93442"
  selfLink: /api/v1/namespaces/default/services/nginx
  uid: 191e3dac-784d-11e8-86b1-00155d9f663c
spec:
  clusterIP: 10.97.242.220
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: nginx
  sessionAffinity: None
  type: NodePort # change cluster IP to nodeport
status:
  loadBalancer: {}
```

```bash
kubectl get svc
```

```
# result:
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        1d
nginx        NodePort    10.107.253.138   <none>        80:31931/TCP   3m
```

```bash
wget -O- NODE_IP:31931 # if you're using Kubernetes with Docker for Windows/Mac, try 127.0.0.1
#if you're using minikube, try minikube ip, then get the node ip such as 192.168.99.117
```

```bash
kubectl delete svc nginx # Deletes the service
kubectl delete pod nginx # Deletes the pod
```
</p>
</details>

### Create a deployment called foo using image 'dgkanatsios/simpleapp' (a simple server that returns hostname) and 3 replicas. Label it as 'app=foo'. Declare that containers in this pod will accept traffic on port 8080 (do NOT create a service yet)

<details><summary>show</summary>
<p>

```bash
kubectl create deploy foo --image=dgkanatsios/simpleapp --port=8080 --replicas=3
```
</p>
</details>

### Get the pod IPs. Create a temp busybox pod and try hitting them on port 8080

<details><summary>show</summary>
<p>


```bash
kubectl get pods -l app=foo -o wide # 'wide' will show pod IPs
kubectl run busybox --image=busybox --restart=Never -it --rm -- sh
wget -O- POD_IP:8080 # do not try with pod name, will not work
# try hitting all IPs to confirm that hostname is different
exit
```

</p>
</details>

### Create a service that exposes the deployment on port 6262. Verify its existence, check the endpoints

<details><summary>show</summary>
<p>


```bash
kubectl expose deploy foo --port=6262 --target-port=8080
kubectl get service foo # you will see ClusterIP as well as port 6262
kubectl get endpoints foo # you will see the IPs of the three replica nodes, listening on port 8080
```

</p>
</details>

### Create a temp busybox pod and connect via wget to foo service. Verify that each time there's a different hostname returned. Delete deployment and services to cleanup the cluster

<details><summary>show</summary>
<p>

```bash
kubectl get svc # get the foo service ClusterIP
kubectl run busybox --image=busybox -it --rm --restart=Never -- sh
wget -O- foo:6262 # DNS works! run it many times, you'll see different pods responding
wget -O- SERVICE_CLUSTER_IP:6262 # ClusterIP works as well
# you can also kubectl logs on deployment pods to see the container logs
kubectl delete svc foo
kubectl delete deploy foo
```

</p>
</details>

### Create an nginx deployment of 2 replicas, expose it via a ClusterIP service on port 80. Create a NetworkPolicy so that only pods with labels 'access: granted' can access the deployment and apply it

kubernetes.io > Documentation > Concepts > Services, Load Balancing, and Networking > [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

<details><summary>show</summary>
<p>

```bash
kubectl create deployment nginx --image=nginx --replicas=2
kubectl expose deployment nginx --port=80

kubectl describe svc nginx # see the 'app=nginx' selector for the pods
# or
kubectl get svc nginx -o yaml

vi policy.yaml
```

```YAML
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: access-nginx # pick a name
spec:
  podSelector:
    matchLabels:
      app: nginx # selector for the pods
  ingress: # allow ingress traffic
  - from:
    - podSelector: # from pods
        matchLabels: # with this label
          access: granted
```

```bash
# Create the NetworkPolicy
kubectl create -f policy.yaml

# Check if the Network Policy has been created correctly
# make sure that your cluster's network provider supports Network Policy (https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/#before-you-begin)
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- http://nginx:80 --timeout 2                          # This should not work. --timeout is optional here. But it helps to get answer more quickly (in seconds vs minutes)
kubectl run busybox --image=busybox --rm -it --restart=Never --labels=access=granted -- wget -O- http://nginx:80 --timeout 2  # This should be fine
```

</p>
</details>

### Additonal Practice Questions


<details><summary>Create an nginx pod with a yaml file with label my-nginx and expose the port 80</summary>
<p>

```
kubectl run nginx --image=nginx --restart=Never --port=80 --dry-run -o yaml > nginx.yaml

// edit the label app: my-nginx and create the pod
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    app: my-nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx.yaml
```
</p>
</details>


<details><summary>Create the service for this nginx pod with the pod selector app: my-nginx</summary>
<p>

```
// create the below service
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376

kubectl create -f nginx-svc.yaml
```
</p>
</details>


<details><summary>Find out the label of the pod and verify the service has the same label</summary>
<p>

```
// get the pod with labels
kubectl get po nginx --show-labels

// get the service and chekc the selector column
kubectl get svc my-service -o wide
```
</p>
</details>


<details><summary>Delete the service and create the service with kubectl expose command and verify the label</summary>
<p>

```
// delete the service
kubectl delete svc my-service

// create the service again
kubectl expose po nginx --port=80 --target-port=9376

// verify the label
kubectl get svc -l app=my-nginx
```
</p>
</details>


<details><summary>Delete the service and create the service again with type NodePort</summary>
<p>

```
// delete the service
kubectl delete svc nginx

// create service with expose command
kubectl expose po nginx --port=80 --type=NodePort
```
</p>
</details>


<details><summary>Create the temporary busybox pod and hit the service. Verify the service that it should return the nginx page index.html</summary>
<p>

```
// get the clusterIP from this command
kubectl get svc nginx -o wide

// create temporary busybox to check the nodeport
kubectl run busybox --image=busybox --restart=Never -it --rm -- wget -o- <Cluster IP>:80
```
</p>
</details>


<details><summary>Create a NetworkPolicy which denies all ingress traffic</summary>
<p>

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```
</p>
</details>

## Routing Traffic to Pods from Inside and Outside of a Cluster

1. Create a deployment named `myapp` that creates 2 replicas for Pods with the image `nginx`. Expose the container port 80.
2. Expose the Pods so that requests can be made against the service from inside of the cluster.
3. Create a temporary Pods using the image `busybox` and run a `wget` command against the IP of the service.
4. Change the service type so that the Pods can be reached from outside of the cluster.
5. Run a `wget` command against the service from outside of the cluster.
5. (Optional) Can you expose the Pods as a service without a deployment?

<details><summary>Show Solution</summary>
<p>

Create a deployment with 2 replicas first. You should end up with one deployment and two Pods.

```bash
$ kubectl run myapp --image=nginx --restart=Always --replicas=2 --port=80
deployment.apps/myapp created
$ kubectl get deployments,pods
NAME                          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/myapp   2         2         2            2           59s

NAME                         READY   STATUS    RESTARTS   AGE
pod/myapp-7bc568bfdd-972wg   1/1     Running   0          59s
pod/myapp-7bc568bfdd-l5nmz   1/1     Running   0          59s
```

Expose the service with the type `ClusterIP` and the target port 80.

``` bash
$ kubectl expose deploy myapp --target-port=80
service/myapp exposed
$ kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
myapp        ClusterIP   10.108.88.208   <none>        80/TCP    15s
```

Determine the cluster IP and use it for the `wget` command.

```bash
$ kubectl run tmp --image=busybox --restart=Never -it --rm -- wget -O- 10.108.88.208:80
Connecting to 10.108.88.208:80 (10.108.88.208:80)
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
-                    100% |********************************|   612  0:00:00 ETA
pod "tmp" deleted
```

Turn the type of the service into `NodePort` to expose it outside of the cluster. Now, the service should expose a port in the 30000 range.

```bash
$ kubectl edit service myapp
...
spec:
  type: NodePort
...

kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
myapp        NodePort    10.108.88.208   <none>        80:30441/TCP   3m
```

Run a `wget` or `curl` command against the service using port `30441`. On Docker for Windows/Mac you may have to use localhost or 127.0.0.1 (see [issue](https://github.com/docker/for-win/issues/1950)).

```bash
$ wget -O- localhost:30441
--2019-05-10 16:32:35--  http://localhost:30441/
Resolving localhost (localhost)... ::1, 127.0.0.1
Connecting to localhost (localhost)|::1|:30441... connected.
HTTP request sent, awaiting response... 200 OK
Length: 612 [text/html]
Saving to: ‘STDOUT’

-                                          0%[                                                                                   ]       0  --.-KB/s               <!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
-                                        100%[==================================================================================>]     612  --.-KB/s    in 0s

2019-05-10 16:32:35 (24.3 MB/s) - written to stdout [612/612]
```

</p>
</details>

## Restricting Access to and from a Pod

Let's assume we are working on an application stack that defines three different layers: a frontend, a backend and a database. Each of the layers runs in a Pod. You can find the definition in the YAML file `app-stack.yaml`. The application needs to run in the namespace `app-stack`.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: frontend
  namespace: app-stack
  labels:
    app: todo
    tier: frontend
spec:
  containers:
    - name: frontend
      image: nginx

---

kind: Pod
apiVersion: v1
metadata:
  name: backend
  namespace: app-stack
  labels:
    app: todo
    tier: backend
spec:
  containers:
    - name: backend
      image: nginx

---

kind: Pod
apiVersion: v1
metadata:
  name: database
  namespace: app-stack
  labels:
    app: todo
    tier: database
spec:
  containers:
    - name: database
      image: mysql
      env:
      - name: MYSQL_ROOT_PASSWORD
        value: example
```

1. Create the required namespace.
2. Copy the Pod definition to the file `app-stack.yaml` and create all three Pods. Notice that the namespace has already been defined in the YAML definition.
3. Create a network policy in the YAML file `app-stack-network-policy.yaml`.
4. The network policy should allow incoming traffic from the backend to the database but disallow incoming traffic from the frontend.
5. Incoming traffic to the database should only be allowed on TCP port 3306 and no other port.

<details><summary>Show Solution</summary>
<p>

Create the namespace 

```bash
$ kubectl create namespace app-stack
namespace/app-stack created

$ vim app-stack.yaml
$ kubectl create -f app-stack.yaml
pod/frontend created
pod/backend created
pod/database created

$ kubectl get pods --namespace app-stack
NAME       READY   STATUS    RESTARTS   AGE
backend    1/1     Running   0          22s
database   1/1     Running   0          22s
frontend   1/1     Running   0          22s
```

The following definition ensure that all rules are fulfilled.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-stack-network-policy
  namespace: app-stack
spec:
  podSelector:
    matchLabels:
      app: todo
      tier: database
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: todo
          tier: backend
    ports:
    - protocol: TCP
      port: 3306
```

Create the network policy.

```bash
$ vim app-stack-network-policy.yaml
$ kubectl create -f app-stack-network-policy.yaml
$ kubectl get networkpolicy --namespace app-stack
NAME                       POD-SELECTOR             AGE
app-stack-network-policy   app=todo,tier=database   5s
```

</p>
</details>
