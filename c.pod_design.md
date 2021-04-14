![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/pod_design&empty)
## Pod design (20%)
### Practice questions based on these concepts

* Understand how to use Labels, Selectors and Annotations
* Understand Deployments and how to perform rolling updates
* Understand Deployments and how to perform rollbacks
* Understand Jobs and CronJobs
## Labels and annotations
kubernetes.io > Documentation > Concepts > Overview > [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors)

### Create 3 pods with names nginx1,nginx2,nginx3. All of them should have the label app=v1

<details><summary>show</summary>
<p>

```bash
kubectl run nginx1 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx2 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx3 --image=nginx --restart=Never --labels=app=v1
```

</p>
</details>

### Show all labels of the pods

<details><summary>show</summary>
<p>

```bash
kubectl get po --show-labels
```

</p>
</details>

### Change the labels of pod 'nginx2' to be app=v2

<details><summary>show</summary>
<p>

```bash
kubectl label po nginx2 app=v2 --overwrite
```

</p>
</details>

### Get the label 'app' for the pods (show a column with APP labels)

<details><summary>show</summary>
<p>

```bash
kubectl get po -L app
# or
kubectl get po --label-columns=app
```

</p>
</details>

### Get only the 'app=v2' pods

<details><summary>show</summary>
<p>

```bash
kubectl get po -l app=v2
# or
kubectl get po -l 'app in (v2)'
# or
kubectl get po --selector=app=v2
```

</p>
</details>

### Remove the 'app' label from the pods we created before

<details><summary>show</summary>
<p>

```bash
kubectl label po nginx1 nginx2 nginx3 app-
# or
kubectl label po nginx{1..3} app-
# or
kubectl label po -l app app-
```

</p>
</details>

### Create a pod that will be deployed to a Node that has the label 'accelerator=nvidia-tesla-p100'

<details><summary>show</summary>
<p>

Add the label to a node:

```bash
kubectl label nodes <your-node-name> accelerator=nvidia-tesla-p100
kubectl get nodes --show-labels
```

We can use the 'nodeSelector' property on the Pod YAML:

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
  nodeSelector: # add this
    accelerator: nvidia-tesla-p100 # the selection label
```

You can easily find out where in the YAML it should be placed by:

```bash
kubectl explain po.spec
```

OR:
Use node affinity (https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/#schedule-a-pod-using-required-node-affinity)

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: accelerator
            operator: In
            values:
            - nvidia-tesla-p100
  containers:
    ...
```

</p>
</details>

### Annotate pods nginx1, nginx2, nginx3 with "description='my description'" value

<details><summary>show</summary>
<p>


```bash
kubectl annotate po nginx1 nginx2 nginx3 description='my description'

#or

kubectl annotate po nginx{1..3} description='my description'
```

</p>
</details>

### Check the annotations for pod nginx1

<details><summary>show</summary>
<p>

```bash
kubectl describe po nginx1 | grep -i 'annotations'

# or

kubectl get po nginx1 -o custom-columns=Name:metadata.name,ANNOTATIONS:metadata.annotations.description
```

As an alternative to using `| grep` you can use jsonPath like `kubectl get po nginx1 -o jsonpath='{.metadata.annotations}{"\n"}'`

</p>
</details>

### Remove the annotations for these three pods

<details><summary>show</summary>
<p>

```bash
kubectl annotate po nginx{1..3} description-
```

</p>
</details>

### Remove these pods to have a clean state in your cluster

<details><summary>show</summary>
<p>

```bash
kubectl delete po nginx{1..3}
```

</p>
</details>

## Deployments

kubernetes.io > Documentation > Concepts > Workloads > Controllers > [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment)

### Create a deployment with image nginx:1.7.8, called nginx, having 2 replicas, defining port 80 as the port that this container exposes (don't create a service for this deployment)

<details><summary>show</summary>
<p>

