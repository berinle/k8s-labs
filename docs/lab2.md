# Scaling, Logs and HA

## What you will learn
* How to scale your apps
* View application logs
* High availability and Self Healing

## Excercise

## Scale your application
* Create the deployment

```
kubectl create deployment articulate --image=berinle/articulate:02a18f91

-- output --
deployment.apps/articulate created
```

* Get the objects created as part of the deployment

```
kubectl get all

-- output --
NAME                              READY   STATUS    RESTARTS   AGE
pod/articulate-5466968447-dhn5v   1/1     Running   0          19s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/articulate   1/1     1            1           19s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/articulate-5466968447   1         1         1       19s
```

Notice a pod was automatically created as part of the deployment. Also notice there is a `ReplicaSet` associated with the deployment. This will be used to scale up/down the deployment as instructed.

* Scale to 3 copies of your application

```
kubectl scale deployments/articulate --replicas=3

-- output --
deployment.apps/articulate scaled
```

* Check the deployment rollout of the scale out

```
kubectl rollout status deployments/articulate

-- output --
Waiting for deployment "articulate" rollout to finish: 1 of 3 updated replicas are available...
Waiting for deployment "articulate" rollout to finish: 2 of 3 updated replicas are available...
deployment "articulate" successfully rolled out
​
```

* Get the deployment

```
kubectl get deployments/articulate

-- output --
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
articulate   3/3     3            3           5m17s
```

* Verify there are 3 pods now up and running

```
kubectl get pods -l app=articulate

-- output --
NAME                          READY   STATUS    RESTARTS   AGE
articulate-5466968447-5hjbp   1/1     Running   0          3m49s
articulate-5466968447-dhn5v   1/1     Running   0          6m39s
articulate-5466968447-lq9th   1/1     Running   0          3m49sm
```


---

## View application logs

* Check the logs of one of the pods from the previous steps

```
kubectl logs pods/articulate-5466968447-5hjbp

-- output --
Container memory limit unset. Configuring JVM for 1G container.
Calculated JVM Memory Configuration: -XX:MaxDirectMemorySize=10M -XX:MaxMetaspaceSize=112169K -XX:ReservedCodeCacheSize=240M -Xss1M -Xmx424406K (Head Room: 0%, Loaded Class Count: 17390, Thread Count: 250, Total Memory: 1073741824)

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.3.RELEASE)

2020-04-16 01:35:28.595  WARN 1 --- [           main] pertySourceApplicationContextInitializer : Skipping 'cloud' property source addition because not in a cloud
2020-04-16 01:35:28.627  WARN 1 --- [           main] nfigurationApplicationContextInitializer : Skipping reconfiguration because not in a cloud
2020-04-16 01:35:28.702  INFO 1 --- [           main] i.p.pcf.sme.ers.PcfErsDemo1Application   : Starting PcfErsDemo1Application on articulate-5466968447-5hjbp with PID 1 (/workspace/BOOT-INF/classes started by cnb in /workspace)
2020-04-16 01:35:28.706  INFO 1 --- [           main] i.p.pcf.sme.ers.PcfErsDemo1Application   : The following profiles are active: dev
2020-04-16 01:35:36.120  INFO 1 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data repositories in DEFAULT mode.
2020-04-16 01:35:36.364  INFO 1 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 219ms. Found 1 repository interfaces.
2020-04-16 01:35:39.199  INFO 1 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration' of type [org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration$$EnhancerBySpringCGLIB$$5590d5f4] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2020-04-16 01:35:39.328  INFO 1 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.hateoas.config.HateoasConfiguration' of type [org.springframework.hateoas.config.HateoasConfiguration$$EnhancerBySpringCGLIB$$d5112326] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2020-04-16 01:35:41.448  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-04-16 01:35:41.567  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
...
```

---

## High Availablity

With 3 copies of the app running, lets trigger a failure on one of them and watch it recovery

* First expose the service so we can 

```
kubectl expose deployment/articulate --type=NodePort --name=articulate --port=8080

-- output --
service/articulate exposed
```

* Get the dynamically assigned port by listing the service

```
kubectl get service/articulate

-- output --
NAME         TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
articulate   NodePort   10.100.200.110   <none>        8080:32068/TCP   4s
```

* In a terminal window, watch the pods

```
kubectl get po -w -l app=articulate

-- output --
NAME                          READY   STATUS    RESTARTS   AGE
articulate-5466968447-5hjbp   1/1     Running   0          70m
articulate-5466968447-dhn5v   1/1     Running   0          73m
articulate-5466968447-lq9th   1/1     Running   0          70m
```

* Open your browser and point to the application at `http://EXTERNAL-IP:PORT`
* Navigate to `Service & HA` tab and click the `Kill` button
* Notice one of the pods exits and is immediately brought back online

```
NAME                          READY   STATUS    RESTARTS   AGE
articulate-5466968447-5hjbp   1/1     Running   0          70m
articulate-5466968447-dhn5v   1/1     Running   0          73m
articulate-5466968447-lq9th   1/1     Running   0          70m
articulate-5466968447-5hjbp   0/1     Completed   0          74m
articulate-5466968447-5hjbp   1/1     Running     1          74m
```

---

## Clean up the deployment

```
kubectl delete deployment,service -l app=articulate

-- output --
deployment.apps "articulate" deleted
service "articulate" deleted
```