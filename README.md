# Servidor-DNS-Esclavo-en-Debian-12

## Creación y Configuración de un Servidor DNS Esclavo en Debian 12 (Sin Interfaz Gráfica)
Este documento explica los pasos para crear y configurar un servidor DNS esclavo utilizando BIND9 en Debian 12, sin interfaz gráfica. 
## Requisitos Previos

- Tener acceso a una máquina virtual (o servidor) con Debian o un sistema operativo basado en Linux.
- Permisos de superusuario (`sudo`) para realizar cambios en los archivos del sistema.
- Conexión a internet para descargar los paquetes de BIND9.

## 1. Instalación de BIND9

Para comenzar, necesitas instalar BIND9, que es el software utilizado para gestionar el servicio de DNS. Ejecuta los siguientes comandos en tu terminal:

```bash
sudo apt update
sudo apt install bind9 bind9utils 
```
![DNS](/img/DNS1.png)

## Paso 1: Configuración del Servidor DNS Esclavo
* El servidor esclavo necesita tener acceso a las zonas que serán transferidas desde el servidor maestro. A continuación, configuraremos el archivo de zonas en el servidor esclavo.

```bash
/etc/bind/named.conf.local
```

* Editar el Archivo de Configuración de Zonas en el Servidor Esclavo
    Abre el archivo `/etc/bind/named.conf.local` en el servidor esclavo para definir la zona esclava. En este ejemplo, configuraremos la zona para el dominio DNSErrabab.com, y asignaremos al servidor maestro la **IP 172.26.2.3**.

    ```bash
    sudo nano /etc/bind/named.conf.local
    ```
* Añadir la Configuración de la Zona Esclava
    Dentro del archivo, añade la siguiente configuración para la zona esclava:

    ```bash
    zone "DNSErrabab.com" {
        type slave;
        masters { 172.26.2.3; };  # IP del servidor maestro
        file "/var/cache/bind/db.DNSErrabab.com";  # Archivo de zona esclava
    };
    ```
    ![DNS](/img/DNS2.png)

    Esta configuración le indica al servidor esclavo que recibirá la información de la zona DNSErrabab.com desde el servidor maestro con la IP 172.26.2.3.
* Reiniciar el Servicio de BIND en el Servidor Esclavo
    Para aplicar los cambios, reinicia el servicio de BIND en el servidor esclavo:

    `sudo systemctl restart bind9`

## Paso 2: Configuración en el Servidor Maestro
* Para permitir que el servidor esclavo reciba las transferencias de zona desde el servidor maestro, es necesario ajustar la configuración en el servidor maestro.

* Editar el Archivo de Configuración de Zonas en el Servidor Maestro
    En el servidor maestro, abre el archivo `/etc/bind/named.conf.local `para configurar la zona maestra:
    ```bash
    sudo nano /etc/bind/named.conf.local
    ```
* Añadir la Configuración de la Zona Maestra
    Añade la siguiente configuración, que define la zona maestra y permite la transferencia de zona hacia el servidor esclavo. En este ejemplo, la IP del servidor esclavo es 172.26.2.6:

    ```bash
    zone "DNSErrabab.com" {
    type master;
    file "/etc/bind/db.DNSErrabab.com";
    allow-transfer { 172.26.2.6; };  # IP del servidor esclavo
    };
    ```
    ![DNS](/img/DNS3.png)

    Esta configuración permite que el **servidor esclavo 172.26.2.6** obtenga las transferencias de zona desde el servidor *maestro* para el **dominio DNSErrabab.com**.
* Reiniciar el Servicio de BIND en el Servidor Maestro
    Para que los cambios surtan efecto, reinicia el servicio de BIND en el servidor maestro:

    `sudo systemctl restart bind9`

    ![DNS](/img/DNS4.png)

## Paso 3: Verificación y Comprobaciones
* Verificación de la Transferencia de Zona
    Una vez que ambos servidores estén configurados, verifica que el servidor esclavo esté recibiendo correctamente las transferencias de zona desde el maestro. Puedes comprobar esto revisando los archivos de zona en el servidor esclavo:

    ```bash
    ls /var/cache/bind/
    ```
    Deberías ver el archivo **db.DNSErrabab.com**, que se genera automáticamente cuando el servidor esclavo recibe la transferencia de zona desde el maestro.

* Comprobación de la Resolución de Nombres
    En el servidor esclavo, puedes utilizar la herramienta dig para comprobar que la resolución de nombres DNS funciona correctamente:

    ```bash
    dig @172.26.2.6 DNSErrabab.com
    ```

    ![DNS](/img/DNS5.png)

## Conclusión
Has configurado un servidor DNS esclavo en Debian 12 utilizando BIND9. El servidor esclavo ahora puede recibir transferencias de zona desde el servidor maestro, permitiendo redundancia y alta disponibilidad en el sistema DNS. Asegúrate de probar y verificar que ambos servidores funcionan correctamente.







