# CKS Notes

## ServiceAccounts

- You can disable automounting of a `ServiceAccount` on a `ServiceAccount` or `Pod` resource: `automountServiceAccountToken: false`
- `mount | grep sec` to show the mount inside the `Pod`.
- Mount for token also shows as mounted volume for the `Pod`. Path can be seen there.

## Kubernetes API

Three components:

- Authentication - Who are you?
- Authorization - What are you allowed to do?
- Admission - admission controllers.

Each request to the API is filtered through these.

Requests are treated as:

- A normal user
- A `ServiceAccount`
- Anonymous access

Every request must authenticate unless it's anonymous.

- `--anonymous-auth` kubelet flag must be set to `false` to disable anonymous access.

- `/etc/kubernetes/manifests/kube-apiserver.yaml`

- `--insecure-port` is set to `0` by default, which disables the insecure port. Setting this to anything else will also disable Authentication and Authorization.

## Config

- `kubectl config view --raw`
- `k config set-context jane --user=jane --cluster=kubernetes`
- `k config set-credentials jane --client-key=jane.key --client-certificate=jane.crt --embed-certs`
- `k config use-context jane`

You can use a different config file:

- `k --kubeconfig` or environment variable `KUBECONFIG`

For example, on a worker `Node`, there is no `kubeconfig` available to use with `kubectl` - but `kubelet` has its own config file that it uses to communicate with the cluster master:

- `cat /etc/kubernetes/kubelet.conf`
- `k --kubeconfig /etc/kubernetes/kubelet.conf get node`

## Auth

- `k auth can-i delete deploment -A`

By extracting the Kubernetes server `ca` and the user `cert` and `key` from `kubectl config view --raw`, you can actually makes manual API calls against the Kubernetes API:

- `curl https://10.142.0.2:6443 --cacert ca --cert cert --key key`

## Certificates

Certificates live at `/etc/kubernetes/pki` on the server. You can also find similar information on clients.

- `/etc/kubernetes/pki`
- `openssl x509 -in apiserver.crt -text`

## Node Restriction Admission Controller

In `/etc/kubernetes/manifests/kube-apiserver.yaml`:

```yaml
- --enable-admission-plugins=NodeRestriction
```

This sets the `NodeRestriction` admission controller to enabled. This means that requests are subject to it. For example, this prevents us from labeling the master `Node` from the worker:

- `k label node cks-master cks/test=yes`

This will fail - however, we can label our own `Node`:

- `k label node cks-worker cks/test=yes`

There are restricted labels that we cannot set ourselves:

- `node-restriction.kubernetes.io/test=yes` - you couldn't set this.

This would prevent a malicious user from changing a label like this to allow `Pods` that look for that label to instead run on a compromised `Node`.

## Updates

It's good to update for support, security fixes, bug fixes, and dependencies.

`1.19.2` - major/minor/patch

Minor version every 3 months. No Long Term Support.

Maintenance release branches for the most recent three minor releases - for now, that's `1.19`, `1.18`, and `1.17`.

#### Update Process

First, master components are upgraded - `apiserver`, `controller-manager`, `scheduler`.

Then, worker components are upgraded - `kubelet`, `kube-proxy`.

Components should always have the same minor version as the `apiserver`.

`kubelet` can be two minor versions below `apiserver`, but in general don't do this.

Stick with same version as `apiserver` or one below for safety.

##### The Process

- `kubectl cordon` then `kubectl drain`
- Upgrade
- `kubectl uncordon`

###### Master

```bash
$ k drain cks-master --ignore-daemonsets
$ apt-cache show kubeadm | grep 1.19
$ apt-get install kubeadm=1.19.3-00 kubelet=1.19.3-00 kubectl=1.19.3-00
$ kubeadm upgrade plan
$ kubeadm upgrade apply v1.19.3
$ k uncordon cks-master
```

###### Worker

```bash
$ k drain cks-worker --ignore-daemonsets (from master)
$ apt-cache show kubeadm | grep 1.19
$ apt-get install kubeadm=1.19.3-00 kubelet=1.19.3-00 kubectl=1.19.3-00
$ systemctl restart kubelet
$ k uncordon cks-worker (from master)
```

###### Application Resiliency

As always, applications should be able to survive an upgrade:

