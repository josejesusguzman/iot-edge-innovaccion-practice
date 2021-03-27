# Práctica de Azure IoT Hub con App Service

Aquí podrás encontrar los comandos del Cloud Shell usados y explicados.

**Remplaza las palabras entre <> por la información correspondiente**

## Herramientas usadas
* Azure IoT Hub
* Azure App Service
* Simulated Temperature Sensor (Este lo buscas desde el Marketplace con este nombre en Inglés)
* Azure Cloud Shell con Bash
* [Repositorio web-apps-node-iot-hub-data-visualization](https://github.com/Azure-Samples/web-apps-node-iot-hub-data-visualization)
* [Git](https://git-scm.com/downloads)
* [Simulador Raspberry Pi](https://azure-samples.github.io/raspberry-pi-web-simulator/#getstarted)
* [Node.js](https://nodejs.org/en/download/)
  * En Windows debes [poner la variable de entorno](https://bertofern.wordpress.com/2019/01/08/solucion-node-js-npm-no-reconocido-como-comando-interno-o-externo/)


## Recursos adicionales
* [Manifiesto del diseño de IoT](https://www.iotmanifesto.com/)

**Indice**
<!-- TOC -->

- [Práctica de Azure IoT Hub con App Service](#práctica-de-azure-iot-hub-con-app-service)
  - [Herramientas usadas](#herramientas-usadas)
  - [Recursos adicionales](#recursos-adicionales)
  - [Creación de IoT Hub con Azure CLI](#creación-de-iot-hub-con-azure-cli)
  - [Adjuntamos un sensor de temperatura simulado](#adjuntamos-un-sensor-de-temperatura-simulado)
  - [Agregamos la Raspberry Pi Simulada](#agregamos-la-raspberry-pi-simulada)
    - [Visualización de los datos de RaspBerry Pi en web](#visualización-de-los-datos-de-raspberry-pi-en-web)
  - [Subir mi web a Azure App Services](#subir-mi-web-a-azure-app-services)

<!-- /TOC -->

## Creación de IoT Hub con Azure CLI

1. Abre portal.azure.com y abre el Cloud Shell. Si no tienes una cuenta de almacenamiento en tu suscripción creala desde la ventana que te aparece

2. Una vez que cargue vamos a generar tres variables para usarlas durante nuestra implementación

```Console
RESOURCE_GROUP=test-arqs \
LOCATION=southcentralus \
IOT_HUB_NAME=iot-res-arqs \
IOT_EDGE_DEVICE_NAME=edge-device-arqs
```

**Nota**: Se pueden hacer variables temporales dentro del CLI de Azure para asegurarte tener el mismo nombre de grupo de recursos, localización y nombres de recursos durante toda la implementación.

3. Creamos un recurso de [IoT Hub](https://azure.microsoft.com/es-mx/services/iot-hub/)

```Console
az iot hub create --resource-group $RESOURCE_GROUP --name $IOT_HUB_NAME --sku F1 --partition-count 2 --location $LOCATION
```

4. Creas un dispositivo de IoT Edge asociado al IoT Hub que acabas de crear

```Console
az iot hub device-identity create --device-id $IOT_EDGE_DEVICE_NAME --edge-enabled --hub-name $IOT_HUB_NAME 
```

5. Obtenemos nuestro ConnectionString que nos servirá para conectarnos después 

```Console
az iot hub device-identity connection-string show --device-id $IOT_EDGE_DEVICE_NAME --hub-name $IOT_HUB_NAME
```
- Obtendrías algo parecido a esto. Guardalo:

```
HostName=iot-res-arqs.azure-devices.net;DeviceId=myEdgeDevice;SharedAccessKey=yNVfE31mVrTFfoDnsTa04XfHq4HYEvS1s9SyLVvsrbY=
```

**Importante**: Guardalo absolutamente todo, hasta la palabra `HostName`.

6. Implementamos la configuración necesaria y creamos un grupo de implementacón asociado a nuestro IoT Hub para que funcione. En este caso se usará una plantilla de ARM de Microsoft para la configuración. https://aka.ms/iotedge-vm-deploy

```
az deployment group create \
--resource-group $IOT_HUB_NAME \
--template-uri "https://aka.ms/iotedge-vm-deploy" \
--parameters dnsLabelPrefix='<NOMBRE_MAQUINA_VIRTUAL>' \
--parameters adminUsername='azureUser' \
--parameters deviceConnectionString=$(az iot hub device-identity connection-string show --device-id $IOT_EDGE_DEVICE_NAME --hub-name iot-res-arqs -o tsv) \
--parameters authenticationType='password' \
--parameters adminPasswordOrKey="<PASSWORD>."
```

**Nota**: Toda la configuración se hace a traves de la plantilla ARM y pasamos todos los datos por medio de parametros. Para saber más de ellas consulta el [tutorial de creación de ARM templates](https://github.com/jose1824/acordeon-az900-innovaccion/blob/main/res/plantilla_arm.md)

- Obtendrás algo parecibo a esto. Guardalo para acceder a tu VM:

![Captura SSH result](/res/outputs-public-ssh.png)

7. Colocamos el acceso SSH obtenido para entrar a la máquina virtual

```Console
ssh <TU USERNAME>@{TU DNS}
```

8. Comprobamos el status de nuestra implementación de iotedge

```Console
sudo systemctl status iotedge
```
- Si te falló la implementación de iotedge espera unos minutos y si continua fallando ve este [tutorial para instalarlo de forma manual.](/iotedge-manual-installing.md)

- Si quieres obtener los registros de creación del dispositivo lo puedes hacer copn el siguiente comando

```Console
journalctl -u iotedge
```

- Si quieres ver que realmente se creo el dispositivo ejecuta este comando. Se encarga de listar todos los dispositivos de IoT asociados a tu Hub:
```Console
sudo iotedge list
```

## Adjuntamos un sensor de temperatura simulado

1. Ve a portal.azure.com y ve a tu recurso IoT Hub que acabas de crear.

2. en el menú de la izquierda del recurso busca IoT Edge.
3. Selecciona el dispostivo que tienes ahí
4. Da clic en **Establecer módulos**

![Captura Azure IoT](/res/select-set-modules.png)

5. Agrega un dispositivo de Marketplace

![Captura Azure IoT](/res/add-marketplace-module.png)

6. Busca **Simulated Temperature Sensor** y agregalo
7. Ve a la sección de rutas y quita la primera por Default

![Captura Azure IoT](/res/view-temperature-sensor-next-routes.png)

![Captura Azure IoT](/res/delete-route-next-review-create.png)

8. Te debe aparecer **SimulatedTemperatureSensor** en la lista de módulos de IoT Edge.
9. Para verificar que esta funcionando nuestro sensor de temperatura escribe lo siguiente en el Azure Cloud Shell. Esto listará todos los dispositivos asociados al IoT Hub actual

```Console
sudo iotedge list
```
**Nota**: si cerraste la ventana de Cloud Shell tendrás que entrar de nuevo a la maquina virtual con SSH.

10. Escribe el siguiente comando para que veas que el sensor esta funcionando y obteniendo datos.

```Console
sudo iotedge logs SimulatedTemperatureSensor -f
```
Te debe aparecer algo así

![Captura Azure IoT](/res/iotedge-logs.png)


## Agregamos la Raspberry Pi Simulada

**Nota**: No necesitas hacer el paso anterior [Adjuntamos un sensor de temperatura simulado](#adjuntamos-un-sensor-de-temperatura-simulado) para agregar la Raspberry Pi, puedes hacerlo directamente después de crear tu servicio IoT Hub.

**Nota**: Ve este [tutorial para conectar tu Raspberry Pi fisica ](https://docs.microsoft.com/es-mx/azure/iot-hub/iot-hub-raspberry-pi-kit-node-get-started)

1. Desde Azure Clod Shell o Azure CLI obtenemos un nuevo tipo de connection string que es el que usaremos de aquí en adelante del tutorial, guardalo. Se hace con este comando:

```Console
az iot hub show-connection-string --hub-name $IOT_HUB_NAME --policy-name service
```
2. Vamos al [simulador de Azure de la Raspberry Pi](https://azure-samples.github.io/raspberry-pi-web-simulator/#getstarted)

![Captura Raspberry](/res/iot-lab2.jpg)

3. Creamos un nuevo dispositivo de IoT Edge como en pasos anteriores.

4. Una vez creado, obtenemos el Primary Connection String

![Captura Azure IoT](/res/device-details-vs2019.png)

5. Lo colocamos en esta parte del código de la Raspberry Pi
![Captura Raspberry](/res/1-connectionstring.png)

6. Creamos un grupo de consumo de Azure para que podamos obtener los datos de la Raspberry Pi. Guarda el nombre del grupo de consumo:

```Console
az iot hub consumer-group create --hub-name iot-res-arqs --name ConsumerGroupArqs
```

### Visualización de los datos de RaspBerry Pi en web

7. Abre cmd o terminal, colocate en la carpeta que más desees, clona el siguiente proyecto de GitHub y dirigete a la carpeta contenedora:

```cmd
git clone https://github.com/Azure-Samples/web-apps-node-iot-hub-data-visualization.git
cd web-apps-node-iot-hub-data-visualization
```

8. Coloca las variables de entorno en la misma CMD o terminal que tienes abierta con los siguientes comandos:

```cmd
set IotHubConnectionString=<TU_CONNECTION_STRING>
set EventHubConsumerGroup=<TU_NOMBRE_CONSUMER_GROUP>
```
9. Ejecuta los siguientes comandos de `npm` para instalar y comenzar a ejecutar la solución.

```cmd
npm install
npm start
```

10. Ve al navegador y accesa a: http://localhost:3000

Debe aparecerte algo así:

![Captura Solución completa](/res/iot-lab1.jpg)

Listo, has obtenido los datos de la Raspberry Pi en una página web

## Subir mi web a Azure App Services

1. Nos dirijimos a portal.azure.com y abrimos el Azure Cloud Shell

2. Ejecuta el siguiente comando para crear un plan de App Service:

```Azure CLI
az appservice plan create --name <NOMBRE_SERVICE_PLAN> --resource-group <NOMBRE_GRUPO_RECURSOS> --sku FREE
```

**Nota**: Aquí ya no esoty usando las variables del inicio pero puedes sentirte libre de usarlas.

3. Creamos una webapp y la asociamos a nuestro plan de App Service. Esto creará un recurso de App Service.

```Azure CLI
az webapp create -n <NOMBRE_WEBAPP> -g <NOMBRE_GRUPO_RECURSOS> -p <NOMBRE_SERVICE_PLAN> --runtime "node|10.6" --deployment-local-git
```

4. Una vez creado nuestro recurso, colocamos las variables de entorno como lo hicimos en nuestro entorno local

```Azure CLI
az webapp config appsettings set -n <NOMBRE_WEBAPP> -g <NOMBRE_GRUPO_RECURSOS> --settings EventHubConsumerGroup=<TU_NOMBRE_CONSUMER_GROUP> IotHubConnectionString="<TU_CONNECTION_STRING>"
```

5. Abrimos los Web Sockets de nuestro App Service y habilitamos solo trafico por HTTPS:

```Azure CLI
az webapp config set -n <NOMBRE_WEBAPP> -g <NOMBRE_GRUPO_RECURSOS> --web-sockets-enabled true

az webapp update -n <NOMBRE_WEBAPP> -g <NOMBRE_GRUPO_RECURSOS> --https-only true
```

- También puedes esta configuración manualmente

![Captura Solución completa](/res/iot-lab4.jpg)

6. si no has hecho antes una implementación de Git con App Service y/o no has establecido antes credenciales de implementación, hazlo ahora con este comando:

```Azure CLI
az webapp deployment user set --user-name <USERNAME>
```

- Después de la ejecución de este comando te pedirá un *password* yq ue lo confirmes. **Recuerda y guarda bien estos dos datos**

**Recordatorio**: En terminales cuando escribes contraseñas no aparece nada por seguridad, es decir, parece que no estas ecribiendo, pero si lo estas haciendo.

8. Obtienes y guardas la URL de Git con el siguiente comando (también puedes obtenerla viendo el detalle del recurso):

```Azure CLI
az webapp deployment source config-local-git -n <NOMBRE_WEBAPP> -g <NOMBRE_GRUPO_RECURSOS>
```

9. Añades el destino remoto a tu repositorio local en tu computadora usando la URL obtenida en el paso anterior:

```cmd
git remote add webapp <URL_GIT>
```

10. Subes tu solución a App Service

```cmd
git push webapp master
```

- Debes obtener una respuesta similar a esta:

```cmd
remote:
remote: Finished successfully.
remote: Running post deployment command(s)...
remote: Deployment successful.
To https://contoso-web-app-3.scm.azurewebsites.net/contoso-web-app-3.git
6b132dd..7cbc994  main -> main
```

11. Comprueba el status de la webapp con el siguiente comando:

```Azure CLI
az webapp show -n <NOMBRE_WEBAPP> -g <NOMBRE_GRUPO_RECURSOS> --query state
```

12. Accede al link de la webapp desde el detalle de tu recurso. Debes tener un resultado igual al que salió en tu computadora:

![Captura Solución completa](/res/iot-lab3.jpg)

(Sin el inspector de elementos claro esta)

13. Recuerda eliminar tus recursos de Azure o todo tu grupo de recursos para cuidar tu saldo.

