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

Nos ayuda mantener una alta disponibilidad, balancear la carga o escalar el numero de **Pods** y **Node**.

Una gran diferencia entre **ReplicaController** y **ReplicaSet**, el ultimo permite manejar **PODs** que son creados fuera de la definición del fichero.

Otra diferencia es que puedes crear el **ReplicaSet** después de crear los **PODs**, si no existe la misma cantidad, añadirá los **PODs** que quedan. Aunque los **PODs** ya existan **SIEMPRE** tienes que escribir la propiedad **template** porque los **PODs** pueden fallar y tener que recrearlo desde nuevo.

> [!WARNING] Borrado de ReplicaSets
> Si se borra un **ReplicaSets**, también se borra los **PODs** que esta monitorizando
#### Notas del Lab
```
k scale --replicas=<amount> -f <replica filename>.yml -> Modificar en runtime el numero de replicas de un ReplicaSet

kubectl describe replicaset <replica-set-name>

kubectl explain replicaset

kubect delete rs <replica-set-name>
```

### Deployments

> [!NOTE] Del curso de Absolute Beginners K8s
> 
> Ver la sección de Deployments para ver como crear un YAML para PODS [Kubernetes Concepts](app://obsidian.md/Kubernetes%20Concepts)


### Namespaces
Por defecto hay 3 namespaces creados por k8s:

- kube-system: Todos los servicios necesarios para hacer funcionar k8s, se hace para evitar que los modifiques por accedente.
- kube-public: Todos los servicios que son de cara al público.
- default: Es el entorno en el que trabajas por defecto.

Cada namespace puede tener sus propias **policies** y **quota storage and cpus**

K8s tiene su propio DNS interno donde lleva el registro de todos los servicios con su namespace como parte de la URL. Si los servicios están dentro del mismo **namespace** la url es solo su nombre del servicio, pero si esta en otro servicio entonces la URL es:

`db-service.dev.svc.cluster.local` esta URL esta formado por:

- `db-service`: el nombre del servicio
- `dev`: namespace name
- `svc`: subdomain de servicio
- `cluster.local`: La url del cluster por defecto.


```
k get pods --namespace/n=<namespace> -> ver los pods en un namespace

k create -f file --namespace/n=<namespace> -> crea el pod en otro namespace.

k config set-context $(kubectl config current-context) --namespace/n=<namespace> -> Para cambiar de namespace y no tener que estar pasando siempre el namespace.

k get pods --all-namespaces -> Ver todos los PODs en todos los namespaces

k get ns -> Ver todos los namespaces
```

Los **PODS** se crean por defecto en el **namespace por defecto**, para crearlos en otros puede ser a partir del comando o añadiendo en el Metadata:
- `namespace: name`

#### YAML Namespace
``` yaml
apiVersion: v1
kind: Namespace
metadata:
	name: dev
```
##### YAML ResourceQuota
```
apiVersion: v1
kind: ResourceQuota
metadata:
	name: compute-quota
	namespace: dev
spec:
	hard:
		pods: "10"
		request.cpu: "4"
		request.memory: 5Gi
		limits.cpu: "10"
		limits.memory: 10Gi
```

