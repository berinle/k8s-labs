# Blue Green deployment

## What you will learn
* Conduct a blue/green deployment

## Excercise
* Deploy v1 of your application

```
kubectl apply -f xxx

-- out --
deployment.apps/articulate created
```

* Verify the deployment is up and running

```
kubectl get deployments

-- out --
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
articulate   1/1     1            1           2m46s
```

* Create a service object to expose the deployment

```
kubectl apply -f bg/svc.yaml

-- out --
service/articulate created
```

* Ensure the service is available

```
kubectl get svc/articulate

-- out --
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
articulate           ClusterIP   10.96.212.251   <none>        80/TCP     5m15s
```

* Use port forwarding to reach the application locally

```
kubectl port-forward svc/articulate 6060:80

-- out --
Forwarding from 127.0.0.1:6060 -> 8080
Forwarding from [::1]:6060 -> 8080
```

* Point your browser to http://localhost:6060 (or the port you are forwarding to)

* Deploy v2 of your application

```
kubectl apply -f bg/green-deploy.yaml

-- out --
deployment.apps/articulate-g created
```

* Verify you able to see both versions of your app being served