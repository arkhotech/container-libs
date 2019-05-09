# Spark Container

This container implements Spark 2.3.1 version. By default when you run "docker run" without any command, this container runs spark's python shell, then in order to do this, execure the following command:

```
docker run -ti --rm arkhotech/spark-python
```

## Volumes

You could mount some volumes by to get the python scripts and some other volume by to get the input data files into the container. Example:

```
docker run -ti --rm -v LOCAL_DIR:/opt/scripts arkhotech/spark-python 
```


##Cluster 

The continer was configured by able to execute on cluster mode (master and worker) that you can execute on Docker compose, docker swarm or Kubernetes (we don't put the kubernetes script on this version). The following YAML has got the docker compose configuration by executes on cluster mode.

```yaml
version: "3.0"

services:
  master:
    image: "arkhotech/spark-python"
    hostname: master
    ports:
     - 8080:8080 
    volumes:
     - ./input:/opt/input
     - ./spark-env.sh:/opt/spark/conf/spark-env.sh
     - ./script:/opt/scripts
     - ./spark.properties:/opt/spark/conf/spark-defaults.conf 
    networks:
     - sparknet
    command: ["/opt/spark/bin/spark-class","org.apache.spark.deploy.master.Master"]

  node:  
    image: "arkhotech/spark-python"
    networks:
     - sparknet  
    volumes:
     - ./input:/opt/input
     - ./spark-env.sh:/opt/spark/conf/spark-env.sh
     - ./spark.properties:/opt/spark/conf/spark-defaults.conf 
    depends_on: 
     - master    
    command: ["/opt/spark/bin/spark-class","org.apache.spark.deploy.worker.Worker","spark://master:7077"]
networks:
   sparknet:
      driver: bridge  #Cambiar por Overlay si esta en un Swarm

```

# Probar la configuración

Para probar la configuración:

```
docker exec -ti docker-spark-python_master_1 spark-submit --master spark://master:7077 PATH_TO_SCRIPT
```


* PATH_TO_SCRIPT * : Es la ruta dentro del contenedor donde se encuentra el script a ejecutar. Considerar montar un volumen externo.


# AWS

Este contendor tiene consideradas las librerías necesarias para trabajar con AWS, por lo que se puede acceder a repostorios S3. Esta caracteristica permite generar un ambiente de desarrollo local para trabajar con EMR.  Para usar esta funcionalidad, agreagr las variables de ambiente de AWS para el __Access Key__ y el __Secret Key__


## Spanish


Este contenedor implementa la aplicación Spark versión 2.3.1. La ejecución de este contendor disponibiliza la shell de python. Para iniciar el contendor, simplemente ejecutar el siguiente comando:

##Volumenes

Los scripts se deben montar en el contanedor para poder acceder a ellos, a menos que se ejecute desde otro shell spark. Para montar un directorio usar el siguiente comando:

## Ejecución en cluster

El container esta diseñado para ejecutrase como cluster local o un orquestador como Swerm o Kubernates.  Para lograr esto, usar el siguiente docker-composer.yml
