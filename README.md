# Instalar y configurar VyOS como router en un entorno virtualizado con Proxmox

## Autor

- [Ixen Rodríguez Pérez - kurosaki1976](ixenrp1976@gmail.com)

## Introducción

`VyOS` es un sistema operativo de red de código abierto basado en `Debian GNU/Linux`. Proporciona una plataforma de enrutamiento gratuita que compite directamente con otras soluciones disponibles en el mercado de proveedores de red, como `Huawei` y `Cisco`, entre los más conocidos. Provee, entre otras, las funcionalidades de enrutamiento, cortafuegos, `VPN` y `VLAN`. Para su gestión, `VyOS` provee una interfaz de línea de comandos (`CLI` por sus siglas en Inglés), al más puro estilo tradicional de configuración de `routers`.

## Escenario

En Cuba la adopción de las tecnologías de virtualización van ganando cada vez más adeptos; al mismo tiempo, va siendo común en las redes cubanas la existencia de conexión por fibra óptica -jaque mate al cobre y las tecnologías de conexión `DSL`-; lo que presupone el despliegue de electrónica avanzada de red -`routers` y `switches L2/L3`-, las cuales en no pocas ocasiones, están subutilizadas.

A los efectos de esta guía, se utilizará un entorno de virtualización basado en `Proxmox`, donde se creará una máquina virtual ejecutando un `VyOS router` con los siguientes requisitos de `hardware` y `software`.

1. Hardware
    - **CPU**: 1 core
    - **RAM**: 512 MiB
    - **HDD**: 2 GiB
    - **Arquitectura**: i586 ó amd64
    - **Adaptador de red**: 2 (solo 1, de utilizarse `VLAN` bajo protocolo de etiquetado `IEEE 802.1Q`)

