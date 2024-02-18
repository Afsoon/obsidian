## Commands and Arguments
### Docker

Cuando estamos creando un Dockerfile y queremos permitir al usuario que nos pase argumentos **custom**, tenemos que acabar con el fichero de la siguiente forma:

```
[.....Dockerfile]

ENTRYPOINT ["sleep"]

CMD ["5"]
```

La anterior definición hace lo siguiente:
- El usuario ejecutar la imagen como `docker run sleeper` : entonces el comando que se ejecuta es `sleep 5`, es decir, al no haber ningún input se usa el valor de `CMD`.
- El usuario ejecuta la imagen como `docker run sleeper 15`: El comando que el contenedor ejecutará es `sleep 15`, hace una sobrescritura de `CMD`.
- El usuario ejecuta la imagen como `docker run --entrypoint sleep2.0 sleeper 10`: El comando que el contenedor ejecutará es `sleep2.0 10`

### K8s

Si quisiéramos lanzar un **POD** modificando tanto el **entrypoint** como el **CMD** sería de la siguiente forma:
```
apiVersion: v1
kind: Pod
metadata:
	name: sleeper
spec:
	containers:
		- name: sleeper
		  image: sleeper
		  command: ["sleep2.0"] // Es el entrypoint
		  args: ["10"] // Es el CMD
```

### Labs
```
k replace --force -f <filename>

k run web --image <image> -- <args> -> Hace un override del CMD de Docker

k run web --image <image> --command -- <cmd> <args>
```

## Environment Variables
### ConfigMaps

Podemos definir las env directamente en el POD pero a medida que crezca el proyecto se vuelve más engorroso. K8s nos da la posibilidad de crear **ConfigMaps** que son ficheros en formato KV. Los **Configmaps** tienen la siguiente estructura YAML:
```
apiVersion: v1
kind: ConfigMap
metadata:
	name: app-config
data:
	APP_COLOR: BLUE
```


La forma imperativa, es más incómodo, que es la siguiente:
```
Imperativo y KV directamente:

k create configmap app-config \
 --from-literal=APP_COLOR=BLUE

Imperativo y a partir de un fichero

k create configmap app-config --from-file=<name>.properties
```

Luego para inyectar todo el **ConfigMap** en el POD es de la siguiente forma:
```
apiVersion: v1
kind: Pod
metadata:
	name: example
spec:
	containers:
		- name: app
		  image: app
		  envFrom:
			  - configMapRef:
				  name: app-config
```

Si queremos insertar solo algunas Keys en el POD es de la siguiente forma:
```
apiVersion: v1
kind: Pod
metadata:
	name: example
spec:
	containers:
		- name: app
		  image: app
		  env:
			  - name: APP_COLOR
				configMapKeyRef:
				  name: app-config
				  key: APP_COLOR
```

> [!WARNING] Cuando toquemos Volumes
> Añadir esto en volumes en como inyectar un **ConfigMap**
> configMap:
> 	name: app-config
### Labs
```
apiVersion: v1
kind: Pod
metadata:
	name: example
spec:
	containers:
		- name: app
		  image: app
		  env:
			  - name: APP_COLOR
				valueFrom:	
					configMapKeyRef:
						name: webapp-app-config
						key: APP_COLOR
```


## Secrets

Es un KV donde el value esta guardado en **Base64**, es decir, no es un sistema seguro para guardar valores. El objetivo es no tener los valores en plano. Los **Secrets** tienen la siguiente estructura YAML:
```
apiVersion: v1
kind: Secret
metadata:
	name: app-secret
data:
	APP_COLOR: <base64value>
```


La forma imperativa, es más incómodo, que es la siguiente:
```
Imperativo y KV directamente:

k create secret generic app-secret \
 --from-literal=APP_COLOR=BLUE

Imperativo y a partir de un fichero

k create secret generic app-secret --from-file=<name>.properties
```

Para convertir la información en texto plano, en la CLI se puede hacer lo siguiente:
```
echo -n "text" | base64
```