- `Pod` termination grace periods.
- `PodDisruptionBudgets`
- Pod Lifecycle Events

## Secrets

Usually passwords, API keys, information needed by an application.

## Container Runtime

`kubelet` args:

- `--container-runtime`
- `--container-runtime-endpoint`

`crictl` is an open source adaption that is container and Kubernetes native.

Kata containers adds an additional virtualization layer - a bit more like traditional VMs.

gVisor (Google) (`runsc`) implements a limited-use kernel that adds further fine grained separation. Runs in user space so it's separated from the Linux kernel.

`RuntimeClasses` allow you to specify further runtime environments for objects.

You use `spec.runtimeClassName` to associate a `Pod` with a given `RuntimeClass`.

## Security Contexts

```yaml
spec:
  volumes:
    - name: vol
      emptyDir: {}
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
    - name: foo
      command:
        - sh
        - c
        - sleep 1d
      image: busybox
      resources: {}
      securityContext:
        runAsUser: 0
```

Notice how here, `securityContext` is set top level for all `Pods`, but you can override for a specific container.

Check out API reference from docs if you want to know more about what a flag does.

Forcing running as non-root:

```yaml
spec:
  volumes:
    - name: vol
      emptyDir: {}
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
    - name: foo
      command:
        - sh
        - c
        - sleep 1d
      image: busybox
      resources: {}
      securityContext:
        runAsNonRoot: true
```

This may error if the image runs as root and there is no top-level `securityContext`: `Error: container has runAsNonRoot and image will run as root`

## Privileged Containers

```yaml
spec:
  volumes:
    - name: vol
      emptyDir: {}
  containers:
    - name: foo
      command:
        - sh
        - c
        - sleep 1d
      image: busybox
      resources: {}
      securityContext:
        privileged: true
```

Setting `privileged: true` will allow the given `Pod` to make OS-level changes, `sysctl` etc., and interact with the kernel. This is bad practice.

Privileged means the container user `root (0)` is directly mapped to host `root (0)`.

## Privilege Escalation

Privilege Escalation controls whether a process can gain more privileges than its parent process.

```yaml
spec:
  volumes:
    - name: vol
      emptyDir: {}
  containers:
    - name: foo
      command:
        - sh
        - c
        - sleep 1d
      image: busybox
      resources: {}
      securityContext:
        allowPrivilegeEscalation: true
```

`allowPrivilegeEscalation: true` is the default. You can set to `false` to disable this behavior.

## Pod Security Policies

Cluster-level resources. Controls under which security conditions a `Pod` has to run.

Pod Security Policy runs as an admission controller. A `Pod` will only be created if it adheres to these rules. It inspects the security contexts.

Enabling this will deny all `Pods` from being created in the cluster since out of the box none of the `ServiceAccounts` have the necessary permissions to look at `PodSecurityPolicies`.

As an admin user, we can create `Pods`, but they wouldn't create as a result of a `Deployment`, as an example, since that creates using the `ServiceAccount` we talked about before.

You'd have to give the `ServiceAccount`, in this case the `default`, the ability to evaluate `PodSecurityPolicies`:

- `k create role psp-access --verb=use --resource=podsecuritypolicies`
- `k create rolebinding psp-access --role=psp-access --serviceaccount=default:default`

You'd want to create the proper RBAC and `PodSecurityPolicy` resources before enabling this.

## Mutual TLS (mTLS)

Two-way bilateral authentication. Two parties authenticate to each other to create a secure communication channel.

By default, all `Pods` can communicate with each other as an implicit function of the chosen CNI.

Typically, TLS is terminated at an `Ingress` and is unencrypted on the backend in the cluster, between `Pods`, etc.

You could use a sidcar proxy container to handle this encryption overhead.

Something like Istio works with this model - a managed proxy that takes care of certificates.

You could use an `initContainer` that creates `iptables` rules that would force all traffic from your application's `Pods` through the proxy container.

All containers in a `Pod` have access to the same network namespace provided you add the capability as given in the example.

## Open Policy Agent

Open source, general purpose policy engine. You've used this before. It works in Kubernetes too.

Uses rego in Kubernetes the same way. Works with JSON/YAML. It is not natively Kubernetes aware with its resources, for example.

