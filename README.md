# Práctica de Azure IoT Hub
## Práctica del módulo 193 del Training de Innovacción Virtual

Aquí podrás encontrar los comandos del Cloud Shell usados y explicados.
**Remplaza las palabras en mayusculas por la información correspondiente**

### Herramientas usadas
* Azure IoT Hub
* Simulated Temperature Sensor (Este lo buscas desde el Marketplace con este nombre en Inglés)
* Azure Cloud Shell con Bash
* Azure Virtual Machine

Con este comando instalamos la extensión de IoT para darle las funcionalidades que necesitamos a nuestro Cloud Shell

```
az extension add --name azure-cli-iot-ext
```

Creación de un grupo de recursos. *Se puede cambiar la localización*

```
az group create --name NOMBRE_GRUPO_DE_RECURSOS --location eastus2
```
Obtenemos la versión del sistema operativo Ubuntu adaptada para dispositivos IoT
```
az vm image accept-terms --urn microsoft_iot_edge:iot_edge_vm_ubuntu:ubuntu_1604_edgeruntimeonly:latest
```
Creación de la Maquina Virtual con el sistema operativo que acabamos de obtener. **Recuerda guardar la IP**
```
az vm create --resource-group NOMBRE_GRUPO_DE_RECURSOS --name NOMBRE_MAQUINA_VIRTUAL --image microsoft_iot_edge:iot_edge_vm_ubuntu:ubuntu_1604_edgeruntimeonly:latest --admin-username azureuser --generate-ssh-key
```
Creamos un servicio de IoT Hub
```
az iot hub create --resource-group NOMBRE_GRUPO_DE_RECURSOS --name NOMBRE_IOT_HUB --sku S1
```
Creamos un dispositivo de IoT Edge que no servirá para emular el funcionamiento de uno real
```
az iot hub device-identity create --hub-name NOMBRE_IOT_HUB --device-id NOMBRE_DISPOSITIVO_EDGE --edge-enabled
```
Obtenemos el String de Conexión del dispositivo. **Recuerda guardarlo**
```
az iot hub device-identity show-connection-string --device-id NOMBRE_DISPOSITIVO_EDGE --hub-name NOMBRE_IOT_HUB
```
Instalación del dispositivo IoT Edge en la maquina virtual
```
az vm run-command invoke -g NOMBRE_GRUPO_DE_RECURSOS -n NOMBRE_MAQUINA_VIRTUAL --command-id RunShellScript --script "/etc/iotedge/configedge.sh 'TU_CONECTION_STRING'"
```
Accedemos a la maquina virtual por SSH
```
ssh azureuser@IP_PUBLICA_MAQUINA_VIRTUAL
```
Comprobamos el status de nuestro dispositivo IoT Edge. **Debe decirte Active**
```
sudo systemctl status iotedge
```

```
journalctl -u iotedge
```
Lista los dispositivos de Iot Edge que tienes en la maquina virtual
```
sudo iotedge list
```

**Aquí primero tienes que ir a azure a conectar en Sensor de temperatura simulado al recurso IoY Hub que se creó**

Obtenemos de manera periodica las lecturas del sensor de temperatura
```
sudo iotedge logs SimulatedTemperatureSensor -f
```

### Links
* [Clase en vivo de IoT](https://web.microsoftstream.com/video/120b1cdc-c267-4ea4-bac6-528a3f707359)
* [Tutorial IoT Edge](https://www.youtube.com/watch?v=lN66jCsFjGs&ab_channel=Innovacci%C3%B3nvirtual)
* [Live Internet de las cosas y tecnología vestible](https://teams.microsoft.com/l/meetup-join/19%3ameeting_NjdlNjZjMmMtZTQ5NC00ZGNkLWI3NjEtZmIzNjQzYjQxNTJl%40thread.v2/0?context=%7b%22Tid%22%3a%224ae54b05-b77e-4224-aef1-8661422e0816%22%2c%22Oid%22%3a%2267a900b5-a64a-4c11-a5b4-e37828d233ed%22%2c%22IsBroadcastMeeting%22%3atrue%7d)
* [Manifiesto del diseño de IoT](https://www.iotmanifesto.com/)
