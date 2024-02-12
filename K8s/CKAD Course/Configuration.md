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
