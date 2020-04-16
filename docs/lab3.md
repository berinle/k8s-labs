# Marketplace and Services 
No application lives in isolation. This works you through how to connect your application to a backing service.

## Additional pre-reqs to complete this lab:
* [Helm 3+](https://helm.sh/docs/intro/install/)

## What you will learn
* How to connect your apps to services

## Excercise
* Add a trusted container registry to helm

```
 helm repo add bitnami https://charts.bitnami.com/bitnami

 -- output --
"bitnami" has been added to your repositories
```

* Deploy a database using helm. The output provides a way to connect our database

```
helm install app-db --set root.password=secr3tpassword,user.database=app_database bitnami/mysql

-- out --
NAME: app-db
LAST DEPLOYED: Thu Apr 16 16:29:26 2020
NAMESPACE: docs
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Please be patient while the chart is being deployed

Tip:

  Watch the deployment status using the command: kubectl get pods -w --namespace docs

Services:

  echo Master: app-db-mysql.docs.svc.cluster.local:3306
  echo Slave:  app-db-mysql-slave.docs.svc.cluster.local:3306

Administrator credentials:

  echo Username: root
  echo Password : $(kubectl get secret --namespace docs app-db-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode)

To connect to your database:

  1. Run a pod that you can use as a client:

      kubectl run app-db-mysql-client --rm --tty -i --restart='Never' --image  docker.io/bitnami/mysql:8.0.19-debian-10-r87 --namespace docs --command -- bash

  2. To connect to master service (read/write):

      mysql -h app-db-mysql.docs.svc.cluster.local -uroot -p my_database

  3. To connect to slave service (read-only):

      mysql -h app-db-mysql-slave.docs.svc.cluster.local -uroot -p my_database

To upgrade this helm chart:

  1. Obtain the password as described on the 'Administrator credentials' section and set the 'root.password' parameter as shown below:

      ROOT_PASSWORD=$(kubectl get secret --namespace docs app-db-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode)
      helm upgrade app-db bitnami/mysql --set root.password=$ROOT_PASSWORD
```

* Deploy a sample application 

```
kubectl run awesome-app --image=busybox --restart=Never -l app=awesome-app -- sh -c "while true; do sleep 3; done"

-- out --
pod/awesome-app created
```

* Verify you can lookup the created database via DNS

```
kubectl exec -it awesome-app -- nslookup app-db-mysql

-- out --
Server:		10.96.0.10
Address:	10.96.0.10:53

Name:	app-db-mysql.docs.svc.cluster.local
Address: 10.101.110.1

*** Can't find app-db-mysql.svc.cluster.local: No answer
*** Can't find app-db-mysql.cluster.local: No answer
*** Can't find app-db-mysql.us-east-2.compute.internal: No answer
*** Can't find app-db-mysql.local: No answer
*** Can't find app-db-mysql.docs.svc.cluster.local: No answer
*** Can't find app-db-mysql.svc.cluster.local: No answer
*** Can't find app-db-mysql.cluster.local: No answer
*** Can't find app-db-mysql.us-east-2.compute.internal: No answer
*** Can't find app-db-mysql.local: No answer
```

* Verify you can connect to the database on the intended port

```
kubectl exec -it awesome-app -- nc -zv app-db-mysql 3306

-- out --
app-db-mysql (10.101.110.1:3306) open
```
Tcp connectivity worked fine.

* Inspect the environment variables that are now available to our application

```
kubectl exec -it awesome-app -- env | grep -i app_db

-- out --
APP_DB_MYSQL_SLAVE_SERVICE_HOST=10.105.10.3
APP_DB_MYSQL_SLAVE_PORT_3306_TCP=tcp://10.105.10.3:3306
APP_DB_MYSQL_PORT_3306_TCP_ADDR=10.101.110.1
APP_DB_MYSQL_SLAVE_SERVICE_PORT_MYSQL=3306
APP_DB_MYSQL_PORT_3306_TCP=tcp://10.101.110.1:3306
APP_DB_MYSQL_PORT_3306_TCP_PROTO=tcp
APP_DB_MYSQL_PORT=tcp://10.101.110.1:3306
APP_DB_MYSQL_PORT_3306_TCP_PORT=3306
APP_DB_MYSQL_SLAVE_PORT_3306_TCP_PROTO=tcp
APP_DB_MYSQL_SLAVE_PORT_3306_TCP_PORT=3306
APP_DB_MYSQL_SERVICE_PORT=3306
APP_DB_MYSQL_SERVICE_HOST=10.101.110.1
APP_DB_MYSQL_SERVICE_PORT_MYSQL=3306
APP_DB_MYSQL_SLAVE_SERVICE_PORT=3306
APP_DB_MYSQL_SLAVE_PORT=tcp://10.105.10.3:3306
APP_DB_MYSQL_SLAVE_PORT_3306_TCP_ADDR=10.105.10.3
```

* Finally, get the credentials to connect to the database with

```
kubectl get secrets/app-db-mysql -o jsonpath="{.data['mysql-root-password']}" | base64 --decode

-- out --
secr3tpassword
```

With DNS, environment variables and creds, you now have everything you need to connect securely to the database!

---

## Go a step further
Won't it be convenient to have a UI to do point and click deployments of COTS and other vendor packaged applications? 

Meet [kubeapps](https://kubeapps.com/)!

[Deploy](https://github.com/kubeapps/kubeapps/blob/master/docs/user/getting-started.md) kubeapps in your cluster and deploy some applications with it.