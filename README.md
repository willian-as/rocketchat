# Rodando o [Rocket.Chat](https://github.com/RocketChat/Rocket.Chat) em um cluster Kubernetes

## Criação do cluster

Utilizando o [k3d](https://github.com/rancher/k3d), criamos o cluster com 3 nodes (1 master e 2 workers):

```
$ k3d cluster create esig --agents 2
INFO[0000] Created network 'k3d-esig'                   
INFO[0000] Created volume 'k3d-esig-images'             
INFO[0001] Creating node 'k3d-esig-server-0'            
INFO[0013] Creating node 'k3d-esig-agent-0'             
INFO[0017] Creating node 'k3d-esig-agent-1'             
INFO[0020] Creating LoadBalancer 'k3d-esig-serverlb'    
INFO[0024] (Optional) Trying to get IP of the docker host and inject it into the cluster as 'host.k3d.internal' for easy access 
INFO[0040] Successfully added host record to /etc/hosts in 4/4 nodes and to the CoreDNS ConfigMap 
INFO[0040] Cluster 'esig' created successfully!         
INFO[0040] You can now use it like this:                
kubectl cluster-info
```

## Manifestos e criação dos Objetos

O Rocket.Chat se divide entre o banco (MongoDB) e a aplicação em si.
Com isso, foram criados os seguintes Objetos:

```
~/projeto-esig/rocketchat/mongodb$ ls -l
total 20
-rw-rw-r-- 1 willian willian 106 out 29 20:45 Namespace.yaml
-rw-rw-r-- 1 willian willian 477 out 29 21:25 PersistentVolume.yaml
-rw-rw-r-- 1 willian willian 407 out 29 21:37 ReplicaSetInit.yaml
-rw-rw-r-- 1 willian willian 215 out 29 20:42 Service.yaml
-rw-rw-r-- 1 willian willian 825 out 29 21:37 StatefulSet.yaml

~/projeto-esig/rocketchat/rocket$ ls -l
total 12
-rw-rw-r-- 1 willian willian 844 out 29 21:36 Deployment.yaml
-rw-rw-r-- 1 willian willian 232 out 29 20:45 Service.yaml
```

Inciando com o banco, criamos um Namespace dedicado para a aplicação:

```
~/projeto-esig/rocketchat$ kubectl create -f mongodb/Namespace.yaml 
namespace/rocketchat created
```

E em seguida os Objetos restantes na seguinte ordem:

```
~/projeto-esig/rocketchat$ kubectl create -f mongodb/Service.yaml 
service/rocketchat-mongo-service created

~/projeto-esig/rocketchat$ kubectl create -f mongodb/PersistentVolume.yaml 
persistentvolume/mongo-pv created
persistentvolumeclaim/mongo-pv-claim created

~/projeto-esig/rocketchat$ kubectl create -f mongodb/StatefulSet.yaml 
statefulset.apps/rocketchat-mongo created

~/projeto-esig/rocketchat$ kubectl create -f mongodb/ReplicaSetInit.yaml 
job.batch/mongo-replica created
```

Verificamos que todos os Objetos do banco foram criados/executados com sucesso:

```
~/projeto-esig/rocketchat$ kubectl get all -n rocketchat 
NAME                      READY   STATUS      RESTARTS   AGE
pod/rocketchat-mongo-0    1/1     Running     0          13m
pod/rocketchat-mongo-1    1/1     Running     0          13m
pod/rocketchat-mongo-2    1/1     Running     0          13m
pod/mongo-replica-qn5pb   0/1     Completed   0          4m23s

NAME                               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)     AGE
service/rocketchat-mongo-service   ClusterIP   None         <none>        27017/TCP   15m

NAME                                READY   AGE
statefulset.apps/rocketchat-mongo   3/3     13m

NAME                      COMPLETIONS   DURATION   AGE
job.batch/mongo-replica   1/1           3s         4m23s
```


Agora criamos os Objetos da aplicação:

```
~/projeto-esig/rocketchat$ kubectl create -f rocket/Service.yaml 
service/rocketchat-server-service created

~/projeto-esig/rocketchat$ kubectl create -f rocket/Deployment.yaml 
deployment.apps/rocketchat-server-deploy created
```

Por fim, verificamos que todos os Objetos foram criados e já podemos acessar a aplicação:

```
~/projeto-esig/rocketchat$ kubectl get all -n rocketchat 
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

![Captura](https://github.com/willian-as/rocketchat/blob/main/images/Captura%20de%20tela%20de%202020-10-30%2021-41-35.png)
