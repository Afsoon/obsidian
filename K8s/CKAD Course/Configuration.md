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


La forma imperativa es más incómodo que es la siguiente:
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
		  envFrom:
			  - configMapKeyRef:
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
