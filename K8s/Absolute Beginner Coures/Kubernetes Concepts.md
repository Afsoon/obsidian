# Pods
El objetivo de Kubernets es desplegar nuestros contenedores en nuestros worker nodes pero nosotros nunca trataremos directamente con los **worker nodes**. K8s desplegará nuestro contenedor en una **Pod** que es una instancia de nuestra aplicación. 

Si tenemos un K8s cluster con un **Woker node** donde tenemos un solo **POD**, si el numero de usuarios se incrementa, lo que hacemos es crear un nuevo **POD**, no añadir otra instancia dentro del **POD**. Si los usuarios siguen creciendo y el **Worker node** esta al limite, lo que hacemos es crear un nue
