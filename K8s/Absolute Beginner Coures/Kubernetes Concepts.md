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
			type: front-end
```

La diferencia es el nuevo atributo llamado `selector` porque al **Replica Set** le podemos pasar recursos que queremos que maneje por nosotros pero no requiere ni de HA o ALB. 
### IDE
Puedes configurar schemas en la app de YAML para tener validators custom

## References
[K8s official documentation](https://kubernetes.io/docs/concepts/workloads/pods/)