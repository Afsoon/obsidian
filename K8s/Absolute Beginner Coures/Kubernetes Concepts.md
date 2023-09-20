# Pods
El objetivo de Kubernets es desplegar nuestros contenedores en nuestros worker nodes pero nosotros nunca trataremos directamente con los **worker nodes**. K8s desplegará nuestro contenedor en una **Pod** que es una instancia de nuestra aplicación. 

Si tenemos un K8s cluster con un **Woker node** donde tenemos un solo **POD**, si el numero de usuarios se incrementa, lo que hacemos es crear un nuevo **POD**, no añadir otra instancia dentro del **POD**. Si los usuarios siguen creciendo y el **Worker node** esta al limite, lo que pasaría es que necesitamos un nuevo **Worker node** y este con un nuevo **POD**.

Los **POD** tiene una relación 1:1 con un contenedor.Esta relación no es estricta, existe casos de usos donde la relación es 1:M, por ejemplo que cada instancia de la app requiera de contenedores helpers para procesar datos. 

Definimos un **POD** con todos los contenedores que indiquemos y ya K8s se encargará de desplegarlos o destruirlos a la vez cuando toque. Estos contenedores se pueden comunicar usando `localhost` ya que se comparte la misma Network en un mismo **POD**, también comparte storage.

```kubectl run nginx --image nginx```

Crea un **POD** con una instancia de `nginx`. 

```kubectl get pods```

Nos devuelve una lista de todos los **PODs** activos.
