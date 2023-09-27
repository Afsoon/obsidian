# Pods

El objetivo de Kubernets es desplegar nuestros contenedores en nuestros worker nodes pero nosotros nunca trataremos directamente con los **worker nodes**. K8s desplegará nuestro contenedor en una **Pod** que es una instancia de nuestra aplicación. 

Si tenemos un K8s cluster con un **Woker node** donde tenemos un solo **POD**, si el numero de usuarios se incrementa, lo que hacemos es crear un nuevo **POD**, no añadir otra instancia dentro del **POD**. Si los usuarios siguen creciendo y el **Worker node** esta al limite, lo que pasaría es que necesitamos un nuevo **Worker node** y este con un nuevo **POD**.

Los **POD** tiene una relación 1:1 con un contenedor.Esta relación no es estricta, existe casos de usos donde la relación es 1:M, por ejemplo que cada instancia de la app requiera de contenedores helpers para procesar datos. 

Definimos un **POD** con todos los contenedores que indiquemos y ya K8s se encargará de desplegarlos o destruirlos a la vez cuando toque. Estos contenedores se pueden comunicar usando `localhost` ya que se comparte la misma Network en un mismo **POD**, también comparte storage.

```kubectl run nginx --image nginx```

Crea un **POD** con una instancia de `nginx`. 

```kubectl get pods```

| Columna  | Explicación                                                                      |
| -------- | -------------------------------------------------------------------------------- |
| Name     | Nombre del Pod                                                                   |
| Ready    | Numero de contenedores en estado ready/ Numero total de contenedores en el POD   |
| Status   | Estado de los contenedores, si hay varios y alguno en error sale en error Reason |
| Restarts | Cuantas veces se ha reiniciado                                                   |
| Age      | Cuanto tiempo lleva activo                                                       |


Nos devuelve una lista de todos los **PODs**, su estado, cuantos hay, uptime y veces que se ha reiniciado

```kubectl describe pod nginx```

Nos da información más detallada de un **POD** en concreto.

```kubectl get pods -o wide```

Nos da algo más de información sobre todos los **PODs**

## Declarando POD con YAML

Los ficheros para **PODs**, **Replica Sets** y demás siguen la misma estructura de fichero YAML

```yaml
apiVersion:
kind:
metadata:

spec:
```

- `apiVersion`: Es la version de la API de K8s que estamos usando para crear los objetos
- `kind`: Qué tipo de objeto queremos crear.

Los posibles valores de los puntos anteriores son

| Kind       | Version |
| ---------- | ------- |
| Pod       | v1      |
| Service    | v1      |
| ReplicaSet | apps/v1 |
| Deployment | apps/v1 |

- `metadata`: Son datos para identificar el objeto en formato de diccionario y tiene unas claves predefinidas, pero la propiedad labels puedes insertar los valores que tú quieras.
- `spec`: Son los contenedores en formato de una lista de Diccionarios.

Ejemplo de POD Yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
	labels:
		app: myapp
		type: front-end
spec:
	containers:
		- name: nginx-container
		  image: nginx
```

Para ejecutar este yaml hay que hacer `kubectl create/apply -f pod-definition.yml`

Env variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: postgres
  labels:
    tier: db-tier
spec:
  containers:
    - name: postgres
      image: postgres
      env:
        - name: POSTGRES_PASSWORD
          value: mysecretpassword
```

Si el `metadata` esta vacio, no me deja crear un recurso en K8s.
### Comandos
- Ver el contexto actual: `kubectl config current-context`
- Ver el namespace usado por el contexto: `kubectl config view --minify --output 'jsonpath={..namespace}'`
- Borrar un pod: `kubectl delete pod <name>`
- Ver todo el cluster: `kubect get all`

# Controllers

Los **Controllers** son los responsables de monitorizar los recursos de K8s.

## Replication controller & Replica Set

**Replication controller** nos ayudar a tener varias instancias ejecutando en un mismo **POD** para tener **HA**.

Si el sistema esta bajo mucha carga, **replication controller** puede ir creando mas instancias para manejar el sistema, para tener **Load Balancing & Scaling**.

**Replication controller** es el método antiguo y **Replica Set** es el nuevo.

```yaml
apiVersion: v1
kind: ReplicacionController
metadata:
	name: myapp-rc
	labels:
		app: myapp
		type: frontend

spec:
	template:
		metadata:
		  name: postgres
		  labels:
			tier: db-tier
		spec:
		  containers:
			- name: postgres
			  image: postgres
			  env:
				- name: POSTGRES_PASSWORD
				  value: mysecretpassword

	replicas: 3
	```

