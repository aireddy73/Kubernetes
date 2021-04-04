# Kubernetes


<!DOCTYPE NETSCAPE-Bookmark-file-1>
<!-- This is an automatically generated file.
     It will be read and overwritten.
     DO NOT EDIT! -->
<META HTTP-EQUIV="Content-Type" CONTENT="text/html; charset=UTF-8">
<TITLE>Kubernetes CKA/ CKAD Bookmarks</TITLE>
<H1>Bookmarks</H1>

<DL><p> 
<DT><A HREF=" " 

            <DL><p>
                <DT><A HREF="https://github.com/mrbobbytables/k8s-intro-tutorials"> K8S Tutorials</A>
				<DT><A HREF="https://github.com/aireddy73/Kubernetes-Certified-Administrator"> K8S Walid Shaari </A>
				<DT><A HREF="https://github.com/aireddy73/CKAD-Practice-Questions"> Bhargav Bachina </A>
				<DT><A HREF="https://github.com/aireddy73/CKAD-exercises "> CKA Zeal Vora</A>
				<DT><A HREF="https://github.com/aireddy73/CKAD-exercises"> CKAD - Dimitris-Ilias Gkanatsios </A> 
				<DT> <A HREF="https://github.com/aireddy73/ckad" > CKAD - Subodh Dharmadhikari</A>
				<DT> <A HREF= "https://github.com/aireddy73/k8s-guide "> Eric Shanks </A>
				<DT> <A HREF= "https://github.com/aireddy73/ckad-prep"> Benjamin Muschko </A>
				<DT> <A HREF= "https://github.com/aireddy73/kubernetes-network-policy-recipes " > Network Policies </A>
				<DT> <A HREF= "https://github.com/aireddy73/K8s-training-official" > K8S Tutorials </A>
				<DT> <A HREF= ""> </A>
				<DT> <A HREF= ""> </A>
				
				<DL><p>




</DL> </P>

