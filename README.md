# Rodando o [Rocket.Chat](https://github.com/RocketChat/Rocket.Chat) em um cluster local de Kubernetes

## Criação do cluster

Utilizando o [k3d](https://github.com/rancher/k3d), criamos o cluster com 3 nodes (1 server e 2 workers):

```
$ k3d cluster create esig --agents 2
```
## Subindo o projeto

### Namespace da aplicação
```
$ kubectl create -f mongodb/Namespace.yaml 
namespace/rocketchat created
```
### Criação dos objetos
```
$ kubectl create -f mongodb/Service.yaml
$ kubectl create -f mongodb/PersistentVolume.yaml
$ kubectl create -f mongodb/StatefulSet.yaml
$ kubectl create -f mongodb/ReplicaSetInit.yaml
$ kubectl create -f rocket/Service.yaml 
$ kubectl create -f rocket/Deployment.yaml 
```

### Overview
```
$ kubectl get all -n rocketchat 
NAME                                            READY   STATUS      RESTARTS   AGE
pod/rocketchat-mongo-0                          1/1     Running     0          15m
pod/rocketchat-mongo-1                          1/1     Running     0          15m
pod/rocketchat-mongo-2                          1/1     Running     0          15m
pod/mongo-replica-qn5pb                         0/1     Completed   0          6m38s
pod/svclb-rocketchat-server-service-5vhvm       1/1     Running     0          100s
pod/svclb-rocketchat-server-service-ptjv6       1/1     Running     0          100s
pod/svclb-rocketchat-server-service-tgbtl       1/1     Running     0          100s
pod/rocketchat-server-deploy-64bff4954f-j9cwf   1/1     Running     0          14s

NAME                                TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/rocketchat-mongo-service    ClusterIP      None            <none>        27017/TCP        17m
service/rocketchat-server-service   LoadBalancer   10.43.220.212   172.17.0.2    3000:32614/TCP   100s

NAME                                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/svclb-rocketchat-server-service   3         3         3       3            3           <none>          100s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rocketchat-server-deploy   1/1     1            1           14s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/rocketchat-server-deploy-64bff4954f   1         1         1       14s

NAME                                READY   AGE
statefulset.apps/rocketchat-mongo   3/3     15m

NAME                      COMPLETIONS   DURATION   AGE
job.batch/mongo-replica   1/1           3s         6m38s
```

### Acesso à aplicação
![Captura](https://github.com/willian-as/rocketchat/blob/main/images/Captura%20de%20tela%20de%202020-10-30%2021-41-35.png)