OPA Gatekeeper creates CRDs to allow Kubernetes to work with OPA.

```yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
```

```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: pod-must-have-foolabel
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: pod-must-have-barlabel
```

It creates admission webhooks - there are two kinds - `validating` and `mutating`. Mutating webhooks are invoked first and can modify objects. After this is done, Validating webhooks are invoked and can reject requests to enforce custom policies.

Use Rego playground to test OPA policies.

## Image Footprints

Remember that containers use cgroups, which means they have acces sto the host OS kernel. It is possible to run containers using the same PID so they share processes.

Docker images are built using layers. Only the instructions `RUN`, `COPY`, and `ADD` create layers. Other instructions create temporary intermediate images, and do not increase the size of the build.

An image size factors in the base image size plus additional layers, etc.

```Dockerfile
FROM ubuntu
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go
COPY app.go .
RUN CGO_ENABLED=0 go build app.go

FROM alpine
RUN chmod a-w /etc
RUN addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser
RUN rm -rf /bin/*
COPY --from=0 /app /home/appuser
USER appuser
CMD ["./app"]
```

Using this logic, the `--from=0` here step copies from the first stage, stage 0, and stage 1 at the bottom copies the file into the local directory in the resultant final image.

This essentially gives you a resulting image that only encompasses the final stage, reducing its size.

### Hardening An Image

- Using specific versions in a Dockerfile is more secure instead of using something like `latest`.
- Don't run as root in a container. In the example above, we establish a dedicated user and then run as that user by calling `USER`.
- Making the filesystem read only is also more secure. This avoids allowing write access to a given filesystem running as part of a container. Using the line `RUN chmod a-w /etc`, we remove write permissions for the `/etc` directory for all users.
- Removing shell access is also an optimization. We do this in the Dockerfile by running `RUN rm -rf /bin/*` - this essentially removes the ability to run anything located in that directory. This explains why sometimes you cannot exec into a container (think top level Kubernetes `Pods`) because that is missing.
- In general, running commands together in a layer, like `apt-get update && apt-get install` all in the same operation is better and decreases size. Also cleanup install packages in the build.

## Static Analysis

Looks at the source code and text files and parses them to check against rules. Those rules can then be enforced.

Examples:
- Always define resource requests and limits.
- `Pods` should never use the default `ServiceAccount`.
- Don't store sensitive data in plain text in Dockerfiles or Kubernetes resources.

Twistlock or Sysdig is an example of this.

You could do this in an image build phase or after the build phase in a test phase. We do this by leveraging the Sysdig inline scanner.

`PodSecurityPolicies` and OPA can be used within the cluster for static analysis.

For example, pulling info from a `Secret` as an environment variable is more secure than hardcoding things. Look out for obvious stuff.

Kubesec is risk analysis for Kubernetes resources. It's open source and opinionated. It runs using a fixed set of rules based on security best practices. It can run as binary, a Docker container, a `kubectl` plugin, or admission controller. Remember that tools like Sysdig also offer an admission controller. You can also past a manifest into Kubesec to evaluate it on demand.

Remember these important features for `securityContext`:
- `readOnlyRootfilesystem = true`
- `runAsNonRoot = true`
- `runAsUser -gt 10000`
- `capabilities .drop`

OPA offers a tool called `conftest` that uses the same OPA rego language. You can run this in Docker as well.

You can also use `confest` against Dockerfiles.

## Image Vulnerability Scanning

Containers that contain exploitable packages are a problem. This could result in privilege escalation, data leaks, DDoS, etc.

https://cve.mitre.org
https://nvd.nist.gov/

Tools use these databases to scan images for vulnerabilities. We use the Sysdig inline scanner for this. You could stop a build, or use an Admission Controller to not allow a compromised image version to run in a cluster.

You could also restrict based on registry hostnames within the cluster using something like OPA or `PodSecurityPolicies`. This would happen either in `MutatingAdmission` or `Validating` webhook stages.

This type of scanning could also take place in a container registry like GCR or ECR.

### Clair

Open source vulnerability assessment tool. CNCF supported. Uses vulnerability databases.

### Trivy

Also open source. One command to run it.

`$ docker run ghcr.io/aquasecurity/trivy:latest image nginx`

This will cross reference failures with CVE numbers.

## Supply Chain Security

