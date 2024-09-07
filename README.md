
## **Actividad integrador de Kubernetes**

### **Introducción**

Este trabajo se enfoca en la migración de la aplicación **compressor-api**, desarrollada previamente en Docker, a un entorno Kubernetes. El objetivo principal es escalar la aplicación y aprovechar las ventajas de orquestación que ofrece Kubernetes.

### **Ambiente de trabajo**
- Windows
- Docker Desktop 

### **Requerimientos de Despliegue**

-   **Replicas:** Se definieron dos réplicas de la aplicación **compressor-api** para asegurar alta disponibilidad y escalabilidad.
-   **Redis:** Se desplegó un pod de Redis para servir como almacén de datos.

### **Manifiestos**
Se crearon 2 manifiestos los cuales son:
- **redis_deployment.yaml**: Define el deployment de redis y su correspondiente service [redis_deployment.yaml](https://github.com/gomezandres/k8s-practical-work/blob/main/redis_deployment.yaml)
- **compressor-app_deployment.yaml**: Define el deployment de la aplicación [compressor-app_deployment.yaml](https://github.com/gomezandres/k8s-practical-work/blob/main/compressor-app_deployment.yaml)

### **Proceso de despliegue**

A continuación se detalla el proceso de despliegue en el orden aplicado
#### **Comandos**
-  Aplicar el manifiesto de redis: ```kubectl.exe apply -f .\redis_deployment.yaml```
	- Se visualiza el siguiente mensaje en la consola
		- ```deployment.apps/redis created```
		- ```service/redis-service created```
-  Aplicar el manifiesto de la aplicación compressor-api: ```kubectl.exe apply -f .\compressor-app_deployment.yaml```
	- Se visualiza el siguiente mensaje en la consola
		- ```deployment.apps/compressor-api created```
-  Verificación de los pods creados: ```kubectl get pods```
	  ```
	  NAME 							  READY   STATUS    RESTARTS   AGE 
	  compressor-api-7869ccc689-4vstn   1/1     Running   0          33s
	 compressor-api-7869ccc689-qstkd   1/1     Running   0          33s
	 redis-6b5bcbb6b6-lpr9g            1/1     Running   0          2m38s ```
- Verificación del redis-service creado: ``` kubectl.exe get svc```
```
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP    45h
redis-service   ClusterIP   10.98.111.83   <none>        6379/TCP   16m
```
### **Prueba Local**
Para poder probar localmente la aplicación, es necesario que el mismo esté accesible.

Para ello se debe crear un túnel (o forwarding) entre la máquina local y el contenedor específico en ejecución dentro de tu cluster de Kubernetes. Esto te permite acceder al servicio que se está ejecutando dentro del contenedor como si se estuviese accediendo a él directamente desde la computadora. 

Se debe aplicar el siguiente comando:
```
kubectl port-forward compressor-api-7869ccc689-qstkd 8080:8080 -n default
```
****Por supuesto que estamos accediendo a un pod específico****

Una vez que el comando se encuentre ejecutandose, podemos acceder al swagger de la aplicación:
-	http://localhost:8080/swagger-ui.html

### **Desafíos y Soluciones**

Durante el primer despliegue, se encontró el siguiente error:

```
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'compressorServiceImpl': Invocation of init method failed; nested exception is org.springframework.data.redis.RedisConnectionFailureException: Unable to connect to Redis;   1. stackoverflow.com stackoverflow.com nested exception is io.lettuce.core.RedisConnectionException:   1. stackoverflow.com stackoverflow.com Unable to connect to redis:6379

```

Este error indicaba que aunque el pod de Redis estaba en ejecución, la aplicación **compressor-api** no podía establecer una conexión con él. Por lo tanto los pods no levantaban.

Para solucionar este problema, se realizaron las siguientes modificaciones:

-   **Configuración Dinámica:** Se modificó la aplicación **compressor-api** para que la URL y el puerto de conexión a Redis sean obtenidos de las variables de entorno. Esto permite una mayor flexibilidad y facilita la configuración en diferentes entornos.  
-   **Publicación en Docker Hub:** Se creó una imagen Docker de la aplicación actualizada y se publicó en Docker Hub para facilitar su despliegue en Kubernetes.
	- última versión: v1.2.0
-   **Manifiesto de Deployment:** Se ajustó el manifiesto de deployment de Kubernetes para pasar las variables de entorno con la URL y el puerto de Redis a los contenedores de la aplicación.
```
env:
- name: REDIS_URL
  value: redis-service
- name: REDIS_PORT
  value: "6379"
```

### **Mejoras a implementar**

Hay varias mejoras que podemos implementar para aumentar la robustez, escalabilidad y seguridad de este despliegue.

1. ***Límites y Solicitudes de Recursos***
	- Estableciendo límites y solicitudes de CPU y memoria, garantizamos que los pods no consuman más recursos de los necesarios y evitamos que afecten a otros pods en el nodo. 
		- Aplicable a compressor-api y redis
2. ***Volumen Persistente***
	- Con un PVC aseguramos que los datos se persistan incluso si el pod se reinicia o se migra a otro nodo.
		- Aplicable a redis
3. ***Sondas de Liveness y Readiness:***
	- Estas sondas permiten a Kubernetes monitorear la salud de los pods y tomar acciones como reiniciarlos o eliminarlos si están fallando.
		- Aplicable a compressor-api y redis
4. ***Estrategias de Actualización***
	- Define cómo se actualizarán los pods
		- Aplicable a compressor-api y redis
5. ***ConfigMap***
	- Utiliza ConfigMaps para gestionar la configuración de forma centralizada.
		- Aplicable a compressor-api y redis



### **Conclusiones**

La migración a Kubernetes ha permitido escalar la aplicación **compressor-api** y mejorar su gestión. Al resolver el problema de conexión con Redis, se ha garantizado el correcto funcionamiento de la aplicación en el nuevo entorno.

**Enlaces:** 
[compressor-api v1.2.0](https://hub.docker.com/layers/andresg278/compressor-api/v1.2.0/images/sha256-9064e7b447ebb7d3f27851683664c1ca895c6b73f97687e010a32a598d745324?context=repo)
