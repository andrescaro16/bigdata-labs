# ST0256 Tópicos Especiales en Telemática - Laboratorios

## Estudiante
- **Nombre:** Andrés Julian Caro Restrepo
- **Correo:** ajcaror@eafit.edu.co

## Profesor
- **Nombre:** Alvaro Enrique Ospina Sanjuan
- **Correo:** aeospinas@eafit.edu.co

------------
<br/>

# Contenido
- [EMR Setup](#emr-setup)
- [Laboratorio 1](#)
- [Laboratorio 2](#)
- [Laboratorio 3](#)

------------
<br/>

# EMR Setup

Para la creación del clúster de Amazon EMR que utilizaremos para el desarrollo de los laboratorios seguiremos los siguientes pasos:


## 1. Crear bucket S3
Una vez nos encontremos en la consola de AWS buscaremos el servicio de S3
![AWS homepage](./EMR-Setup-images/00.png)
![AWS homepage](./EMR-Setup-images/01.png)

Nos dirigimos a la sección de buckets y allí le daremos click a `Create Bucket`.
![AWS homepage](./EMR-Setup-images/02.png)

Le daremos un nombre al bucket. Éste debe contener de 3 a 63 caracteres, ser único en el mundo y no debe contener mayúsculas. Para nuestro laboratorio hemos decidido nombrarlo `ajcarornotebooks`.
![AWS homepage](./EMR-Setup-images/03.png)

Para el desarrollo, hemos desactivado la opción de `Block all public access`.
![AWS homepage](./EMR-Setup-images/04.png)

Luego de esto procedemos a crear el bucket. Lo cual se debe ver reflejado de la siguiente manera:
![AWS homepage](./EMR-Setup-images/05.png)


## 2. Crear clúster EMR
Ya con nuestro bucket creado ahora sí podremos crear nuestro clúster de EMR. Para ello, nos dirigimos a dicho servicio.
![AWS homepage](./EMR-Setup-images/06.png)

En la sección de `Clústers` podemos ver los que hemos creado con anterioridad (si es el caso). Le damos click a `Create clúster`.
![AWS homepage](./EMR-Setup-images/07.png)

Aquí le daremos un nombre a nuestro clúster, seleccionaremos la versión de Amazon EMR y las aplicaciones que queremos sean instaladas en el clúster. Para nuestro caso, elegimos:
+ **Name:** `bigdata-labs-cluster`
+ **Versión:** `emr-6.14.0`
+ **Aplicaciones:**
  - [x] `Flink 1.17.1`
  - [x] `HCatalog 3.1.3`
  - [x] `Hue 4.11.0`
  - [x] `Livy 0.7.1`
  - [x] `Spark 3.4.1`
  - [x] `Tez 0.10.2`
  - [x] `ZooKeeper 3.5.10`
  - [x] `Hadoop 3.3.3`
  - [x] `JupyterEnterpriseGateway 2.6.0`
  - [x] `Sqoop 1.4.7`
  - [x] `Hive 3.1.3`
  - [x] `JupyterHub 1.5.0`
  - [x] `Zeppelin 0.10.1`

Además, es muy importante que activemos las casillas:
- `Use for Hive table metadata`
- `Use for Spark table metadata`

El resto de opciones las dejamos con su valor por defecto.
![AWS homepage](./EMR-Setup-images/08.png)

Aquí seleccionamos los tipos de instancias que deseamos para el clúster. Podemos dejarla en las máquimnas default `m5.xlarge`, sin embargo, con éstas en ocasiones la creación del clúster falla porque dependemos de la disponibilidad de las mismas. Es por ello que en nuestro caso hemos elegido las `m4.xlarge`.
![AWS homepage](./EMR-Setup-images/09.png)

Dejamos las siguientes opciones con sus valores por defecto:
![AWS homepage](./EMR-Setup-images/10.png)

Y en el apartado de `Software settings` enlazaremos el bucket s3 que creamos en el [paso 1](#1-crear-bucket-s3).
Para ello, añadiremos el siguiente json:
```json
[
  {
    "Classification": "jupyter-s3-conf",
    "Properties": {
      "s3.persistence.enabled": "true",
      "s3.persistence.bucket": "ajcarornotebooks"
    }
  }
]
```
**Nota:** Asegurate de reemplazar `ajcarornotebooks` por el nombre de tu bucket s3.
![AWS homepage](./EMR-Setup-images/11.png)

Asignamos una key pair. Puedes seleccionar una ya existente o crear una nueva.
![AWS homepage](./EMR-Setup-images/12.png)

En esta sección seleccionamos los roles con los que vamos a trabajar. En nuestro caso, hemos seleccionado los siguientes:
- **Service role:** `EMR_DefaultRole`
- **Instance profile:** `EMR_EC2_DefaultRole`
- **Custom automatic scaling role:** `LabRole`

![AWS homepage](./EMR-Setup-images/13.png)

Revisamos el resumen de la configuración de instalación del clúster y si todo se encuentra acorde a lo que deseamos le damos en `Create cluster`.

![AWS homepage](./EMR-Setup-images/14.png)


## 3. Configuración de acceso al clúster

La creación del clúster puede tomar entre 12 y 20 minutos. Mientras se realiza su instalación, podemos dirigirnos a `Block public access`, darle en `Edit` y `Turn off` para permitir la apertura de los puertos necesarios para las aplicaciones. Le damos en `Save`. Tener en consideración que esto no se debe realizar en un entorno de producción, pues lo debido en dichos ambientes es tener la seguridad pertinente.
![AWS homepage](./EMR-Setup-images/15.png)

Una vez el clúster se haya creado lo debemos ver algo así:
![AWS homepage](./EMR-Setup-images/16.png)

Y ya podremos dirigirnos hacia EC2 para configurar los grupos de seguridad de las instancias.
![AWS homepage](./EMR-Setup-images/17.png)

Identificamos la instancia `master` gracias a la columna `Security group name` y entramos en ella.
![AWS homepage](./EMR-Setup-images/18.png)

Aquí nos dirigiremos hacia la pestaña de `Security` y al Security group en cuestión.
![AWS homepage](./EMR-Setup-images/19.png)

Entramos a `Edit inbound rules` y añadimos la regla `All traffic`. (Reiterar que no pertinente para producción).
![AWS homepage](./EMR-Setup-images/20.png)


## 4. Parche de acceso a Hue

Siempre que iniciemos el clúster debemos realizar este proceso para arreglar el acceso a Hue. Para ello, nos conectaremos mediante SSH a nuestra instancia `master` y editaremos el archivo `hue.ini` con el siguiente comando:

```bash
vim /etc/hue/conf/hue.ini
```
![AWS homepage](./EMR-Setup-images/21.png)

Aquí buscaremos la linea que necesitamos editar y lo haremos con:

```bash
/webhdfs_url
```
![AWS homepage](./EMR-Setup-images/22.png)

Al encontrar la linea que necesitamos presionaremos `Enter`, luego la tecla `I` y cambiaremos el puerto `14000` por el `9870`. Con lo cual se nos debe ver así:
![AWS homepage](./EMR-Setup-images/23.png)

Una vez realizado el cambio presionamos `ESC` y digitamos `:x` para guardar y salir.
Después, reiniciaremos el servicio con el comando:
```bash
sudo systemctl restart hue.service
```


## 5. Validar acceso correcto a las aplicaciones

Para verificar que todo se encuentra en orden podemos entrar a Hue, JupyterHub y Zeppelin para probar el acceso. Para ello nos dirigimos nuevamente a nuestro clúster e ingresamos a la pestaña de `Applications`, donde podremos encontrar los diferentes enlaces para cada aplicación.
![AWS homepage](./EMR-Setup-images/24.png)

Entraremos al enlace de `Hue` y en esta interfaz desdes nuestro browser debemos ingresar `hadoop` como usuario y crear nuestra propia contraseña que cumpla con los criterios.
![AWS homepage](./EMR-Setup-images/25.png)

Una vez dentro podremos acceder, por ejemplo, al apartado de s3 y ver que allí se encuentra enlazado el bucket que creamos.
![AWS homepage](./EMR-Setup-images/26.png)

Ahora intentemos ingresar a JupyterHub siguiendo su respectivo enlace. Aquí ingresaremos con `jovyan` como usuario y `jupyter` como contraseña.
![AWS homepage](./EMR-Setup-images/27.png)

Ahora podemos crear un nuevo notebook de PySpark.
![AWS homepage](./EMR-Setup-images/28.png)

Y podemos observar como nuestro ambiente se ejecuta de manera correcta.
![AWS homepage](./EMR-Setup-images/29.png)

Ahora, dirigiendonos al enlace de Zeppelin podremos ingresar de manera directa como anónimo (sin necesidad de login).
![AWS homepage](./EMR-Setup-images/30.png)

Podemos crear un nuevo notebook con `spark` como intérprete.
![AWS homepage](./EMR-Setup-images/31.png)

Y de forma similar a como lo hicimos en JupyterHub, validar la correcta ejecución de las variables.
![AWS homepage](./EMR-Setup-images/32.png)

Siendo así, ya hemos configurado correctamente nuestro clúster.


## 6. Clonar clúster EMR

Cada vez que el clúster se termine no se puede volver a iniciar, es por esto que debemos ya sea crear uno nuevo, o como podemos ver, clonar uno que existió. Lo único que debemos hacer es seleccionar el cluster en estado "Terminated" y darle en `Clone`.
![AWS homepage](./EMR-Setup-images/33.png)

Aquí podremos cambiar alguna configuración en caso de requerirlo y luego dar click en `Clone cluster`.
![AWS homepage](./EMR-Setup-images/34.png)

De la misma forma a la creación, tendremos que esperar a que se inicialice nuestro clúster. Una vez terminado, debemos repetir el proceso de [parche de acceso a hue](#4-parche-de-acceso-a-hue) y crear la cuenta de Hue.
![AWS homepage](./EMR-Setup-images/35.png)

------------
<br/>