Using a private registry is an example of a secure supply chain component.

You can create a `docker-registry` `Secret` in Kubernetes and then associate the `imagePullSecrets` to the `ServiceAccounts`, for example.

You can run an image using an image digest instead of a tag since tags can theoretically change and point to different digests.

You could use OPA via the Admission Controller to limit images to specific repositories.

Remember that you create a kind of `ConstraintTemplate` that uses `spec.crd.spec.names[0].kind=K8sTrustedImages` then create a `K8sTrustedImages` object. The admission webhook would then deny creation of a `Pod` if its spec fails the checks specified by these templates.

`ImagePolicyWebhook` creates a kind of `ImageReview` which can be assessed by an external tool as part of an admission workflow.

You would add this to `--enable-admission-plugins` as `ImagePolicyWebhook` to enable the Admission Controller.

- `--admission-control-config-file=path-to-admission-config.yaml`

You have to adjust Volumes to mount this data in the `kube-apiserver` `Pod`. You must have certificates, a `kubeconfig`, etc.

`AdmissionConfiguration` kind is where this logic is configured. Remember that `defaultAllow` is set to `false` by default which means no `Pods` will create out of the box if this configuration is incomplete.

## Behavioral Analytics

Syscall interface is provided by the kernel, for example `getpid()` or `reboot()`.

Applications run in the user space. Applications can communicate with the syscall interface or they can use libraries. The request is then passed to the kernel and the hardware.

`seccomp` and AppArmor lie between the user space and syscall interface for added protection.

Processes in a container are able to communicate with the kernel given how they run in shared spaces. Remember the concept of `cgroups`.

`strace` intercepts and logs system calls made by a process. It can also log and display signals received by a process so it's good for debugging etc.

- `$ strace ls /`

This would provide a list of syscalls made to the kernel.

In the end, all commands ran on the command line result in syscalls for how they operate.

- `/proc` directory contains information and connections to processes and kernel.
- Study it to learn how processes work.
- Configuration and administrative tasks.
- Contains files that don't exist, yet you can access these.

You can do `docker ps | grep etcd` and then `ps aux | grep etcd` to find the running `etcd` process.

- `$ strace -p 3502 -f` - this would show you syscalls made by a process, in this case `etcd`. `-f` follows it.

You can go to `cd /proc/3502` and `ls` and see open files related to a process - `etcd` in this case.

Writes to `etcd` for things like `Secrets` will show up in `/proc/3502/fd/7 (symlink)` as an example.

- `$ pstree -p` shows process tree of running processes. You could find `containerd` in here and see a list of running containers.

You could use this to find the `pid` of a running container, navigate to its `/proc/pid` folder and `cat environ` and see environment variable data.

### Falco

CNCF native runtime security. Deep kernel tracing build on the Linux kernel.

Describe security rules against a system, detect unwanted behavior.

Automated response to a security violations.

Kubernetes docs has a page specific to Falco with instructions for installation.

`/etc/falco` where configuration files are stored.

- `$ tail -f /var/log/syslog | grep falco`

You would see log output from `falco` activity that shows running processes, package management launched, shell executions, etc.

Know how to find those rules and review them.

## Immutability of Containers at Runtime

Immutability simply means the container won't be modified during its lifetime.

### Mutable

- `ssh` to a container, stop application, update application, start application

### Immutable

- Create new container image, delete container instance, create new container instance.

With immutable containers, we always know the state. With mutable instances, we know less.

Immutability allows us to use advanced deployment methods native to Kubernetes, easy rollbacks, more reliability, and better security.

### Making Containers Immutable

- Remove bash/shell, make filesystem read only, run as user and non-root
- You could remove write privileges to all non-essential directories in line using `command`.
- `startupProbe` - runs prior to liveness/readiness checks. You could use this to enforce changes too.
- Use `SecurityContexts` and `PodSecurityPolicies` to enforce read only fs, etc.
- You could use an init container to handle read/write permissions and then harden the app container.

### StartupProbe

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: immutable
  name: immutable
spec:
  containers:
  - image: httpd
    name: immutable
    resources: {}
    startupProbe:
      exec:
        command:
          - rm
          - /bin/touch
      initialDelaySeconds: 1
      periodSeconds: 5
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### SecurityContext

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: immutable
  name: immutable
