# v7k8s 

### Login vsphere
```shell
ubuntu@cli-vm:~$ kubectl vsphere login --vsphere-username  Administrator@vsphere.local  --server=https://10.40.14.65   
Password:
Logged in successfully.

You have access to the following contexts:
   10.40.14.65
   k8s.corp.local
   ron-namespace
   svc
   svc-10.40.14.65
   tkg
   tkg-10.40.14.65
   tkg-cluster

If the context you wish to use is not in this list, you may need to try
logging in again later, or contact your cluster administrator.

To change context, use `kubectl config use-context <workload name>`
```


```shell
ubuntu@cli-vm:~$ k config use-context tkg
Switched to context "tkg".
ubuntu@cli-vm:~$ k get nodes
error: You must be logged in to the server (Unauthorized)
```
### Login TKG Cluster
```shell
ubuntu@cli-vm:~$ k config use-context tkg-cluster
Switched to context "tkg-cluster".
ubuntu@cli-vm:~$ k get nodes
NAME                                         STATUS   ROLES    AGE   VERSION
tkg-cluster-control-plane-6kf2n              Ready    master   93d   v1.16.8+vmware.1
tkg-cluster-workers-h77w2-5448b6977c-d29kk   Ready    <none>   93d   v1.16.8+vmware.1
tkg-cluster-workers-h77w2-5448b6977c-jvn4p   Ready    <none>   93d   v1.16.8+vmware.1
```


```shell

ubuntu@cli-vm:~$ k config use-context tkg-10.40.14.65
Switched to context "tkg-10.40.14.65".
ubuntu@cli-vm:~$ k get nodes
NAME                               STATUS     ROLES    AGE   VERSION
422d4255db46d9b3edf6160fe3091d61   Ready      master   93d   v1.16.7-2+bfe512e5ddaaaa
422d57850d90409f18bc206ae651d69f   Ready      master   93d   v1.16.7-2+bfe512e5ddaaaa
422d80b7bffaf609af79be00c5176206   NotReady   master   93d   v1.16.7-2+bfe512e5ddaaaa
esx-01a.corp.local                 Ready      agent    93d   v1.16.7-sph-4d52cd1
esx-02a.corp.local                 Ready      agent    93d   v1.16.7-sph-4d52cd1
esx-03a.corp.local                 Ready      agent    93d   v1.16.7-sph-4d52cd1
```

```shell
ubuntu@cli-vm:~$ k get nodes
NAME                               STATUS     ROLES    AGE   VERSION
422d4255db46d9b3edf6160fe3091d61   Ready      master   93d   v1.16.7-2+bfe512e5ddaaaa
422d57850d90409f18bc206ae651d69f   Ready      master   93d   v1.16.7-2+bfe512e5ddaaaa
422d80b7bffaf609af79be00c5176206   NotReady   master   93d   v1.16.7-2+bfe512e5ddaaaa
esx-01a.corp.local                 Ready      agent    93d   v1.16.7-sph-4d52cd1
esx-02a.corp.local                 Ready      agent    93d   v1.16.7-sph-4d52cd1
esx-03a.corp.local                 Ready      agent    93d   v1.16.7-sph-4d52cd1
```

```shell
ubuntu@cli-vm:~$ k get ns
NAME                               STATUS   AGE
default                            Active   93d
kube-node-lease                    Active   93d
kube-public                        Active   93d
kube-system                        Active   93d
ron-namespace                      Active   80m
svc                                Active   93d
tkg                                Active   93d
vmware-system-capw                 Active   93d
vmware-system-csi                  Active   93d
vmware-system-kubeimage            Active   93d
vmware-system-nsx                  Active   93d
vmware-system-registry             Active   93d
vmware-system-registry-143459887   Active   87m
vmware-system-tkg                  Active   93d
vmware-system-ucs                  Active   93d
vmware-system-vmop                 Active   93d
```

```shell
ubuntu@cli-vm:~$ k get all -n vmware-system-registry-143459887
NAME                                                      READY   STATUS         RESTARTS   AGE
pod/harbor-143459887-harbor-core-7d544c6c8-zwhsg          1/1     Running        1          86m
pod/harbor-143459887-harbor-database-0                    1/1     Running        0          86m
pod/harbor-143459887-harbor-jobservice-7c475b94f9-vq2f9   1/1     Running        0          86m
pod/harbor-143459887-harbor-nginx-5cf6c4d678-n4b5g        1/1     Running        0          86m
pod/harbor-143459887-harbor-portal-7b84c9dddf-xcdwc       1/1     Running        0          86m
pod/harbor-143459887-harbor-redis-0                       1/1     Running        0          86m
pod/harbor-143459887-harbor-registry-5875b67c79-mxrjl     0/2     ErrImagePull   0          86m
pod/harbor-143459887-harbor-registry-5875b67c79-t27ck     2/2     Running        0          79m

NAME                                         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)             AGE
service/harbor-143459887                     LoadBalancer   10.96.0.246   10.40.14.67   443:30988/TCP       87m
service/harbor-143459887-harbor-core         ClusterIP      10.96.0.249   <none>        80/TCP              86m
service/harbor-143459887-harbor-database     ClusterIP      10.96.0.180   <none>        5432/TCP            86m
service/harbor-143459887-harbor-jobservice   ClusterIP      10.96.0.116   <none>        80/TCP              86m
service/harbor-143459887-harbor-portal       ClusterIP      10.96.0.239   <none>        80/TCP              86m
service/harbor-143459887-harbor-redis        ClusterIP      10.96.0.94    <none>        6379/TCP            86m
service/harbor-143459887-harbor-registry     ClusterIP      10.96.0.188   <none>        5000/TCP,8080/TCP   86m

NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/harbor-143459887-harbor-core         1/1     1            1           86m
deployment.apps/harbor-143459887-harbor-jobservice   1/1     1            1           86m
deployment.apps/harbor-143459887-harbor-nginx        1/1     1            1           86m
deployment.apps/harbor-143459887-harbor-portal       1/1     1            1           86m
deployment.apps/harbor-143459887-harbor-registry     1/1     1            1           86m

NAME                                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/harbor-143459887-harbor-core-7d544c6c8          1         1         1       86m
replicaset.apps/harbor-143459887-harbor-jobservice-7c475b94f9   1         1         1       86m
replicaset.apps/harbor-143459887-harbor-nginx-5cf6c4d678        1         1         1       86m
replicaset.apps/harbor-143459887-harbor-portal-7b84c9dddf       1         1         1       86m
replicaset.apps/harbor-143459887-harbor-registry-5875b67c79     1         1         1       86m

NAME                                                READY   AGE
statefulset.apps/harbor-143459887-harbor-database   1/1     86m
statefulset.apps/harbor-143459887-harbor-redis      1/1     86m
```
