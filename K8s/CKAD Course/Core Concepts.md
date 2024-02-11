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

Es la unidad minima que K8s puede crear. Un **Pod** es una instancia de tu aplicación. Primer se escalan los **PODs** en un aumento de usuarios, pero si el **Node** no puede con el trabajo, se crea un nuevo **Node**.

Las características que tiene un **POD** son:
- Puede contener más de un **container**. Esto significa que si un POD se elimina, se elimina **todas los containers**.
- Comparten el mismo namespace de networking y espacio.

Ver la sección de PODS para ver como crear un YAML para PODS [[Kubernetes Concepts]]
### Notas del labs

```
kubectl get pods -o wide -> Muestra mas información de los pods del namespace

kubectl run <name> --image=<image> --dry-run=client -o yaml > filename.yml -> Vale esto nos genera un yml con toda la información base para crear un POD.

kubectl get pod <pod-name> -o yaml > pod-definition.yaml -> Si el pod ya existe, genera un yaml con la misma configuración.

kubectl edit pod <pod-name> -> solo permite modificar las propiedades:

- spec.containers[*].image
- spec.initContainers[*].image
- spec.activeDeadlineSeconds
- spec.tolerations
- spec.terminationGracePeriodSeconds
```

> [!NOTE] Declerative vs Imperative
> El comando que se ha usado en las soluciones muestra un warning cuando se hace el edit, mas adelante se hablará de las dos formas de hacer YAML y como solventar ese error.

### ReplicaSet

> [!NOTE] Del curso de Absolute Beginners K8s
> Ver la sección de ReplicaSets para ver como crear un YAML para PODS [[Kubernetes Concepts]]
