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


## Cluster 

The continer was configured by able to execute on cluster mode (master and worker) that you can execute on Docker compose, docker swarm or Kubernetes (we don't put the kubernetes script on this version). The following YAML has got the docker compose configuration by executes on cluster mode.

```yaml
version: "3.0"

services:
  master:
    image: "arkhotech/spark-python:2.1.0"
    hostname: master
    ports:
     - 8080:8080 
    environment:
     - SPARK_MASTER_HOST='master'
    volumes:
     - ./input:/opt/input
     - ./script:/opt/scripts
    networks:
     - sparknet
    command: ["/opt/spark/bin/spark-class","org.apache.spark.deploy.master.Master"]

  node:  
    image: "arkhotech/spark-python:2.1.0"
    networks:
     - sparknet  
    environment:
     - SPARK_MASTER_HOST='master'
    volumes:
     - ./input:/opt/input
    depends_on: 
     - master    
    command: ["/opt/spark/bin/spark-class","org.apache.spark.deploy.worker.Worker","spark://master:7077"]
networks:
   sparknet:
      driver: bridge  #Cambiar por Overlay si esta en un Swarm

```
On the script there is severals volumes:

* if you want chenge the name of the container master, you must especify the environment var SPARK_MASTER_HOST with the new name. By default is "master". 
* The **input** directory it's mounted on **/opt/input** and it's the input for data on the example, could be another one directory.
* the environment variables are for load AWS libreries and to be able work with S3 and Redshift.
* if you want to work with AWS, you must create some especific configuration for it. You must crete a file and copy all the following content and save:

```properties
spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem  
spark.hadoop.fs.s3.impl=org.apache.hadoop.fs.s3.S3FileSystem
spark.hadoop.fs.s3n.impl=org.apache.hadoop.fs.s3native.NativeS3FileSystem
spark.executor.extraClassPath=/opt/hadoop-2.8.3/share/hadoop/tools/lib/aws-java-sdk-core-1.10.6.jar:/opt/hadoop-2.8.3/share/hadoop/tools/lib/hadoop-aws-2.8.3.jar
spark.driver.extraClassPath=/opt/hadoop-2.8.3/share/hadoop/tools/lib/aws-java-sdk-core-1.10.6.jar:/opt/hadoop-2.8.3/share/hadoop/tools/lib/hadoop-aws-2.8.3.jar
```

Once you has saved the file you must load like a volume on both containers (master and worker) like:

```yaml
  volume:
    - ./spark.properties:/opt/spark/conf/spark-defaults.conf 
```

## Cluster with AWS capabilities



```yaml
version: "3.0"

services:
  master:
    image: "arkhotech/spark-python:2.1.0"
    hostname: master
    ports:
     - 8080:8080 
    volumes:
     - ./input:/opt/input
     - ./script:/opt/scripts
     - ./spark.properties:/opt/spark/conf/spark-defaults.conf 
    networks:
     - sparknet
    command: ["/opt/spark/bin/spark-class","org.apache.spark.deploy.master.Master"]

  node:  
    image: "arkhotech/spark-python:2.1.0"
    networks:
     - sparknet  
    volumes:
     - ./input:/opt/input
     - ./spark.properties:/opt/spark/conf/spark-defaults.conf 
    depends_on: 
     - master    
    command: ["/opt/spark/bin/spark-class","org.apache.spark.deploy.worker.Worker","spark://master:7077"]
networks:
   sparknet:
      driver: bridge  #change by Overlay if you are working over Docker swarm
```


## Test the configuration

In order by test the configuration you can execute the following command:

```bash
docker exec -ti docker-spark-python_master_1 spark-submit --master spark://master:7077 PATH_TO_SCRIPT
```

where PATH_TO_SCRIPT it's the path inside the container where you has put the scripts. The workdir in the container it's "/opt" then you must consider that if you give a relative path. 
If you want put the files on another directory you must especify with absolute path.


## Use with AWS

This container has installed all the necesaries libraries for work with AWS. By example, we could access to S3. If you want to use these capability you must provide the __Access Key__ and __Secret Key__ with the appropriate permissions as a environment variables.



## Spanish


Este contenedor implementa la aplicación Spark versión 2.3.1. La ejecución de este contendor disponibiliza la shell de python. Para iniciar el contendor, simplemente ejecutar el siguiente comando:

##Volumenes

Los scripts se deben montar en el contanedor para poder acceder a ellos, a menos que se ejecute desde otro shell spark. Para montar un directorio usar el siguiente comando:

## Ejecución en cluster

El container esta diseñado para ejecutrase como cluster local o un orquestador como Swerm o Kubernates.  Para lograr esto, usar el siguiente docker-composer.yml

```yaml
version: "3.0"

services:
  master:
    image: "arkhotech/spark-python:2.1.0"
    hostname: master
    ports:
     - 8080:8080 
    environment:
     - SPARK_MASTER_HOST='master'
    volumes:
     - ./input:/opt/input
     - ./script:/opt/scripts
    networks:
     - sparknet
    command: ["/opt/spark/bin/spark-class","org.apache.spark.deploy.master.Master"]

  node:  
    image: "arkhotech/spark-python:2.1.0"
    networks:
     - sparknet  
    environment:
     - SPARK_MASTER_HOST='master'
    volumes:
     - ./input:/opt/input
    depends_on: 
     - master    
    command: ["/opt/spark/bin/spark-class","org.apache.spark.deploy.worker.Worker","spark://master:7077"]
networks:
   sparknet:
      driver: bridge  #Cambiar por Overlay si esta en un Swarm

```


## Probar la configuración

Para probar la configuración:

```
docker exec -ti docker-spark-python_master_1 spark-submit --master spark://master:7077 PATH_TO_SCRIPT
```

* PATH_TO_SCRIPT * : Es la ruta dentro del contenedor donde se encuentra el script a ejecutar. Considerar montar un volumen externo.


# AWS

Este contendor tiene consideradas las librerías necesarias para trabajar con AWS, por lo que se puede acceder a repostorios S3. Esta caracteristica permite generar un ambiente de desarrollo local para trabajar con EMR.  Para usar esta funcionalidad, agreagr las variables de ambiente de AWS para el __Access Key__ y el __Secret Key__
