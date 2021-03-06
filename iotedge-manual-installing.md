# Si no se instaló IoT Edge correctamente

En dado caso de que no se haya instalado iotedge en la máquina virtual de manera correcta, sigue estos pasos:

1. Descarga la imagen del sistema operativo:

```Bash
curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list
```
2. Copia lo que se acaba de descargar en una carpeta del sistema de Linux

```Bash
sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
```

3. Descarga los paquetes de Microsoft para que funcionen las librerias de iotedge

```Bash
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
```

4. Copia lo que se acaba de descargar en una carpeta del sistema de Linux

```Bash
sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
```

5. Actualiza las listas de paquetes de Linux

```Bash
sudo apt-get update
```

5. Instala el motor de Moby

```Bash
sudo apt-get install moby-engine
```

:book: El motor Moby es el único motor de contenedor compatible oficialmente con Azure IoT Edge. Las imágenes de contenedor de Docker CE/EE son totalmente compatibles con el entorno de ejecución de Moby.

7. Actualiza las listas de paquetes de Linux

```Bash
sudo apt-get update
```

8. Comprueba qué versiones de IoT Edge están disponibles.

```Bash
apt list -a iotedge
```
9. Instala la versión más reciente de IoT Edge

```Bash
sudo apt-get install iotedge
```

10. Reinicia el sistema de IoT Edge

```Bash
sudo systemctl restart iotedge
```