```bash
kubectl create deployment nginx  --image=nginx:1.7.8  --dry-run=client -o yaml > deploy.yaml
vi deploy.yaml
# change the replicas field from 1 to 2
# add this section to the container spec and save the deploy.yaml file
# ports:
#   - containerPort: 80
kubectl apply -f deploy.yaml
```

or, do something like:

```bash
kubectl create deployment nginx  --image=nginx:1.7.8  --dry-run=client -o yaml | sed 's/replicas: 1/replicas: 2/g'  | sed 's/image: nginx:1.7.8/image: nginx:1.7.8\n        ports:\n        - containerPort: 80/g' | kubectl apply -f -
```

or, 
```bash
kubectl create deploy nginx --image=nginx:1.7.8 --replicas=2 --port=80
```

</p>
</details>

### View the YAML of this deployment

<details><summary>show</summary>
<p>

```bash
kubectl get deploy nginx -o yaml
```

</p>
</details>

### View the YAML of the replica set that was created by this deployment

<details><summary>show</summary>
<p>

```bash
kubectl describe deploy nginx # you'll see the name of the replica set on the Events section and in the 'NewReplicaSet' property
# OR you can find rs directly by:
kubectl get rs -l run=nginx # if you created deployment by 'run' command
kubectl get rs -l app=nginx # if you created deployment by 'create' command
# you could also just do kubectl get rs
kubectl get rs nginx-7bf7478b77 -o yaml
```

</p>
</details>

### Get the YAML for one of the pods

<details><summary>show</summary>
<p>

```bash
kubectl get po # get all the pods
# OR you can find pods directly by:
kubectl get po -l run=nginx # if you created deployment by 'run' command
kubectl get po -l app=nginx # if you created deployment by 'create' command
kubectl get po nginx-7bf7478b77-gjzp8 -o yaml
```

</p>
</details>

### Check how the deployment rollout is going

<details><summary>show</summary>
<p>

```bash
kubectl rollout status deploy nginx
```

</p>
</details>

### Update the nginx image to nginx:1.7.9

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.7.9
# alternatively...
kubectl edit deploy nginx # change the .spec.template.spec.containers[0].image
```

The syntax of the 'kubectl set image' command is `kubectl set image (-f FILENAME | TYPE NAME) CONTAINER_NAME_1=CONTAINER_IMAGE_1 ... CONTAINER_NAME_N=CONTAINER_IMAGE_N [options]`

</p>
</details>

### Check the rollout history and confirm that the replicas are OK

<details><summary>show</summary>
<p>

```bash
kubectl rollout history deploy nginx
kubectl get deploy nginx
kubectl get rs # check that a new replica set has been created
kubectl get po
```

</p>
</details>

### Undo the latest rollout and verify that new pods have the old image (nginx:1.7.8)

<details><summary>show</summary>
<p>

```bash
kubectl rollout undo deploy nginx
# wait a bit
kubectl get po # select one 'Running' Pod
kubectl describe po nginx-5ff4457d65-nslcl | grep -i image # should be nginx:1.7.8
```

</p>
</details>

### Do an on purpose update of the deployment with a wrong image nginx:1.91

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.91
# or
kubectl edit deploy nginx
# change the image to nginx:1.91
# vim tip: type (without quotes) '/image' and Enter, to navigate quickly
```

</p>
</details>

### Verify that something's wrong with the rollout

<details><summary>show</summary>
<p>

```bash
kubectl rollout status deploy nginx
# or
kubectl get po # you'll see 'ErrImagePull'
```

</p>
</details>


### Return the deployment to the second revision (number 2) and verify the image is nginx:1.7.9

<details><summary>show</summary>
<p>

```bash
kubectl rollout undo deploy nginx --to-revision=2
kubectl describe deploy nginx | grep Image:
kubectl rollout status deploy nginx # Everything should be OK
```

</p>
</details>

### Check the details of the fourth revision (number 4)

<details><summary>show</summary>
<p>

```bash
kubectl rollout history deploy nginx --revision=4 # You'll also see the wrong image displayed here
```

</p>
</details>

### Scale the deployment to 5 replicas