En  `template` tenemos que poner toda la definición del **POD** pero sin el `kind` ni `apiVersion` y luego cómo hijo directo de `spec` tenemos `replicas` que es el numero de instancias que queremos.

Con esta definición ya solo tenemos que hacer `kubectl create -f rc-definition.yml` y para ver si se ha ejecutado correctamente hay que usar `kubectl get replicationcontroller`

### Replication Set

Es casi igual pero con pequeñas diferencias.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
	name: myapp-replicaset
	labels:
		app: myapp
		type: frontend

spec:
	template:
		metadata:
		  name: postgres
		  labels:
			tier: db-tier
		spec:
		  containers:
			- name: postgres
			  image: postgres
			  env:
				- name: POSTGRES_PASSWORD
				  value: mysecretpassword

	replicas: 3
	selector:
		matchLabels:
			tier: db-tier
```

La diferencia es el nuevo atributo llamado `selector` porque al **Replica Set** le podemos pasar recursos que queremos que maneje por nosotros pero no requiere ni de HA o ALB. 

Como siempre, para ejecutar es con `kubectl create -f rs-definition.yml` y para ver si se ha ejecutado correctamente `kubectl get replicaset`

## Labels y Selectors

¿Por qué los usamos?

Imaginemos que hemos desplegado 3 instancias de nuestra aplicación front end en tres partes. Estos **PODs** están desplegados en un cluster con decenas o centenares de **PODs**. ¿Cómo puede saber nuestro **ReplicaSet** que **PODs** tiene que monitorizar? Pues a través de los labels y selectors.

Si creamos **PODs** individualmente y luego creamos un objeto **ReplicaSet** que hace match con el `selector`, si el número de matchs es igual al número de `replicas`, la **ReplicaSet** no va a crear nuevas instancias porque ya tenemos el número necesario para ello.

Siempre es necesario incluir la definición de un **POD**, porque si los **PODs** existen de antes y luego en el futuro falla alguno, la **ReplicaSet** tiene que saber como crear un nuevo **POD**.

## Scale

¿Cómo escalar?

- Cambiando la definición manualmente y reaplicando la definición `kubectl replace -f replicaset-definition.yml`
- `kubectl scale --replicas=6 -f replicaset-definition-yml`
- `kubect scale --replicas=6 replicaset myapp-replicaset`
- Es posible basado en la carga pero más adelante.

## Notas de la demo

Si creamos pods a manos que superen el número de `replicas`, la **ReplicaSet** controller automáticamente borrará esos **PODS**.

Podemos usar `kubectl edit replicaset myapp-replicaset` y este nos abre nuestro editor favorito para modificar el fichero. Todos los cambios guardados se **sincronizán automaticamente** con el cluster de K8s.

# Deployments

Estrategias de manejo de **deployments**:

- Rolling updates: Ir de uno en uno hasta cambiar la version de todos los PODs
- Rollback: Si la actualización es fallida, poder volver a la versión antigua.
- Cambiar el **environment** parando las instancias y luego reanunando los PODs.

## Definition YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
	name: myapp-deployment
	labels:
		app: myapp
		type: front-end
spec
	template:
		metadata:
			name: myapp-pod
			labels:
				app: myapp
				type: front-end
		spec:
			containers:
			- name: nginx-container
			  image: nginx
	replicas:3
	selector:
		matchLabels:
			type: front-end
```

Luego como siempre ejecutamos `kubectl create -f deployment-definition.yml`

Esto crear un recurso llamado `Deployment` que a la vez crea un `ReplicaSet` y esto a su vez los `PODs` indicados.

### Soluciones

Puedes crear un comando mediante cli por ejemplo:

`kubectl create deployment httpd-frontend --image=http:2.4-alpine --replicas=3`

Crea un environment con 3 **PODs** en un **Replica Set** bajo el entorno del **Deployment** `http-frontend`
## Update & Rollback

Cada vez que creamos un `Rollout` K8s crea una versión del estado actual, para permitirnos hacer rollback en caso de error. Para el estado de un rollout tenemos que hacer `kubectl rollout status deployment/myapp-deployment`. Para ver el histórico de versiones de un **deployment** es con `kubectl rollout history deployment/myapp-deployment`.

Hay dos estrategias de **deployment**:

- **Recreate**: Consiste en acabar con todos los **PODs** y luego ejecutar la nueva versión de los **PODs**. Uno de las pegas que hay caída del servicio durante el tiempo que estamos destruyendo y recreando los **PODs**.
- **Rolling Update** (por defecto): Vamos cambiando poco a poco los **PODs** por la nueva versión.

Una forma de empezar el proceso de **Rollout** es:

