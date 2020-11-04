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
-rw-rw-r-- 1 willian willian 104 out 29 20:45 Namespace.yaml
-rw-rw-r-- 1 willian willian 232 out 29 20:45 Service.yaml
```

Inciando com o banco, criamos um Namespace dedicado:

```
~/projeto-esig/rocketchat$ kubectl create -f mongodb/Namespace.yaml 
namespace/rocketchat-mongodb created
```

E em seguida os Objetos restantes:

```
~/projeto-esig/rocketchat$ kubectl create -f mongodb/
namespace/rocketchat-mongodb unchanged
persistentvolume/mongo-persistent-storage-persistentvolume created
persistentvolumeclaim/mongo-persistent-storage-claim created
job.batch/mongo-replica created
service/rocketchat-mongo-service created
statefulset.apps/rocketchat-mongo created
```

Verificamos que os pods foram criados/executados com sucesso:

```
~/projeto-esig/rocketchat/mongodb$ kubectl get pods -n rocketchat-mongodb 
NAME                  READY   STATUS      RESTARTS   AGE
rocketchat-mongo-0    1/1     Running     0          64m
rocketchat-mongo-1    1/1     Running     0          63m
mongo-replica-4vr8j   0/1     Completed   0          62m
```


Assim como feito com o banco, também criamos um Namespace dedicado para a aplicação:

```
~/projeto-esig/rocketchat$ kubectl create -f rocket/Namespace.yaml
namespace/rocketchat-server created
```

E os Objetos restantes:

```
~/projeto-esig/rocketchat$ kubectl apply -f rocket/
deployment.apps/rocketchat-server-deploy created
namespace/rocketchat-server unchanged
service/rocketchat-server-service created
```

Por fim, verificamos os recursos da aplicação criados e já podemos acessá-la:

```
$ kubectl -n rocketchat-server get all
NAME                                            READY   STATUS    RESTARTS   AGE
pod/svclb-rocketchat-server-service-76q96       1/1     Running   0          69m
pod/svclb-rocketchat-server-service-jml7v       1/1     Running   0          69m
pod/svclb-rocketchat-server-service-zrjzq       1/1     Running   0          69m
pod/rocketchat-server-deploy-5858b86dcb-ks769   1/1     Running   4          69m

NAME                                TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/rocketchat-server-service   LoadBalancer   10.43.221.84   172.17.0.2    3000:32457/TCP   69m

NAME                                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/svclb-rocketchat-server-service   3         3         3       3            3           <none>          69m

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rocketchat-server-deploy   1/1     1            1           69m

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/rocketchat-server-deploy-5858b86dcb   1         1         1       69m
```

![Captura](https://github.com/willian-as/rocketchat/blob/main/images/Captura%20de%20tela%20de%202020-10-30%2021-41-35.png)