<details><summary>show</summary>
<p>

```bash
kubectl scale deploy nginx --replicas=5
kubectl get po
kubectl describe deploy nginx
```

</p>
</details>

### Autoscale the deployment, pods between 5 and 10, targetting CPU utilization at 80%

<details><summary>show</summary>
<p>

```bash
kubectl autoscale deploy nginx --min=5 --max=10 --cpu-percent=80
```

</p>
</details>

### Pause the rollout of the deployment

<details><summary>show</summary>
<p>

```bash
kubectl rollout pause deploy nginx
```

</p>
</details>

### Update the image to nginx:1.9.1 and check that there's nothing going on, since we paused the rollout

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.9.1
# or
kubectl edit deploy nginx
# change the image to nginx:1.9.1
kubectl rollout history deploy nginx # no new revision
```

</p>
</details>

### Resume the rollout and check that the nginx:1.9.1 image has been applied

<details><summary>show</summary>
<p>

```bash
kubectl rollout resume deploy nginx
kubectl rollout history deploy nginx
kubectl rollout history deploy nginx --revision=6 # insert the number of your latest revision
```

</p>
</details>

### Delete the deployment and the horizontal pod autoscaler you created

<details><summary>show</summary>
<p>

```bash
kubectl delete deploy nginx
kubectl delete hpa nginx

#Or
kubectl delete deploy/nginx hpa/nginx
```
</p>
</details>

## Jobs

### Create a job named pi with image perl that runs the command with arguments "perl -Mbignum=bpi -wle 'print bpi(2000)'"

<details><summary>show</summary>
<p>

```bash
kubectl create job pi  --image=perl -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```

</p>
</details>

### Wait till it's done, get the output

<details><summary>show</summary>
<p>

```bash
kubectl get jobs -w # wait till 'SUCCESSFUL' is 1 (will take some time, perl image might be big)
kubectl get po # get the pod name
kubectl logs pi-**** # get the pi numbers
kubectl delete job pi
```
OR 

```bash
kubectl get jobs -w # wait till 'SUCCESSFUL' is 1 (will take some time, perl image might be big)
kubectl logs job/pi
kubectl delete job pi
```

</p>
</details>

### Create a job with the image busybox that executes the command 'echo hello;sleep 30;echo world'

<details><summary>show</summary>
<p>

```bash
kubectl create job busybox --image=busybox -- /bin/sh -c 'echo hello;sleep 30;echo world'
```

</p>
</details>

### Follow the logs for the pod (you'll wait for 30 seconds)

<details><summary>show</summary>
<p>

```bash
kubectl get po # find the job pod
kubectl logs busybox-ptx58 -f # follow the logs
```

</p>
</details>

### See the status of the job, describe it and see the logs

<details><summary>show</summary>
<p>

```bash
kubectl get jobs
kubectl describe jobs busybox
kubectl logs job/busybox
```

</p>
</details>

### Delete the job

<details><summary>show</summary>
<p>

```bash
kubectl delete job busybox
```

</p>
</details>

### Create a job but ensure that it will be automatically terminated by kubernetes if it takes more than 30 seconds to execute

<details><summary>show</summary>
<p>
  
```bash
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'while true; do echo hello; sleep 10;done' > job.yaml
vi job.yaml
```
  
Add job.spec.activeDeadlineSeconds=30

```bash
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  activeDeadlineSeconds: 30 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - while true; do echo hello; sleep 10;done
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```
</p>
</details>

### Create the same job, make it run 5 times, one after the other. Verify its status and delete it

<details><summary>show</summary>
<p>

```bash
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'echo hello;sleep 30;echo world' > job.yaml
vi job.yaml
```

Add job.spec.completions=5

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  completions: 5 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

```bash
kubectl create -f job.yaml
```

Verify that it has been completed:

```bash
kubectl get job busybox -w # will take two and a half minutes
kubectl delete jobs busybox
```

</p>
</details>

### Create the same job, but make it run 5 parallel times

<details><summary>show</summary>
<p>

```bash
vi job.yaml
```

Add job.spec.parallelism=5

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  parallelism: 5 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

```bash
kubectl create -f job.yaml
kubectl get jobs
```

It will take some time for the parallel jobs to finish (>= 30 seconds)

```bash
kubectl delete job busybox
```

</p>
</details>

## Cron jobs

kubernetes.io > Documentation > Tasks > Run Jobs > [Running Automated Tasks with a CronJob](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/)

### Create a cron job with image busybox that runs on a schedule of "*/1 * * * *" and writes 'date; echo Hello from the Kubernetes cluster' to standard output

<details><summary>show</summary>
<p>

```bash
kubectl create cronjob busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
```

</p>
</details>

### See its logs and delete it

<details><summary>show</summary>
<p>

```bash
kubectl get cj
kubectl get jobs --watch
kubectl get po --show-labels # observe that the pods have a label that mentions their 'parent' job
kubectl logs busybox-1529745840-m867r
# Bear in mind that Kubernetes will run a new job/pod for each new cron job
kubectl delete cj busybox
```

</p>
</details>

### Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it takes more than 17 seconds to start execution after its schedule.

<details><summary>show</summary>
<p>

```bash
kubectl create cronjob time-limited-job --image=busybox --restart=Never --dry-run=client --schedule="* * * * *" -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > time-limited-job.yaml
vi time-limited-job.yaml
```
Add cronjob.spec.jobTemplate.spec.activeDeadlineSeconds=17

```bash
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  creationTimestamp: null
  name: time-limited-job
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: time-limited-job
    spec:
      activeDeadlineSeconds: 17 # add this line
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: time-limited-job
            resources: {}
          restartPolicy: Never
  schedule: '* * * * *'