spec:
  containers:
  - image: httpd
    name: immutable
    resources: {}
    securityContext:
      readOnlyRootFilesystem: true
    volumeMounts:
      - mountPath: /usr/local/apache2/logs
        name: cache-volume
  volumes:
    - name: cache-volume
      emptyDir: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

In this example, we use `readOnlyRootFilesystem` but `apache` will complain if it can't write to the directory it needs for logs. We get around this by using `emptyDir` to allow it to write to that temporary directory.

## Kubernetes Auditing

Kubernetes is all based on API requests. Auditing allows us to hold those requests so that we can review them for security purposes.

- Did someone access an important `Secret` while it was not protected? We can check who accessed it.
- When was the last time that user X did access cluster Y?
- Does my CRD work properly?

Requests to the Kubernetes API run through stages:

- `RequestReceived` - The stage for events generated as soon as the audit handler receives the request, and before it is delegated down the handler chain.
- `ResponseStarted` - Once the response headers are sent, but before the response body is sent. This stage is only generated for long-running requests (e.g. `watch`).
- `ResponseComplete` - The response body has been completed and no more bytes will be sent.
- `Panic` - Events generated when a panic occurred.

Using these stages, we can customize exactly what we want to log.

Audit policy rule levels:

- `None` - don't log events that match this rule.
- `Metadata` - log request metadata (requesting user, timestamp, resource, verb, etc.) not not request or response body.
- `Request` - log event metadata and request body but not response body. This does not apply for non-resource requests.
- `RequestResponse` - log event metadata, request, and response bodies. This does not apply for non-resource requests.

```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - "RequestReceived"
rules:
# log no "read" actions
  - level: None
    verbs: ["get", "watch", "list"]

# log nothing regarding events
  - level: None
    resources:
      - group: "" # core
        resources: ["events"]

# log nothing coming from some groups
  - level: None
    userGroups: ["system:nodes"]
  - level: RequestResponse
    resources:
      - group: ""
        resources: ["secrets"]

# for everything else log
  - level: Metadata
```

### Configuring

- `$ mkdir /etc/kubernetes/audit`
- Create `policy.yaml` in the `audit` folder.
- Edit `kube-apiserver.yaml` with the following flags:
  - `--audit-policy-file=/etc/kubernetes/audit/policy.yaml`
  - `--audit-log-path=/etc/kubernetes/audit/logs/audit.log`
  - `--audit-log-maxsize=500`
  - `--audit-log-maxbackup=5`
- Edit `kube-apiserver.yaml` to mount the location in the `apiserver` `Pods` to store the logs:

```yaml
    volumeMounts:
    - mountPath: /etc/kubernetes/audit
      name: audit
  volumes:
    - hostPath:
        path: /etc/kubernetes/audit
        type: DirectoryOrCreate
      name: audit
```

Use the docs for auditing to help with this part.

Backends for storing audit data can be JSON logs, external APIs (webhooks), dynamic backend (ElasticSearch, FileBeat, Fluentd).

This file is largely use for parsing on your own. It should really be exported somewhere useful and readable. But you can do something like create a `Secret` and `grep` for it in the log file.

### Advanced Audit Policy

```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - 'RequestReceived'
rules:
  # log no "read" actions
  - level: None
    verbs: ['get', 'watch', 'list']

  # log only metadata from Secrets
  - level: Metadata
    resources:
      - group: ''
        resources: ['secrets']

  # for everything else log
  - level: RequestResponse
```

If you create a policy and the `kube-apiserver` `Pod` doesn't come back up, remember the trick using `/var/log/pods` to see what's going on. You'll likely see more than one since the new one didn't start, so use the latest and read its logs.

### Looking at API Access History for Secrets

```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - 'RequestReceived'
rules:
  # log no "read" actions
  - level: None
    verbs: ['get', 'watch', 'list']

  # log only metadata from Secrets
  - level: RequestResponse
    resources:
      - group: ''
        resources: ['secrets']

  # for everything else
  - level: RequestResponse
```

Commenting out the line for the file path for the policy is enough to cause the `apiserver` to restart. You'll have to uncomment it to get it to restart again and use your new policy.

You could use this to see API calls for things - including patching objects, etc.

## Kernel Hardening Tools

