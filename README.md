# demo-camel-amq-broker-quarkus
Repositorios:

[https://github.com/SaulVazquezRedHat/demo-camel-amq.git](https://github.com/SaulVazquezRedHat/demo-camel-amq.git)

[https://github.com/SaulVazquezRedHat/camel-producer-v2.git](https://github.com/SaulVazquezRedHat/camel-producer-v2.git)

[https://github.com/SaulVazquezRedHat/camel-consumer-v2.git](https://github.com/SaulVazquezRedHat/camel-consumer-v2.git)

quay.io/rh-ee-savazque/demo-camel-producer:v1

quay.io/rh-ee-savazque/demo-camel-consumer:v1

1\. Creación del proyecto
-------------------------

Primero se crea un proyecto llamado:

Name:

**demo-camel**

Display name:

**Demo Camel with Quarkus and AMQ Broker**

![](demo-camel-amq-broker-quarkus/43_image.png)

![](demo-camel-amq-broker-quarkus/44_image.png)

2\. Instalación de operadores
-----------------------------

Se instala el operador de AMQ Broker

![](demo-camel-amq-broker-quarkus/45_image.png)

![](demo-camel-amq-broker-quarkus/46_image.png)

Update channel:

**7.11.x**

Installation mode:

**A specific namespace on the cluster**

Installed Namespace:

**demo-camel**

![](demo-camel-amq-broker-quarkus/47_image.png)

Esperamos a que se termine de instalar.

![](demo-camel-amq-broker-quarkus/48_image.png)

Se instala el operador de Dev Spaces desde el Operator Hub

![](demo-camel-amq-broker-quarkus/49_image.png)

![](demo-camel-amq-broker-quarkus/50_image.png)

En mi caso, dejo los valores por default. 

![](demo-camel-amq-broker-quarkus/51_image.png)

Espero a que se termine de instalar.

![](demo-camel-amq-broker-quarkus/52_image.png)

3\. Configuración de AMQ Broker
-------------------------------

Los archivos de configuración se encuentran en el repositorio:

[https://github.com/SaulVazquezRedHat/demo-camel-amq.git](https://github.com/SaulVazquezRedHat/demo-camel-amq.git)

### Creación de cluster de AMQ Broker

Abrimos el operador

![](demo-camel-amq-broker-quarkus/53_image.png)

Creamos un nuevo componente de **ActiveMQ Artemis**

![](demo-camel-amq-broker-quarkus/54_image.png)

Creamos el broker a partir del siguiente YAML `broker_activemqartemis_cr.yaml`

```text-plain
apiVersion: broker.amq.io/v1beta1
kind: ActiveMQArtemis
metadata:
  name: ex-aao
  application: ex-aao-app
  namespace: demo-camel
spec:
  deploymentPlan:
    size: 1
    image: placeholder
    requireLogin: false
    persistenceEnabled: false
    journalType: nio
    messageMigration: false
    resources:
      limits:
        cpu: 500m
        memory: 1024Mi
      requests:
        cpu: 250m
        memory: 512Mi
    storage:
      size: "4Gi"
    jolokiaAgentEnabled: false
    managementRBACEnabled: true
  console:
    expose: true
  acceptors:
    - name: amqp
      protocols: amqp
      port: 5672
      sslEnabled: false
      enabledProtocols: TLSv1,TLSv1.1,TLSv1.2
      needClientAuth: true
      wantClientAuth: true
      verifyHost: true
      sslProvider: JDK
      sniHost: localhost
      expose: true
      anycastPrefix: jms.queue.
      multicastPrefix: /topic/
  connectors:
    - name: connector0
      host: localhost
      port: 22222
      sslEnabled: false
      enabledProtocols: TLSv1,TLSv1.1,TLSv1.2
      needClientAuth: true
      wantClientAuth: true
      verifyHost: true
      sslProvider: JDK
      sniHost: localhost
      expose: true
  upgrades:
      enabled: false
      minor: false
```

![](demo-camel-amq-broker-quarkus/55_image.png)

Esperamos a que el estatus cambie a **Ready**.

![](demo-camel-amq-broker-quarkus/56_image.png)

![](demo-camel-amq-broker-quarkus/57_image.png)

Creamos un nuevo componente de **ActiveMQ Artemis Address**

![](demo-camel-amq-broker-quarkus/58_image.png)

Creamos el broker a partir del siguiente YAML `broker_activemqartemisaddress_cr.yaml`

```text-plain
apiVersion: broker.amq.io/v1beta1
kind: ActiveMQArtemisAddress
metadata:
  name: orders
  namespace: demo-camel
spec:
  addressName: orders
  queueName: orders
  routingType: anycast
```

![](demo-camel-amq-broker-quarkus/59_image.png)

Hemos creado una cola en AMQ Broker que se llama **orders**

![](demo-camel-amq-broker-quarkus/60_image.png)

4\. Configuración de Red Hat OpenShift Dev Spaces
-------------------------------------------------

Abrimos el operador de Red Hat OpenShift Dev Spaces

![](demo-camel-amq-broker-quarkus/61_image.png)

Creamos una nuevo cluster.

![](demo-camel-amq-broker-quarkus/62_image.png)

Dejo los valores por default.

![](demo-camel-amq-broker-quarkus/63_image.png)

Abro la especificación del cluster

![](demo-camel-amq-broker-quarkus/64_image.png)

Y esperamos a que aparezca el link de **Red Hat OpenShift Dev Spaces URL.** 

_**Puede tardar alrededor de 10 minutos.**_

![](demo-camel-amq-broker-quarkus/65_image.png)

![](demo-camel-amq-broker-quarkus/67_image.png)

5\. Despliegue de artefactos
----------------------------

En este momento, la vista de la topología con los pods desplegados se ve de la siguiente manera.

![](demo-camel-amq-broker-quarkus/66_image.png)

### a) Opción 1: Despliegue desde Dev Spaces

Creamos workspaces de Dev Spaces

Los creamos a partir de los siguientes repositorios:

```text-plain
https://github.com/SaulVazquezRedHat/camel-producer-v2.git
https://github.com/SaulVazquezRedHat/camel-consumer-v2.git
```

Construimos y desplegamos desde Dev Spaces:

```text-plain
oc project demo-camel
./mvnw clean package -Dquarkus.container-image.build=true
```

#### Creando el productor

Abrir URL de Red Hat OpenShift Dev Spaces

![](demo-camel-amq-broker-quarkus/68_image.png)

![](demo-camel-amq-broker-quarkus/69_image.png)

Metemos credenciales

![](demo-camel-amq-broker-quarkus/70_image.png)

![](demo-camel-amq-broker-quarkus/71_image.png)

Veremos la siguiente UI.

![](demo-camel-amq-broker-quarkus/72_image.png)

Creamos un nuevo Workspace de:

[https://github.com/SaulVazquezRedHat/camel-producer-v2.git](https://github.com/SaulVazquezRedHat/camel-producer-v2.git)

![](demo-camel-amq-broker-quarkus/73_image.png)

![](demo-camel-amq-broker-quarkus/74_image.png)

Cuando termine de crearse el workspace, va a abrirse un IDE de VS Code.

Click en **Yes, I trust the authors**.

![](demo-camel-amq-broker-quarkus/75_image.png)

Aquí podemos detenernos a explorar el código, pero para propósitos de este documento voy a continuar directamente con la construcción y despliegue del artefacto. 

Abrimos la terminal:

![](demo-camel-amq-broker-quarkus/76_image.png)

Construimos y desplegamos con:

```text-plain
oc project demo-camel
./mvnw clean package -Dquarkus.kubernetes.deploy=true
```

![](demo-camel-amq-broker-quarkus/77_image.png)

Esperamos a que termine.

![](demo-camel-amq-broker-quarkus/78_image.png)

Y si regresamos a la vista de topología, deberíamos ver los siguiente.

![](demo-camel-amq-broker-quarkus/79_image.png)

Ahora vamos a construir y desplegar el consumidor.

Abrimos la UI de Red Hat OpenShift Dev Spaces. Y creamos un nuevo Workspace de:

![](demo-camel-amq-broker-quarkus/80_image.png)

Esperamos a que se termine de generar el workspace.

![](demo-camel-amq-broker-quarkus/81_image.png)

Click en **Yes, I trust the authors**.

![](demo-camel-amq-broker-quarkus/82_image.png)

Nuevamente, voy a pasar directo a la compilación y despliegue.

Construimos y desplegamos con:

```text-plain
oc project demo-camel
./mvnw clean package -Dquarkus.kubernetes.deploy=true
```

![](demo-camel-amq-broker-quarkus/83_image.png)

Esperamos a que termine.

![](demo-camel-amq-broker-quarkus/84_image.png)

Y si regresamos a la vista de topología, deberíamos ver los siguiente.

![](demo-camel-amq-broker-quarkus/85_image.png)

### b) Opción 2: Despliegue desde registro externo (quay.io)

#### Desplegando Producer

Se va a desplegar desde u container image

![](demo-camel-amq-broker-quarkus/7_image.png)

Este es el registry desde el que se va a desplegar

```text-plain
quay.io/rh-ee-savazque/demo-camel-producer:v1
```

Image name from external registry

**quay.io/rh-ee-savazque/demo-camel-producer:v1**

Runtime icon

**quarkus**

Application

**Create application**

Application name

**producer**

Name

**demo-camel-producer**

Target port

**8080**

Create a route

![](demo-camel-amq-broker-quarkus/18_image.png)

Click en **Create**

![](demo-camel-amq-broker-quarkus/19_image.png)

![](demo-camel-amq-broker-quarkus/14_image.png)

#### Desplegando Consumer

Se va a desplegar desde u container image

![](demo-camel-amq-broker-quarkus/7_image.png)

Este es el registry desde el que se va a desplegar

```text-plain
quay.io/rh-ee-savazque/demo-camel-consumer:v1
```

Image name from external registry

**quay.io/rh-ee-savazque/demo-camel-consumer:v1**

Runtime icon

**quarkus**

Application

**Create application**

Application name

**consumer**

Name

**demo-camel-consumer**

Target port

**8080**

Create a route

![](demo-camel-amq-broker-quarkus/20_image.png)

Click en **Create**

![](demo-camel-amq-broker-quarkus/13_image.png)

![](demo-camel-amq-broker-quarkus/15_image.png)

![](demo-camel-amq-broker-quarkus/16_image.png)

Al final, deberíamos terminar con algo así en topología.

![](demo-camel-amq-broker-quarkus/86_image.png)

6\. Pruebas
-----------

### Probando dentro del cluster

Se debe tener una vista de Topology así:

![](demo-camel-amq-broker-quarkus/21_image.png)

Abrimos la UI de AMQ Broker.

Buscamos la ruta de nomobre **ex-aao-wconsj-0-svc-rte** y abrimos el link de Location

![](demo-camel-amq-broker-quarkus/25_image.png)

Credenciales:

Username:

redhat

Password:

redhat

![](demo-camel-amq-broker-quarkus/26_image.png)

Y en la sección de colas, vemos que hay un consumidor conectado.

![](demo-camel-amq-broker-quarkus/27_image.png)

Abrimos la **Terminal** del consumer.

![](demo-camel-amq-broker-quarkus/22_image.png)

![](demo-camel-amq-broker-quarkus/23_image.png)

Consumir servicio REST desde **dentro** del cluster:

```text-plain
curl -X POST -H "Content-Type: plain/text" -d 'Message from consumer' http://localhost:8080/api/hello
```

![](demo-camel-amq-broker-quarkus/24_image.png)

Abrimos los Logs del productor

![](demo-camel-amq-broker-quarkus/29_image.png)

![](demo-camel-amq-broker-quarkus/30_image.png)

### Prueba de persistencia de mensajes

Se escala a cero el pod que consume mensajes

![](demo-camel-amq-broker-quarkus/31_image.png)

![](demo-camel-amq-broker-quarkus/32_image.png)

Vemos que ya no hay consumidores conectados a AMQ Broker

![](demo-camel-amq-broker-quarkus/33_image.png)

Enviamos mensajes

![](demo-camel-amq-broker-quarkus/34_image.png)

Se tienen los 5 mensajes en el broker en la cola

![](demo-camel-amq-broker-quarkus/35_image.png)

Creamos un pod que los consuma

![](demo-camel-amq-broker-quarkus/36_image.png)

Y revisamos sus logs

![](demo-camel-amq-broker-quarkus/37_image.png)

![](demo-camel-amq-broker-quarkus/38_image.png)

### Envió de mensajes desde afuera del cluster

Se pueden enviar mensajes hacia la ruta del endpoint REST.

Recuperamos la ruta del consumidor:

![](demo-camel-amq-broker-quarkus/39_image.png)

Hacemos una consulta con **curl**

```text-plain
curl -X POST -H "Content-Type: plain/text" -d 'Mensaje desde afuera' \
https://demo-camel-consumer-demo-camel.apps.cluster-9cwzh.9cwzh.sandbox119.opentlc.com/api/hello
```

![](demo-camel-amq-broker-quarkus/41_image.png)

![](demo-camel-amq-broker-quarkus/42_image.png)

Apéndice A: Construcción de las imágenes  
------------------------------------------

A continuación se muestra el proceso de creación de las imágenes del registro de quay.io.

Se clona el repositorio:

[https://github.com/SaulVazquezRedHat/camel-producer-v2.git](https://github.com/SaulVazquezRedHat/camel-producer-v2.git)

```text-plain
git clone https://github.com/SaulVazquezRedHat/camel-producer-v2.git
```

![](demo-camel-amq-broker-quarkus/image.png)

Se construyen los artefactos:

```text-plain
cd camel-producer-v2/
./mvnw clean compile package
```

![](demo-camel-amq-broker-quarkus/1_image.png)

![](demo-camel-amq-broker-quarkus/2_image.png)

Se construye la imagen

```text-plain
podman version
podman build -f src/main/docker/Dockerfile.jvm -t quay.io/rh-ee-savazque/demo-camel-producer:v1 .
```

![](demo-camel-amq-broker-quarkus/3_image.png)

Inicio sesión para subir imagen

```text-plain
podman login quay.io
```

![](demo-camel-amq-broker-quarkus/4_image.png)

Subiendo imagen a registro.

```text-plain
podman push quay.io/rh-ee-savazque/demo-camel-producer:v1
```

![](demo-camel-amq-broker-quarkus/5_image.png)

![](demo-camel-amq-broker-quarkus/6_image.png)

Y lo mismo aplica para el repositorio

[https://github.com/SaulVazquezRedHat/camel-consumer-v2.git](https://github.com/SaulVazquezRedHat/camel-consumer-v2.git)

![](demo-camel-amq-broker-quarkus/9_image.png)

![](demo-camel-amq-broker-quarkus/10_image.png)

![](demo-camel-amq-broker-quarkus/11_image.png)

![](demo-camel-amq-broker-quarkus/12_image.png)