status: {}
```

</p>
</details>

### Additonal Practice Questions

<details><summary>Get the pods with label information</summary>
<p>
   
```
kubectl get pods --show-labels
```
</p>
</details>


<details><summary>Create 5 nginx pods in which two of them is labeled env=prod and three of them is labeled env=dev</summary>
<p>
   
```
kubectl run nginx-dev1 --image=nginx --restart=Never --labels=env=dev
kubectl run nginx-dev2 --image=nginx --restart=Never --labels=env=dev
kubectl run nginx-dev3 --image=nginx --restart=Never --labels=env=dev
kubectl run nginx-prod1 --image=nginx --restart=Never --labels=env=prod
kubectl run nginx-prod2 --image=nginx --restart=Never --labels=env=prod
```
</p>
</details>


<details><summary>Verify all the pods are created with correct labels</summary>
<p>
   
```
kubeclt get pods --show-labels
```
</p>
</details>

<details><summary>Get the pods with label env=dev</summary>
<p>
   
```
kubectl get pods -l env=dev
```
</p>
</details>

<details><summary>Get the pods with label env=dev and also output the labels</summary>
<p>
   
```
kubectl get pods -l env=dev --show-labels
```
</p>
</details>

<details><summary>Get the pods with label env=prod</summary>
<p>
   
```
kubectl get pods -l env=prod
```
</p>
</details>

<details><summary>Get the pods with label env=prod and also output the labels</summary>
<p>
   
```
kubectl get pods -l env=prod --show-labels
```
</p>
</details>


<details><summary>Get the pods with label env</summary>
<p>
   
```
kubectl get pods -L env
```
</p>
</details>


<details><summary>Get the pods with labels env=dev and env=prod</summary>
<p>
   
```
kubectl get pods -l 'env in (dev,prod)'
```
</p>
</details>


<details><summary>Get the pods with labels env=dev and env=prod and output the labels as well</summary>
<p>
   
```
kubectl get pods -l 'env in (dev,prod)' --show-labels
```
</p>
</details>


<details><summary>Change the label for one of the pod to env=uat and list all the pods to verify</summary>
<p>
   
```
kubectl label pod/nginx-dev3 env=uat --overwrite

