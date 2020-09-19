# Minikube con Docker desde Jenkins

En la siguiente configuración usaremos un Jenkis local (desde docker) al cual le podremos configurar dos tipos de agente

1. Agente por SSH
2. Agente Cloud Docker

**Nota:** Esta configuración no debe ser usada en un ambiente productivo ni accesible en línea. Esta pensada para un ambiente de pruebas controlado.

Este proyecto simula una arquitectura similar a la que podríamos desplegar en un ambiente productivo de jenkins donde tenemos un agente **spot EC2** para realizar construcciones y tiene que desplegar en un **cluster de K8S**.

La diferencia radica en que en este caso el cluster de k8s es un **minikube** desplegado dentro del mismo nodo "EC2" (en este caso tu propio PC).

La imagen a continuación muestra un [esquema](jenkinsAgent.drawio) de lo que queremos lograr.

![Digrama Despliegue](media/DeploymentDiagram.png)

Para este proceso necesitaras lo siguiente:

1. Llaves publica y privada ya generadas
2. Minikube instalado en el nodo a usar como agente (NodoLocal)
3. Docker instalado en el nodo a usar por medio del API

Todos los comandos fueron probados sobre Ubuntu 18.04.

## Iniciando docker como imagen con su directorio

```
docker run -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```
![Entrada a Jenkins](media/LoginAdmin.png)

### Instalando Llaves de manera local

Lo primero es conocer la IP de tu máquina.

Luego copiar el public_id al mismo host

```
ssh-copy-id -i .ssh/id_rsa.pub alejandro@192.168.20.107
```

Luego prueba el ingreso enviando la llave privada para no solicitar clave

```
ssh -i .ssh/id_rsa alejandro@192.168.20.107
```

### Adicionando Credenciales y Nodo por SSH a Jenkins

Antes de adicionar el agente es necesario instalar los plugins de SSH en Jekins para ello vamos
al menu de instalación:

![Menú instlacion Plugins](media/InstalaciónPlugins.png)

acto seguido seleccionamos las aplicaciones que quermos instalar.

![SHH Plugins](media/SSHPluginsInstalados.png)

Luego se debe adicionar la llave privada en las credenciales de Jenkins

![SSH Credencial Privada](media/AdicionandoLlavesSSH.png)

Luego de esto no vamos a la configuración de los nodos, para adicionar el nodo local.

![Configurar Nodo](media/ConfigNodes.png)

Por último Creamos el Nodo local.

![Configurando Nodo Local](media/ConfiguraciónNodoLocal.png)

## Configurando el API de Docker en Ubuntu 18.04

El archivo daemon.json del cual se encuentra un sinfin de documentación ya no suele tener el mismo efecto en esta versión de ubuntu

Ahora para configurar el **Socket del API** en el puerot 2375 se debe modificar el siguiente archivo: /etc/systemd/system/sockets.target.wants/docker.socket adicionando otro **ListenStream** con el puerto deseado.

```
[Unit]
Description=Docker Socket for the API
PartOf=docker.service

[Socket]
ListenStream=/var/run/docker.sock
ListenStream=2375
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.target
```

Una vez configurado se debe Habiliar el puerto y reiniciar el servicio de docker

```
systemctl enable docker.socket
systemctl restart docker.service
```

Para probar que API esta habilitado use de forma local:

```bash
curl -X GET http://localhost:2375/media/json
```
El anterior comando te arroja un listado de las imágenes disponibles en la máquina.

## Corriendo un Proyecto que Inicia el Minikube

El NodoLocal **tiene por defecto instalado el minikube** sino sabes instalarlo [este proceso de Instalación](https://medium.com/@alejandroleon09/minikube-con-hyper-v-en-windows10-2f3fae956c3b) en Windows 10 te puede servir. También puedes intentar crear tu propio cluster [aquí](https://tutmosisii.wordpress.com/2018/10/10/kubernetes-cluster-con-vagrant/).

### Construcción con Lineas de Producción.

Los **Pipelines** como son conocidos en inglés son scripts en groovy que permiten automaizar por pasos los procesos de construcción del sofotware.
En este caso el [Jenkinsfile](Jenkinsfile) contiene los pasos necesarios para este ejemplo.

1. Crear un proyecto de tipo pipeline.

   En este Tarea configurar el pipeline desde un Un SCM (Software Configuration Managemenent) y seleccionar a git como administrador.

## Plugins a Usar en Visual Studio Code

Se aconseja trabajar en parejas para complementar el proyecto y usar los siguientes plugins

[![Drawio Integration](media/drawio_integration.png)](https://marketplace.visualstudio.com/items?itemName=hediet.vscode-drawio)

ext install hediet.vscode-drawio

[![Drawio Integration](media/Live_Share.png)](https://marketplace.visualstudio.com/items?itemName=MS-vsliveshare.vsliveshare)

ext install MS-vsliveshare.vsliveshare

[![Jenkins Linter pluging](media/JenkinsLinter.png)](https://marketplace.visualstudio.com/items?itemName=janjoerke.jenkins-pipeline-linter-connector)

Para **Validar** Jenkinsfiles

ext install janjoerke.jenkins-pipeline-linter-connector

Este plugin require unas configuraciones para alcazar el API de Jenkins, ademas del usuario y el passwor se debe definir las siguiente URLs:

jenkins.pipeline.linter.covnnector.url: http://localhost:8080>/pipeline-model-converter/validate
jenkins.pipeline.linter.connector.crumbUrl : http://localhost:8080/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,%22:%22,//crumb)

## Referencias

[Configuración de puerto API](https://riptutorial.com/es/docker/example/15951/habilitar-el-acceso-remoto-a-la-api-de-docker-en-linux-ejecutando-systemd)

[Instalacion Agente SSH](https://plugins.jenkins.io/ssh-agent/)

[Construcción con Pipelines](https://www.jenkins.io/solutions/pipeline/)

[KubeInvaders](https://kubernetes.io/blog/2020/01/22/kubeinvaders-gamified-chaos-engineering-tool-for-kubernetes/)