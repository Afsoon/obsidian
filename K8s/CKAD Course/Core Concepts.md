## Recap Concepts
### Node
Es una instancia, aka VM, donde puede contender varios PODs.

### Cluster
Es un conjuntos de **Nodes**.

### Master
Es un nodo dentro del **Cluster** que se encarga de gestionar los **Nodos** dentro.

### Components
- API Server: La API que usa los usuarios para interactuar con el **cluster** de K8s.
- etcd: Es un **key-value store distributed** que contiene toda la información de los **clusters** y **masters**.
- Scheduler: Se encarga de distribuir el carga de trabajo entre diferentes **nodes**.
- Controller: Se encarga de gestionar cuando los nodos se caen o están en estado de error.
- Contanier Runtime: Puede ser Docker, RKT, CRI-O.
- Kubelet: Es el agente que se encarga de que los containers funcioner adecuadamente.

Así es como se organiza un Worker Node y Master Node, de forma simplificada:
![[Pasted image 20240211073221.png]]

### Kubectl

Algunos comandos básico son:
```
kubectl run minikube -> Ejecuta un pod

kubectl cluster-info -> obtener la información del cluster

kubectl get nodes -> Los nodos de un cluster
```

### Docker VS ContainerD

Nerdclt es una cli con una api similar a Docker pero para ContainerD.

Crictl es una herramienta que se usa en K8s para hacer debugging en K8s.

> [!NOTE] Discusiones de Github a sobre el uso de Crictl y expandir esta sección
> https://github.com/kubernetes-sigs/cri-tools/issues/868
> https://github.com/kubernetes-sigs/cri-tools/pull/869

### Pods