kubectl get pods --show-labels
```
</p>
</details>


<details><summary>Remove the labels for the pods that we created now and verify all the labels are removed</summary>
<p>
   
```
kubectl label pod nginx-dev{1..3} env-
kubectl label pod nginx-prod{1..2} env-

kubectl get po --show-labels
```
</p>
</details>


<details><summary>Let’s add the label app=nginx for all the pods and verify</summary>
<p>
   
```
kubectl label pod nginx-dev{1..3} app=nginx
kubectl label pod nginx-prod{1..2} app=nginx

kubectl get po --show-labels
```
</p>
</details>


<details><summary>Get all the nodes with labels (if using minikube you would get only master node)</summary>
<p>
   
```
kubectl get nodes --show-labels
```
</p>
</details>


<details><summary>Label the node (minikube if you are using) nodeName=nginxnode</summary>
<p>
   
```
kubectl label node minikube nodeName=nginxnode
```
</p>
</details>


<details><summary>Create a Pod that will be deployed on this node with the label nodeName=nginxnode</summary>
<p>
   
```
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > pod.yaml

// add the nodeSelector like below and create the pod

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  nodeSelector:
    nodeName: nginxnode
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f pod.yaml
```
</p>
</details>


<details><summary>Verify the pod that it is scheduled with the node selector</summary>
<p>
   
```
kubectl describe po nginx | grep Node-Selectors
```
</p>
</details>


<details><summary>Verify the pod nginx that we just created has this label</summary>
<p>
   
```
kubectl describe po nginx | grep Labels
```
</p>
</details>


<details><summary>Annotate the pods with name=webapp</summary>
<p>
   
```
kubectl annotate pod nginx-dev{1..3} name=webapp
kubectl annotate pod nginx-prod{1..2} name=webapp
```
</p>
</details>


<details><summary>Verify the pods that have been annotated correctly</summary>
<p>
   
```
kubectl describe po nginx-dev{1..3} | grep -i annotations
kubectl describe po nginx-prod{1..2} | grep -i annotations
```
</p>
</details>



<details><summary>Remove the annotations on the pods and verify</summary>
<p>
   
```
kubectl annotate pod nginx-dev{1..3} name-
kubectl annotate pod nginx-prod{1..2} name-

kubectl describe po nginx-dev{1..3} | grep -i annotations
kubectl describe po nginx-prod{1..2} | grep -i annotations
```
</p>
</details>


<details><summary>Remove all the pods that we created so far</summary>
<p>
   
```
kubectl delete po --all
```
</p>
</details>


<details><summary>Create a deployment called webapp with image nginx with 5 replicas</summary>
<p>
   
```
kubectl create deploy webapp --image=nginx --dry-run -o yaml > webapp.yaml

// change the replicas to 5 in the yaml and create it

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: webapp
  name: webapp
spec:
  replicas: 5
  selector:
    matchLabels:
      app: webapp
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: webapp
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}

kubectl create -f webapp.yaml
```
</p>
</details>


<details><summary>Get the deployment you just created with labels</summary>
<p>
   
```
kubectl get deploy webapp --show-labels
```
</p>
</details>


<details><summary>Output the yaml file of the deployment you just created</summary>
<p>
   
```
kubectl get deploy webapp -o yaml
```
</p>
</details>


<details><summary>Get the pods of this deployment</summary>
<p>
   
```
// get the label of the deployment
kubectl get deploy --show-labels

// get the pods with that label
kubectl get pods -l app=webapp
```
</p>
</details>


<details><summary>Scale the deployment from 5 replicas to 20 replicas and verify</summary>
<p>
   
```
kubectl scale deploy webapp --replicas=20

kubectl get po -l app=webapp
```
</p>
</details>


<details><summary>Get the deployment rollout status</summary>
<p>
   
```
kubectl rollout status deploy webapp
```
</p>
</details>


<details><summary>Get the replicaset that created with this deployment</summary>
<p>
   
```
kubectl get rs -l app=webapp
```
</p>
</details>


<details><summary>Get the yaml of the replicaset and pods of this deployment</summary>
<p>
   
```
kubectl get rs -l app=webapp -o yaml