<DL><p>
        <DT><H3 ADD_DATE="1596599838" LAST_MODIFIED="1597181536">CKAD</H3>
        <DL><p>
            <DT><H3 ADD_DATE="1596599860" LAST_MODIFIED="1596650209">Core Concepts</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/concepts/overview/components/">Kubernetes Components | Kubernetes</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/overview/working-with-objects/">Working with Kubernetes Objects | Kubernetes</A>
            </DL><p>
            <DT><H3 ADD_DATE="1596931542" LAST_MODIFIED="1597107879">Cluster Architecture</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/concepts/architecture/">Cluster Architecture | Kubernetes</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/architecture/nodes/" >Nodes | Kubernetes</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/architecture/control-plane-node-communication/" >CP-Node Communication</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/containers/images/#using-a-private-registry" >Pod Image - Image Secret</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/">Container Lifecycle Hooks</A>
            </DL><p>
            <DT><H3 ADD_DATE="1597107891" LAST_MODIFIED="1598398915">Pods</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/pods/">Pods | Kubernetes</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/" >Pod Lifecycle | Kubernetes</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/pods/init-containers/" >Init Containers | Kubernetes</A>
                <DT><H3 ADD_DATE="1597534609" LAST_MODIFIED="1597620293">Taints and Tolerations</H3>
                <DL><p>
                    <DT><A HREF="https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/#concepts" >Pod with tolerations</A>
                    <DT><A HREF="https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#node-affinity" >Node affinity example</A>
                    <DT><A HREF="https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity" >Antiaffinity Example</A>
                    <DT><A HREF="https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#always-co-located-in-the-same-node" >Pod antifaffinity real example</A>
                </DL><p>
                <DT><A HREF="https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-a-container-environment-variable-with-data-from-a-single-configmap">Configmap mounting in Pods</A>
                <DT><A HREF="https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/" ">Configure Liveness, Readiness and Startup Probes | Kubernetes</A>
            </DL><p>
            <DT><H3 ADD_DATE="1597182272" LAST_MODIFIED="1597192613">ReplicaSets</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/#example" >ReplicaSet Example</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/" >ReplicaSet | Kubernetes</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/#replicaset-as-a-horizontal-pod-autoscaler-target">ReplicaSet as HP Autoscaler</A>
            </DL><p>
            <DT><H3 ADD_DATE="1597192634" LAST_MODIFIED="1597193243">ReplicationControllers</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/#responsibilities-of-the-replicationcontroller" >ReplicationController Capabilities</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/#running-an-example-replicationcontroller">ReplicationController Example</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/" >ReplicationController | Kubernetes</A>
            </DL><p>
            <DT><H3 ADD_DATE="1597193254" LAST_MODIFIED="1597252439">Deployments</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/deployment/" >Deployments | Kubernetes</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment">Deployments Example</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#checking-rollout-history-of-a-deployment" >Deployment Rollout kubectl</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment" >Autoscaling Deployments kubectl</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#proportional-scaling" >Deployment - Proportional Scaling</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#canary-deployment" >Canary Release Deployments</A>
            </DL><p>
            <DT><H3 ADD_DATE="1597258153" LAST_MODIFIED="1597788129">StatefulStates</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/" >StatefulSets | Kubernetes</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#components" >StatefulSets Example</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#pod-identity" >Pod Identity</A>
                <DT><A HREF="https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/#statefulset" >StatefulSet MySQL example</A>
            </DL><p>
            <DT><H3 ADD_DATE="1597270310" LAST_MODIFIED="1597272090">DaemonSet</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/" >DaemonSet | Kubernetes</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#create-a-daemonset" >DaemonSet Example</A>
            </DL><p>
            <DT><H3 ADD_DATE="1597272100" LAST_MODIFIED="1604690709">Jobs</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/job/">Jobs | Kubernetes</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/job/#running-an-example-job" >Jobs Example</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/job/#parallel-jobs" >Types of Parallel Jobs</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/job/#job-patterns" >Jobs Patterns</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/pods/#pod-templates" >Jobs Example</A>
            </DL><p>
            <DT><H3 ADD_DATE="1597275373" LAST_MODIFIED="1598308150">CronJobs</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/" >CronJob | Kubernetes</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/#example" >CronJob Example</A>
                <DT><A HREF="https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/#creating-a-cron-job" >CronJob Example</A>
            </DL><p>
            <DT><H3 ADD_DATE="1597289232" LAST_MODIFIED="1597359424">Services</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/concepts/services-networking/service/#defining-a-service" >Service Example 1</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/services-networking/service/#headless-services" >Headless Service</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/services-networking/service/#nodeport" >NodePort Example</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer" >LoadBalancer Example</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/services-networking/service/#external-ips" >ExternalIP example</A>
            </DL><p>
            <DT><H3 ADD_DATE="1597359438" LAST_MODIFIED="1609726683">Ingress</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource"> Ingress Example</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/services-networking/ingress/#types-of-ingress" >Types of Ingress</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/services-networking/ingress/#hostname-wildcards" >Ingress With Hosts</A>
            </DL><p>
            <DT><H3 ADD_DATE="1597360730" LAST_MODIFIED="1597450355">Network Policy</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/concepts/services-networking/network-policies/#networkpolicy-resource" >Network Policies Example</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/services-networking/network-policies/#default-deny-all-ingress-traffic" >Deny All Ingress</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/services-networking/network-policies/#default-allow-all-ingress-traffic" >Allow All Ingress</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/services-networking/network-policies/#default-deny-all-egress-traffic" >Deny All Egress</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/services-networking/network-policies/#default-allow-all-egress-traffic" >Allow All Egress</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/services-networking/network-policies/#default-deny-all-ingress-and-all-egress-traffic" >Deny all ingress and egress</A>
            </DL><p>
            <DT><H3 ADD_DATE="1597450428" LAST_MODIFIED="1609780585">PersistentVolumes</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims" >Persistent Volumes Claim Example</A>
                <DT><A HREF="https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume" >PV with hostPath as mount point</A>
                <DT><A HREF="https://kubernetes.io/docs/tasks/configure-pod-container/configure-projected-volume-storage/#configure-a-projected-volume-for-a-pod" >Projected Volume PV Example</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/storage/volumes/#projected" >Projected Volumes</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/storage/volumes/#example-pod-with-a-secret-a-downward-api-and-a-configmap">Projected Volume Example 1 (All types mounted)</A>
                <DT><A HREF="https://kubernetes.io/docs/tasks/configure-pod-container/configure-projected-volume-storage/" >Prjected Volume Example</A>
            </DL><p>
            <DT><H3 ADD_DATE="1597453196" LAST_MODIFIED="1598381597">Configuration</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/concepts/configuration/configmap/" >ConfigMaps</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/configuration/configmap/#using-configmaps-as-files-from-a-pod">ConfigMaps as File Mounts</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/configuration/secret/#creating-a-secret-using-kubectl" >Secrets kubectl</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/configuration/secret/#creating-a-secret-manually" A>Secrets file</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod" >Secrets as File in Pod</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/configuration/secret/#projection-of-secret-keys-to-specific-paths">Secrets Keys as volume mounts</A>
                <DT><A HREF="https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables">Secrets as ENV</A>
                <DT><A HREF="https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables" >All Key Value Pairs as ENV for configmap</A>
            </DL><p>
            <DT><H3 ADD_DATE="1597620327" LAST_MODIFIED="1597688731">Tasks</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/" >Assign Memory to Containers &amp; Pods</A>
                <DT><A HREF="https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/" >Quality of Service for Pods</A>
                <DT><A HREF="https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/#configure-a-volume-for-a-pod" >Use EmptyDir for Pod Volume Storage</A>
            </DL><p>
            <DT><H3 ADD_DATE="1597695184" LAST_MODIFIED="1597698085">Security Context</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-the-security-context-for-a-pod" >Security Context Pod Example 1</A>
                >Add Capabilities </A>
            </DL><p>
            <DT><H3 ADD_DATE="1603063758" LAST_MODIFIED="1603063758">Observations</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/tasks/debug-application-cluster/events-stackdriver/" >events filtering</A>
            </DL><p>
            <DT><H3 ADD_DATE="1609444476" LAST_MODIFIED="1609709726">Examples</H3>
            <DL><p>
                <DT><A HREF="https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#step-two-add-a-nodeselector-field-to-your-pod-configuration" >Assigning Pods to Nodes | Kubernetes</A>
            </DL><p>
        </DL><p>
</DL><p>
