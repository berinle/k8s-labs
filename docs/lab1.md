# Deploying your first application

[Deploying your first app]
## Requirements
* A running kubernetes cluster. You could use [minikube](https://minikube.sigs.k8s.io/docs/start/)
* [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl)

## What you will learn
* How to deploy an application to kubernetes
* How to access the deployed application
* Concepts of a Pod
* Concepts of a Deployment
* How to scale your apps

## Excercise

In order to deploy our first container we must use a Pod. A Pod is a Kubernetes abstraction that represents a group of one or more application containers (such as Docker or rkt), and some shared resources for those containers. It is the smallest unit in kubernetes

### Deploy your first pod

Lets create our very first container application in the cluster. To do this we’ll use the kubectl run command to create a single pod:
1. Lets create our very first container application in the cluster. To do this we’ll use the kubectl run command to create a single pod:
```
kubectl run nginx --image=nginx:latest --restart=Never -l app=nginx

--- output ---
pod/nginx created
```
2. You can use `kubectl` to retrieve details about the running pod. 
- First let's check the status of the newly created pod
```
kubectl get pods/nginx

--- output ---
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          25s
```
- Next let's check what node the pod was scheduled to run on. We can do this with the `-o wide` optional flags
```
kubectl get pods/nginx -o wide

--- output ---
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE                                   NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          61s   10.200.72.5   88377460-9e77-4461-ba61-e0da9c27bd8e   <none>           <none>
```
- Finally, let's describe the pod to see additional details
```
kubectl describe pods/nginx

--- output ---
Name:         nginx
Namespace:    docs
Priority:     0
Node:         88377460-9e77-4461-ba61-e0da9c27bd8e/192.168.4.143
Start Time:   Wed, 15 Apr 2020 15:01:58 -0400
Labels:       app=nginx
Annotations:  <none>
Status:       Running
IP:           10.200.72.5
IPs:
  IP:  10.200.72.5
Containers:
  nginx:
    Container ID:   docker://b7f7feef3bdd34523615621c556975585834eb3030989e68a251808e767cb051
    Image:          nginx:latest
    Image ID:       docker-pullable://nginx@sha256:282530fcb7cd19f3848c7b611043f82ae4be3781cb00105a1d593d7e6286b596
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 15 Apr 2020 15:02:00 -0400
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-f645l (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-f645l:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-f645l
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From                                           Message
  ----    ------     ----       ----                                           -------
  Normal  Scheduled  <unknown>  default-scheduler                              Successfully assigned docs/nginx to 88377460-9e77-4461-ba61-e0da9c27bd8e
  Normal  Pulling    108s       kubelet, 88377460-9e77-4461-ba61-e0da9c27bd8e  Pulling image "nginx:latest"
  Normal  Pulled     108s       kubelet, 88377460-9e77-4461-ba61-e0da9c27bd8e  Successfully pulled image "nginx:latest"
  Normal  Created    108s       kubelet, 88377460-9e77-4461-ba61-e0da9c27bd8e  Created container nginx
  Normal  Started    107s       kubelet, 88377460-9e77-4461-ba61-e0da9c27bd8e  Started container nginx
```
3. Access your pod. By default, workloads within kubernetes are not exposed outside the cluster. To reach our nginx instance, we would have to expose it outside the cluster somehow. There are several ways to do that.

- Port Forward
- NodePort
- LoadBalancer

#### Port Forward
1. First let's expose our pod via port-forward
```
## this maps port 8080 on our host to 80 on the container (which is the port exposed by the nginx application)
## be sure to use a port that is NOT already taken on your host. If 8080 is taken, use a different port
kubectl port-forward pods/nginx 8080:80

--- output ---
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80 
```
2. Point your browser to http://localhost:8080
3. Control-C out of the terminal running `port-forward` to stop it

#### Port Forward
1. First let's expose our pod via port-forward
```
## this maps port 8080 on our host to 80 on the container (which is the port exposed by the nginx application)
## be sure to use a port that is NOT already taken on your host. If 8080 is taken, use a different port
kubectl port-forward pods/nginx 8080:80

--- output ---
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80 
```
2. Point your browser to http://localhost:8080
3. Control-C out of the terminal running `port-forward` to stop it


#### NodePort
1. Let's expose our pod via `NodePort`. This allows you to reach the application from `NODE_IP:PORT`
```
## this maps a random port on the service object to your app 
kubectl expose pods/nginx --port=80 --type=NodePort

-- output --
service/nginx exposed
```
2. Get the service object to know which got assigned to your app. In this particular case, we got assigned `30491`. Yours will mostly be different
```
kubectl get service/nginx

-- output --
NAME    TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nginx   NodePort   10.100.200.213   <none>        80:30491/TCP   4s
```
3. Get the node port IP address(es)
```
kubectl get nodes -o wide

-- output --
NAME                                   STATUS   ROLES    AGE    VERSION            INTERNAL-IP     EXTERNAL-IP     OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
4efdc9df-6adc-4ad6-9f1e-60f880724fd3   Ready    <none>   5d1h   v1.16.7+vmware.1   192.168.x.x     192.168.x.x     Ubuntu 16.04.6 LTS   4.15.0-88-generic   docker://18.9.9
88377460-9e77-4461-ba61-e0da9c27bd8e   Ready    <none>   5d1h   v1.16.7+vmware.1   192.168.x.x     192.168.x.x     Ubuntu 16.04.6 LTS   4.15.0-88-generic   docker://18.9.9
```
4. Point your browser to `http://EXTERNAL-IP:NODE-PORT`


#### LoadBalancer (TODO)
This option can ONLY be performed if the infrastructure your cluster is hosted on has the ability to create load balancers. 

Congrats! You have deployed your first pod.

Delete the created service and pod

```
kubectl delete service,pod nginx

-- output --
service "nginx" deleted
pod "nginx" deleted
```

Pods by themselves are quite limited. For example, they are not resilient to failures, don't self heal, nor can they be scaled. This is why `Deployment`s comes in. A deployment a controller for a group of pods. It brings scalability, resilence, and high availability to pods.

Let's re-create our nginx pod, but this time as a deployment.

1. Create the deployment

```
kubectl create deployment nginx --image=nginx:latest

-- output --
deployment.apps/nginx created
```

2. Get the objects created as part of the deployment
```
kubectl get all

-- output --
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-75b7bfdb6b-tqpch   1/1     Running   0          3m51s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           3m51s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-75b7bfdb6b   1         1         1       3m51s
```

Notice a pod was automatically created as part of the deployment. Also notice there is a `ReplicaSet` associated with the deployment. This will be used to scale up/down the deployment as instructed.

3. Expose the deployment on a `NodePort`

```
kubectl create service nodeport nginx --tcp=8080:80

-- output --
service/nginx created
```

4. Get the nginx service and see what port was assigned to your app
```
kubectl get service/nginx

-- output --
NAME    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
nginx   NodePort   10.100.200.25   <none>        8080:32154/TCP   2m16s
```
5. Point your browser to `http://EXTERNAL-IP:NODE-PORT`

---

## Scale your application
1. Scale to 3 copies of your application
```
kubectl scale deployment/nginx --replicas=3

-- output --
deployment.apps/nginx scaled
```

2. Check the deployment rollout of the scale out
```
kubectl rollout status deployment/nginx

-- output --
deployment "nginx" successfully rolled out
```

3. Get the deployment
```
kubectl get deployments/nginx

-- output --
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           30m
```

4. Verify there are 3 pods now up and running
```
kubectl get pods -l app=nginx

-- output --
NAME                     READY   STATUS    RESTARTS   AGE
nginx-75b7bfdb6b-6kcwf   1/1     Running   0          55s
nginx-75b7bfdb6b-jbrwh   1/1     Running   0          55s
nginx-75b7bfdb6b-tqpch   1/1     Running   0          27m
```

5. Clean up the deployment

```
kubectl delete deployment,service -l app=nginx

-- output --
deployment.apps "nginx" deleted
service "nginx" deleted
```