- Modificar el yml
- Lanzar `kubectl apply -f deployment-definition.yml` para actualizar el deployment.

Si queremos modificar la imagen de un **POD** podemos usar la CLI directamente:

`kubectl set image deployment/myapp-deployment nginx-container=nginx.1.9.1`

Pero esto no modifica el fichero, sino solo el deployment que esta ejecutandose en el momento.

### ¿Cómo funciona **Rolling Update**?

K8s por debajo creara un nuevo **ReplicaSet** y a la vez que crea un nuevo **POD** en la nueva **ReplicaSet**, se va destruyendo otro **POD** en el antiguo **ReplicaSet**. Asi tantas veces como **PODs** hay y una vez que el antiguo esta 0, es destruido el objeto.

Podemos configurar el % de PODs que destruimos para ir haciedo el rollout de los **PODs**.

### Rollback

Si vemos que hay un problema con la nueva versión podemos hacer `kubectl rollout undo deployt/myapp-deployment` para volver a la versión antigua.

### Demo

Por defecto el motivo de los cambios no se guarda en el registro, para forzar guardar el motivo de los cambios hay que hacer:

`kubectl create -f deployment.yaml --record`

**¿No es algo que se puede añadir a posteriori?**, tiene que estar desde el principio.

Si usamos `kubectl edit` también le tenemos que pasar el argumento `--record`, por defecto no va a ir guardando todos los cambios en el histórico. Con `set image` pasa lo mismo.

# Networking

Las IPs se asignan a un **POD** y no a un contenedor, en el caso de crear un **POD** con varios contenedores. Cada vez que se recrea un **POD** cambia su IP, asi que **NO** es buena idea usar una IP hardcodeada.

Cuando estamos en un sistema multi nodo, K8s no hace nada por crear un Networking, es más, espera que nosotros le indiquemos como funciona la red con los siguientes requisitos:

- Todos los **PODs/Containers** se puede comunicar con el resto y viceverse sin **NAT**
- Todos los **Containers/PODs** se puede comunicar con uno sin **NAT**

Servicios como Calico o Flannel si estamos en entornos **Self hosted**

# Services

Los servicios permite la comunicación entre **componentes** y con el mundo exterior. **Services** permite hacer un montón de cosas pero vamos a centrarnos en un caso de uso, conectarnos a una WebApp que tenemos en un **POD** desde el exterior.

### ¿Cómo nos conectamos a un POD?

El nodo de K8s tiene una IP asignada que es `192.168.1.2` y el **host** tiene la IP `192.168.1.10` y la IP interna del **POD** es `10.244.0.2` y la red interna es `10.244.0.0`. Por defecto, no podemos conectarnos directamente a la IP del **POD** porque es una red distinta. 

Si hacemos un curl a la IP del **POD**, recibimos respuesta pero en el momento que creemos otro **POD** la IP va a cambiar. Lo que queremos es hacer una petición al nodo de K8s y que este nos devuelva el resultado del antiguo `curl`.

Esto ultimo se consigue usando un **Service** donde exponemos un puerto. El servicio escuchará las peticiones en ese puerto y reenviará esa petición a la red interna del **POD** y al puerto indicado. Esto servicios son llamadas **NodePort Service**.

## Tipos de servicios

- **NodePort**: Expone un puerto en el nodo para comunicarnos con un **POD**.
- **ClusterIP**: Crea un servicio dentro del cluster que permite la comunicación entre servicios.
- **LoadBalancer**: Un servicio que redistribuye la carga entre distintos **PODs**
## NodePort

Un servicio de tipo **NodePort** consta de las siguientes partes:

- **TargetPort**: Puerto del **POD** donde espera recibir las peticiones.
- **Port**: Puerto del **Service** que se enlazará con él **TargetPort**.
- **NodePort**: Puerto del K8s node que se expone, por defecto esta en el rango de 30.000 - 32.767
- **ClusterIP**: IP del **Service**.

Ahora pasamos a como definir un servicio

```yaml
apiVersion: v1
kind: Service
metadata: 
	name: myapp-service
spec:
	type: NodePort
	ports:
		- targetPort: 80
		  port: 80
		  nodePort: 30008
	selector:
		app: myapp
		type: front-end
```

Si no especificamos `port` por defecto usa el valor de `targetPort`. Si no especificamos `nodePort`, se selecciona uno automáticamente empezando por el 30.000.

Igual que **ReplicaSet** y **Deployment**, los **Services** usando `selector` para saber con que **PODs** se tienen que conectar.
### IDE
Puedes configurar schemas en la app de YAML para tener validators custom

## References
[K8s official documentation](https://kubernetes.io/docs/concepts/workloads/pods/)