2. Software
    - Imagen `ISO` de `VyOS` para la arquitectura seleccionada, disponibles en [VyOS Downloads](https://downloads.vyos.io/).

> **NOTA**: Si se despliega `VLAN` bajo protocolo de etiquetado `IEEE 802.1Q`, debe disponerse de un `switch L2`, donde se crearán las mismas `VLANs` que utilizará el `VyOS router`.
    
## Instalación

El proceso de instalación de `VyOS` es en extremo sencillo, basta con ejecutar el comando `install image` y aceptar (**recomendado**) o modificar las opciones que nos presenta el asistente; al concluir, reiniciar.

Una vez instalado, `VyOS` dispone de dos modos de gestión; el modo operacional y el modo de configuración. En el primero es donde se ejecutan todos los comandos de verificación y monitoreo, y en el segundo, como su propio nombre indica; los comandos de configuración.

## Configuración

Acceder al modo de configuración a través del comando `configure`.

> **NOTA**: Si se utiliza versiones igual o inferior a la `1.1.8`, es recomendable ejecutar `delete system console device ttyS0`.

1. Parámetros globales

    ```bash
    set system host-name 'router'
    set system name-server 'ns1.etecsa.cu'
    set system name-server 'ns2.etecsa.cu'
    set system time-zone 'America/Havana'
    ```

    > **NOTA**: Si su `ISP` cuenta con un servidor de hora, ejecutar `set system ntp server 'IP_O_FQDN_SERVIDOR_NTP'`.
    
    - Utilizar el `router` como servidor `DNS` para la subred `LAN`
    
    ```bash
    set service dns forwarding name-server 'ns1.etecsa.cu'
    set service dns forwarding name-server 'ns2.etecsa.cu'
    set service dns forwarding listen-on 200.55.143.153
    set service dns forwarding allow-from 200.55.143.152/29
    ```

2. Parámetros de los adaptadores de red

    - Sin utilizar protocolo de etiquetado `IEEE 802.1Q`

    ```bash
    set interfaces ethernet eth0 address '192.168.30.2/30'
    set interfaces ethernet eth0 description 'WAN'
    set interfaces ethernet eth1 address '200.55.143.153/29'
    set interfaces ethernet eth1 description 'LAN'
    ```

    - Utilizando protocolo de etiquetado `IEEE 802.1Q`

    ```bash
    set interfaces ethernet eth0 vif '100' address '192.168.30.2/30'
    set interfaces ethernet eth0 vif '100' description 'WAN'
    set interfaces ethernet eth0 vif '101' address '200.55.143.153/29'
    set interfaces ethernet eth0 vif '101' description 'LAN'
    ```

3. Parámetros de enrutamiento

    - Para versiones <=1.1.8

    ```bash
    set system gateway-address '192.168.30.1'
    ```

    > **NOTA**: También puede usarse `set protocols static route 0.0.0.0/0 next-hop '192.168.30.1'`.

    - Para versiones >1.1.8

    ```bash
    set protocols static route 0.0.0.0/0 next-hop '192.168.30.1'
    ```

4. Parámetros de seguridad

    - Reglas de cortafuegos

    ```bash
    set firewall all-ping enable
    set firewall broadcast-ping disable
    set firewall config-trap disable
    set firewall name OutSide
    set firewall name OutSide default-action accept
    set firewall name OutSide rule 1 action drop
    set firewall name OutSide rule 1 destination port 'ssh'
    set firewall name OutSide rule 1 protocol 'tcp'
    set firewall name OutSide rule 2 action drop
    set firewall name OutSide rule 2 destination port 'telnet'
    set firewall name OutSide rule 2 protocol 'tcp'
    ```

    - Sin utilizar protocolo de etiquetado `IEEE 802.1Q`

    ```bash
    set interfaces ethernet eth0 firewall local name OutSide
    ```

    - Utilizando protocolo de etiquetado `IEEE 802.1Q`

    ```bash
    set interfaces ethernet eth0 vif '100' firewall local name OutSide
    ```

    > **NOTA**: La aplicación de la política de cortafuegos `OutSide` , evita que el `router` sea gestionado a través de la interfaz `WAN`.

    - Definir un usuario distinto al `vyos` por defecto

    ```bash
    set system login user 'username' full-name 'Descripción del usuario'
    set system login user 'username' authentication plaintext-password 'P@s$w0rd.2019'
    set system login user 'username' level 'admin'
    ```

    - Permitir acceso a través del protocolo `SSH`

    ```bash
    set service ssh listen-address '200.55.143.153'
    set service ssh port '22'
    ```

5. Habilitar monitoreo de tráfico de red `SNMPv2 (Simple Network Management Protocol)`

    ```bash
    set service snmp listen-address '200.55.143.153'
    set service snmp community public authorization ro
    set service snmp community public network 200.55.143.152/29
    set service snmp location "CU, Nueva Gerona"
    set service snmp contact "postmaster@dominio.cu"
    ```

    > **NOTA**: Para habilitar `SNMPv3`, revisar las [referencias](#referencias).

    Aplicar la configuración, guardar los cambios -para hacerlos permanentes en cada reinicio-, y salir del modo de configuración.

    ```bash
    commit && save
    exit
    ```

## Verificación

1. Mostrar los parámetros de configuración establecidos

    ```bash
    show configuration
    ```

2. Mostrar los parámetros de los adaptadores de red

    ```bash
    show interfaces
    ```

3. Mostrar el tráfico de red

    ```bash
    monitor interfaces
    ```

> **NOTA**: Para listar todos los comandos de configuración ejecutados, utilizar, `show configuration commands`.

## Actualización

1. Obtener versión en ejecución

    ```bash
    show system image

    The system currently has the following image(s) installed:

       1: 1.2-rolling-201911061649 (default boot) (running image)
    ```

2. Añadir nueva versión

    ```bash
    add system image https://downloads.vyos.io/rolling/current/amd64/vyos-rolling-latest.iso
    
    Checking MD5 checksums of files on the ISO image...OK.
    Done!
    What would you like to name this image? [1.3-rolling-202001160217]:
    OK.  This image will be named: 1.3-rolling-202001160217
    Installing "1.3-rolling-202001160217" image.
    Copying new release files...
    Would you like to save the current configuration
    directory and config file? (Yes/No) [Yes]:
    Copying current configuration...
    Would you like to save the SSH host keys from your
    current configuration? (Yes/No) [Yes]:
    Copying SSH keys...
    Running post-install script...
    Setting up grub configuration...
    Done.
    ```

3. Reiniciar el sistema

    Contestar afirmativamente a las preguntas que se ofrecen durante el proceso de añadir versión y al concluir, reiniciar el sistema con el comando `reboot`.

> **NOTA**: Se pude volver a verificar las versiones existentes con el comando `show image system`. También es posible añadir la nueva versión desde una imagen `ISO` previamente descargada y subida al sistema.

## Conclusiones

La virtualización representa una de las tecnologías más eficientes para reducir los costos de las infraestructuras informáticas y de telecomunicaciones; jugando un papel esencial en Cuba, atendiendo a las limitaciones que enfrenta el país para la adquisición de equipamiento de última generación.

La virtualización puede ser aplicada tanto a servidores como a redes. Además, genera mayor eficiencia, agilidad y autonomía para la empresa cliente de un provedor de servicios de conectividad; con una mínima inversión que no rebase los presupuestos planificados.

## Referencias

* [Welcome to the VyOS project, a Linux-based network operating system](https://wiki.vyos.net/wiki/Main_Page)
* [VyOS User Guide](https://wiki.vyos.net/wiki/User_Guide)
* [VyOS Readthedocs](https://vyos.readthedocs.io/en/latest/)
* [VyOS KB](https://support.vyos.io/en/kb)
* [SNMPv3](https://wiki.vyos.net/wiki/SNMPv3)
* [DNS forwarding](https://wiki.vyos.net/wiki/DNS_forwarding)