kubectl get po -l app=webapp -o yaml
```
</p>
</details>


<details><summary>Delete the deployment you just created and watch all the pods are also being deleted</summary>
<p>
   
```
kubectl delete deploy webapp

kubectl get po -l app=webapp -w
```
</p>
</details>


<details><summary>Create a deployment of webapp with image nginx:1.17.1 with container port 80 and verify the image version</summary>
<p>
   
```
kubectl create deploy webapp --image=nginx:1.17.1 --dry-run -o yaml > webapp.yaml

// add the port section and create the deployment

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: webapp
  name: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: webapp
    spec:
      containers:
      - image: nginx:1.17.1
        name: nginx
        ports:
        - containerPort: 80
        resources: {}
status: {}

kubectl create -f webapp.yaml

// verify
kubectl describe deploy webapp | grep Image
```
</p>
</details>


<details><summary>Update the deployment with the image version 1.17.4 and verify</summary>
<p>
   
```
kubectl set image deploy/webapp nginx=nginx:1.17.4

kubectl describe deploy webapp | grep Image
```
</p>
</details>


<details><summary>Check the rollout history and make sure everything is ok after the update</summary>
<p>
   
```
kubectl rollout history deploy webapp

kubectl get deploy webapp --show-labels
kubectl get rs -l app=webapp
kubectl get po -l app=webapp
```
</p>
</details>


<details><summary>Undo the deployment to the previous version 1.17.1 and verify Image has the previous version</summary>
<p>
   
```
kubectl rollout undo deploy webapp

kubectl describe deploy webapp | grep Image
```
</p>
</details>



<details><summary>Update the deployment with the image version 1.16.1 and verify the image and also check the rollout history</summary>
<p>
   
```
kubectl set image deploy/webapp nginx=nginx:1.16.1

kubectl describe deploy webapp | grep Image

kubectl rollout history deploy webapp
```
</p>
</details>



<details><summary>Update the deployment to the Image 1.17.1 and verify everything is ok</summary>
<p>
   
```
kubectl rollout undo deploy webapp --to-revision=3

kubectl describe deploy webapp | grep Image

kubectl rollout status deploy webapp
```
</p>
</details>


<details><summary>Update the deployment with the wrong image version 1.100 and verify something is wrong with the deployment</summary>
<p>
   
```
kubectl set image deploy/webapp nginx=nginx:1.100

kubectl rollout status deploy webapp (still pending state)

kubectl get pods (ImagePullErr)
```
</p>
</details>


<details><summary>Undo the deployment with the previous version and verify everything is Ok</summary>
<p>
   
```
kubectl rollout undo deploy webapp
kubectl rollout status deploy webapp

kubectl get pods
```
</p>
</details>


<details><summary>Check the history of the specific revision of that deployment</summary>
<p>
   
```
kubectl rollout history deploy webapp --revision=7
```
</p>
</details>



<details><summary>Pause the rollout of the deployment</summary>
<p>
   
```
kubectl rollout pause deploy webapp
```
</p>
</details>


<details><summary>Update the deployment with the image version latest and check the history and verify nothing is going on</summary>
<p>
   
```
kubectl set image deploy/webapp nginx=nginx:latest

kubectl rollout history deploy webapp (No new revision)
```
</p>
</details>



<details><summary>Resume the rollout of the deployment</summary>
<p>
   
```
kubectl rollout resume deploy webapp
```
</p>
</details>


<details><summary>Check the rollout history and verify it has the new version
</summary>
<p>
   
```
kubectl rollout history deploy webapp

kubectl rollout history deploy webapp --revision=9
```
</p>
</details>


<details><summary>Apply the autoscaling to this deployment with minimum 10 and maximum 20 replicas and target CPU of 85% and verify hpa is created and replicas are increased to 10 from 1
</summary>
<p>
   
```
kubectl autoscale deploy webapp --min=10 --max=20 --cpu-percent=85