Luego para inyectar todo el **ConfigMap** en el POD es de la siguiente forma:
```
apiVersion: v1
kind: Pod
metadata:
	name: example
spec:
	containers:
		- name: app
		  image: app
		  envFrom:
			  - secretRef:
				  name: app-secret
```

Si queremos insertar solo algunas Keys en el POD es de la siguiente forma:
```
apiVersion: v1
kind: Pod
metadata:
	name: example
spec:
	containers:
		- name: app
		  image: app
		  env:
			  - name: APP_COLOR
				secretKeyRef:
				  name: app-secret
				  key: APP_COLOR
```

> [!Warning] Secrets en Volumes
> Por cada secreto inyectado en un volume, se crea un fichero correspondiente
### Notes
- Los secretos no estas encriptados, asi que hay que evitar ser **comiteados** en el git repository.
- Los secretos se ve en plano en **ETCD**
	- Puedes encriptarlo activando **Encryption At Rest**
- Cualquiera que pueda crear **PODs**/**Deployments** en el mismo **namespace** puede acceder a los secretos.
	- Fix: Configurar un **RBAC** - Least-Privilege access to Secrets
- Para guardar secretos con más seguridad, mejor usar servicios de terceros dedicado a ellos.
<details>
	<summary> Furthermore about secrets </summary>
Remember that secrets encode data in base64 format. Anyone with the base64 encoded secret can easily decode it. As such the secrets can be considered as not very safe.

The concept of safety of the Secrets is a bit confusing in Kubernetes. The [kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/secret) page and a lot of blogs out there refer to secrets as a "safer option" to store sensitive data. They are safer than storing in plain text as they reduce the risk of accidentally exposing passwords and other sensitive data. In my opinion it's not the secret itself that is safe, it is the practices around it. 

Secrets are not encrypted, so it is not safer in that sense. However, some best practices around using secrets make it safer. As in best practices like:

- Not checking-in secret object definition files to source code repositories.    
- [Enabling Encryption at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) for Secrets so they are stored encrypted in ETCD. 

Also the way kubernetes handles secrets. Such as:

- A secret is only sent to a node if a pod on that node requires it.
- Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.
- Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.

Read about the [protections](https://kubernetes.io/docs/concepts/configuration/secret/#protections) and [risks](https://kubernetes.io/docs/concepts/configuration/secret/#risks) of using secrets [here](https://kubernetes.io/docs/concepts/configuration/secret/#risks)

Having said that, there are other better ways of handling sensitive data like passwords in Kubernetes, such as using tools like Helm Secrets, [HashiCorp Vault](https://www.vaultproject.io/). I hope to make a lecture on these in the future.
</details>
### Labs
Nada que anotar de las soluciones

## Security Context
### Docker Security

#### Cosas que sabía
- Docker por defecto ejecuta todo como root user, y que puedes configurarlo para que se ejecute con otro user.
- Los containers estas aislados a través de **namespaces**.
#### Cosas que no sabía
- Que puedes quitar y añadir los privilegios de root, Docker ya quitas algunos que puede ser destructivos para los contenedores
### K8s
La configuración de **Docker Security** se puede crear desde **K8s** donde tenemos la posibilidad de:
- Crear un contexto padre donde afecte a todos los **containers** que se crean dentro de un **POD**, de la siguiente forma:
```
apiVersion: v1
kind: Pod
metadata:
	name: web-pod
spec:
	securityContext:
	  runAsUSer: 1000
	containers:
		- name: ubuntu
		  image: ubuntu
		  command: []
```
- Modificar solo a nivel de **container** los privilegios del **root user**, de la siguiente forma:
```
apiVersion: v1
kind: Pod
metadata:
	name: web-pod
spec:
	containers:
		- name: ubuntu
		  image: ubuntu
		  command: []
		  securityContext:
			  runAsUSer: 1000
			  capabilities:
				  add: ["MAC_ADMIN"] 
```

> [!Warning] La propiedad **capabilities**
> La propiedad **capabilites** solo se puede modificar a nivel de **container** y no a nivel de **POD**.

#### Labs
- Los **security context** se puede mezclar tanto a nivel de **POD** como de **container**, se hace un merge de propiedades donde las más cercanas tienen prioridad.
- Para ejecutar como root un **container**, hay que omitir `runAsUser`, no vale con poner 1.
- `k exec <pod-name> -- whoami` para saber el usuario que esta ejecutando el **container**.
## ServiceAccounts
Hay dos tipos de cuentas en K8s,

- User Accounts
- Service accounts: Estas cuentas interactuan con las API de K8s, por ejemplo **Prometheus** para sacar métricas, **Jenkins** para desplegar de forma automatizada.

Para crear un **ServiceAccount** hay que hacer lo siguiente:
```
k create serviceaccount dashboard-sa

k get serviceaccount
```

Cuando creamos un **ServiceAccount**, un token es creado **automáticamente** y este es el que deben de usar los servicios. Para ver el token podemos hacer:

```
k describe serviceaccount dashboard-sa

Tokens: dashboard-sa-token-kbbdm
// Se mostrará el nombre del secreto, NO el valor
```
Este token esta guardado en un **Secret**, por lo tanto, para ver el valor del token tenemos que ver la información del **Secret** con:
```
k describe secret <service-account-secret-token-name>
k describe secret Tokens: dashboard-sa-token-kbbdm
```
Este **Token** es usado como un **Bearer** **authentication**. 

Los **ServiceAccounts** se le puede asignar un sistema de roles de accesos (RBAC).

El sistema definido es para los sistemas esta fuera K8s para comunicarse, pero si los servicios se encuentran dentro del cluster. ¿Existe alguna forma más fácil de inyectar el secretos en los **PODs**?. La respuesta es si, estos secretos se inyectan como un volumen en los **PODs** y estos ficheros se encuentra en la siguiente ruta:

- `/var/run/secrets/kubernets.io/serviceaccount/`

Cuando estamos definiendo un **POD**, tenemos que indicar que **ServiceAccount** queremos relacionarle, de la siguiente forma:

```
apiVersion: v1
kind: Pod
metadata:
	name: example-pod-sa
spec:
	containers:
		- name: my-k8s-dashboard
		  containers: my
	serviceAccountName: dashboard-sa
```

K8s por defecto añade el **ServiceAccount** por defecto, si no queremos que haga esto, tenemos que indicar en el **POD** de la siguiente forma:

```
apiVersion: v1
kind: Pod
metadata:
	name: example-pod-sa
spec:
	containers:
		- name: my-k8s-dashboard
		  containers: my
	automountServiceAccountToken: false
```

### Cambios en K8s 1.22 & 1.24

Pre 1.22, los tokens no estaban limitados de ninguna forma, en otras palabras, era válido todo el tiempo que el **ServiceAccount** existía. Esto generaba problemas de escabilidad, necesitaba un **Secret object** por cada token, y de seguridad, tiempo, usuarios.

En K8s 1.22 (KEP 1205) se introdujo la **TokenRequestAPI** donde estos tokens si estaban limitados por:
- Audiencia
- Tiempo
- Object

A partir de este momento los **PODs** crean un **Projected volumen** donde se puede ver una expiración del token.

En K8s 1.24 (KEP 2799) cuando creamos un **ServiceAccount** ya no se crea un **Secret** con un token. Si queremos un token, tenemos que generarlo de la siguiente forma:

```
k create token dashboard-sa

// Mostrará por pantalla el token
```

Estos tokens generados **ya tienen una fecha de expiración**. El comportamiento antiguo no ha sido borrado, se queremos el antiguo flow tenemos que hacer lo siguiente:

```
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
	name: mysecretname

	annotations:
		kubernets.io/service-account.name: dashboard-sa
```

Pero este método ya solo se deberías de usar si el uso de Tokens que no expiran es aceptable para tu caso de uso, en cualquier otra situación, mejor usar el nuevo flow.

### Labs