Processes are restricted in namespaces which restrict what they can see - users, filesystems, other processes. `cgroups` restrict the resource usage of processes.

Between the syscall interface and top level spaces like user and app space, we can use AppArmor and `seccomp` to harden the kernel.

### AppArmor

You create profiles in AppArmor to define what a process can and cannot do. You could do this for something like `kubelet` or any other app.

Profiles have the concept of modes:

- `Unconfined` - process can escape, nothing is enforced.
- `Complain` - processes can escape but will be logged.
- `Enforce` - processes cannot escape, cannot do more than we allow them to do in their profile.

Commands:

- `aa-status` - show profiles
- `aa-genprof` - create new profile (smart wrapper around `aa-logprof`)
- `aa-complain` - put profile in complain mode
- `aa-enforce` - put profile in enforce mode
- `aa-logprof` - update the profile if app produced some more usage logs (syslog)

You could do this for `curl`: `$ aa-genprof curl`

Profiles are located in `/etc/apparmor.d`.

You could then run `aa-logprof` to review recommended changes and then save them so your processes are more secure.

#### With Docker

`/etc/apparmor.d` is where policies go.

For example `/etc/apparmor.dr/docker-nginx`

`$ apparmor_parser path-to-file` adds it.

`$ docker run --security-opt apparmor=docker-nginx`

#### With Kubernetes

Container runtime need sto support AppArmor. Docker does by default.

AppArmor must be installed on every node, and the profiles need to be available on every node.

AppArmor profiles are specified **per container**, not per `Pod`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    container.apparmor.security.beta.kubernetes.io/secure: localhost/docker-nginx # you reference the name of the pod after / and the profile as the value
  labels:
    run: secure
  name: secure
spec:
  containers:
  - image: nginx
    name: secure
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

The profile must exist in order for the `Pod` to create, otherwise its creation will be `Blocked`.

Annotations for AppArmor can be found in Kubernetes docs.

### seccomp

"Security computing mode."

Security facility in the Linux kernel.

Restricts execution of syscalls made by processes.

By default, applications can make syscalls as needed. Using `seccomp`, we can restrict syscalls.

It onlyallows `exit()`, `sigreturn()`, `read()`, and `write()` by default.

Nowadays `seccomp` is combined with BPF to make `seccomp-bpf`.

#### With Docker

`$ docker run --security-opt seccomp=default.json nginx`

#### With Kubernetes

In order for kubelet to use `seccomp` profiles, you can consult the Kubernetes docs and find the argument for specifying the directory, which is `/var/lib/kubelet/seccomp` by default.

You can create the directory and then add your profile to it.

```yaml
apiVersion: v1
kind: Pod
metadata:
  #annotations:
  #  seccomp.security.alpha.kubernetes.io/secure: localhost/profiles/default.json # pre 1.19 you could use this annotation
  labels:
    run: secure
  name: secure
spec:
  # in 1.19+, you enable seccomp using securityContext
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: default.json
  containers:
  - image: nginx
    name: secure
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Like AppArmor, if the profile doesn't exist, `Pod` will not be created with status `CreateContainerError`. Running `describe` will show this error.

## Reduce Attack Surface of Host

Attack surface consists of anything exposed that can be sourced by malicious code. Networking, applications, IAM, etc.

Applications and kernel should always be up to date. Additional packages should not be present if not needed.

Use a firewall to eliminate access to a port.

Run applications as a specific user instead of root to avoid privilege escalation.

`Nodes` should be ephemeral. They should be created from images and immutable. They should be recycled efficiently and without downtime.

A lot of included services with base level OS images, like `ubuntu`, are unnecessary and increase attack surface.

- `$ netstat -plnt`
- `$ lsof -i :22`
- `$ systemctl status kubelet`
- `$ ps aux`

For example, to disable a service:

- `$ systemctl list-units --type=service --state=running | grep snapd`
- `$ systemctl stop snapd`
- `$ systemctl disable snapd`

Show users:

- `$ cat /etc/passwd`

Add users:

- `$ adduser foo`
  
Delete users:

- `$ deluser foo`

Show bash processes:

- `$ ps aux | grep bash`

## Handy Stuff

- `/var/log/pods` - You can see `Pod` logs here if you're unable to use the API for some reason.