kubectl get hpa

kubectl get pod -l app=webapp
```
</p>
</details>


<details><summary>Clean the cluster by deleting deployment and hpa you just created</summary>
<p>
   
```
kubectl delete deploy webapp

kubectl delete hpa webapp
```
</p>
</details>


<details><summary>Create a Job with an image node which prints node version and also verifies there is a pod created for this job</summary>
<p>
   
```
kubectl create job nodeversion --image=node -- node -v

kubectl get job -w
kubectl get pod
```
</p>
</details>



<details><summary>Get the logs of the job just created</summary>
<p>
   
```
kubectl logs <pod name> // created from the job
```
</p>
</details>



<details><summary>Output the yaml file for the Job with the image busybox which echos “Hello I am from job”</summary>
<p>
   
```
kubectl create job hello-job --image=busybox --dry-run -o yaml -- echo "Hello I am from job"
```
</p>
</details>


<details><summary>Copy the above YAML file to hello-job.yaml file and create the job</summary>
<p>
   
```
kubectl create job hello-job --image=busybox --dry-run -o yaml -- echo "Hello I am from job" > hello-job.yaml

kubectl create -f hello-job.yaml
```
</p>
</details>


<details><summary>Verify the job and the associated pod is created and check the logs as well</summary>
<p>
   
```
kubectl get job
kubectl get po

kubectl logs hello-job-*
```
</p>
</details>



<details><summary>Delete the job we just created</summary>
<p>
   
```
kubectl delete job hello-job
```
</p>
</details>


<details><summary>Create the same job and make it run 10 times one after one</summary>
<p>
   
```
kubectl create job hello-job --image=busybox --dry-run -o yaml -- echo "Hello I am from job" > hello-job.yaml

// edit the yaml file to add completions: 10

apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: hello-job
spec:
  completions: 10
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - echo
        - Hello I am from job
        image: busybox
        name: hello-job
        resources: {}
      restartPolicy: Never
status: {}

kubectl create -f hello-job.yaml
```
</p>
</details>


<details><summary>Watch the job that runs 10 times one by one and verify 10 pods are created and delete those after it’s completed</summary>
<p>
   
```
kubectl get job -w
kubectl get po

kubectl delete job hello-job
```
</p>
</details>



<details><summary>Create the same job and make it run 10 times parallel</summary>
<p>
   
```
kubectl create job hello-job --image=busybox --dry-run -o yaml -- echo "Hello I am from job" > hello-job.yaml

// edit the yaml file to add parallelism: 10

apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: hello-job
spec:
  parallelism: 10
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - echo
        - Hello I am from job
        image: busybox
        name: hello-job
        resources: {}
      restartPolicy: Never
status: {}

kubectl create -f hello-job.yaml
```
</p>
</details>



<details><summary>Watch the job that runs 10 times parallelly and verify 10 pods are created and delete those after it’s completed</summary>
<p>
   
```
kubectl get job -w
kubectl get po

kubectl delete job hello-job
```
</p>
</details>



<details><summary>Create a Cronjob with busybox image that prints date and hello from kubernetes cluster message for every minute</summary>
<p>
   
```
kubectl create cronjob date-job --image=busybox --schedule="*/1 * * * *" -- bin/sh -c "date; echo Hello from kubernetes cluster"
```
</p>
</details>



<details><summary>Output the YAML file of the above cronjob</summary>
<p>
   
```
kubectl get cj date-job -o yaml
```
</p>
</details>



<details><summary>Verify that CronJob creating a separate job and pods for every minute to run and verify the logs of the pod</summary>
<p>
   
```
kubectl get job
kubectl get po

kubectl logs date-job-<jobid>-<pod>
```
</p>
</details>



<details><summary>Delete the CronJob and verify all the associated jobs and pods are also deleted</summary>
<p>
   
```
kubectl delete cj date-job

// verify pods and jobs
kubectl get po
kubectl get job
```
</p>
</details>



