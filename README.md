# FTS3 on Kubernetes on Azure
## Deploying the MySQL database
Firstly create a storage class:
```
$ kubectl create -f mysql-sc.yaml
```
and check that it worked:
```
$ kubectl get storageclasses
NAME                TYPE
db                  kubernetes.io/azure-disk
default (default)   kubernetes.io/azure-disk
```
Note that a `storageAccount` isn't specified in `mysql-sc.yaml` as this doesn't seem to work (the persistent volume claim will get stuck in the pending state). Now create a persistent volume claim:
```
$ kubectl create -f mysql-pvc.yaml
```
After a few moments you should see something like this:
```
$ kubectl get pvc
NAME                                       CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS    CLAIM                    REASON    AGE
pvc-fc20620e-2529-11e7-854c-000d3a2947a2   100Gi      RWO           Delete          Bound     default/mysql-pv-claim             8m
```
Before creating the MySQL database we need to create secrets containing the root password as well as the password to be used by FTS3 services:
```
$ kubectl create secret generic mysql-pass --from-file=password.txt
$ kubectl create secret generic mysql-pass-fts3 --from-file=password-fts3.txt
```
Finally we can now create a deployment for the MySQL database in addition to the service:
```
$ kubectl create -f mysql-deployment.yaml
```
Eventually there should be a running MySQL pod:
```
$ kubectl get svc,pods,deployments
NAME                           CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
svc/kubernetes                 10.0.0.1       <none>        443/TCP    16d
svc/mysql-fts3                 10.0.92.171    <none>        3306/TCP   30m

NAME                            READY     STATUS    RESTARTS   AGE
po/mysql-fts3-414169266-xdfnq   1/1       Running   0          30m

NAME                DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/mysql-fts3   1         1         1            1           30m
```

## Deploying FTS3
Create a secret containing a valid host certificate:
```
$ kubectl create secret generic fts3-cert --from-file=hostcert=hostcert.pem --from-file=hostkey=hostkey.pem
secret "fts3-cert" created
```

## References
The following were helpful in preparing this:
* https://github.com/bigchaindb/bigchaindb/tree/master/k8s/mongodb
* https://github.com/kubernetes/kubernetes/tree/master/examples/mysql-wordpress-pd
