# Instalar y configurar VyOS como router en un entorno virtualizado con Proxmox

## Autor

- [Ixen Rodríguez Pérez - kurosaki1976](ixenrp1976@gmail.com)

## Introducción

`VyOS` es un sistema operativo de red de código abierto basado en `Debian GNU/Linux`. Proporciona una plataforma de enrutamiento gratuita que compite directamente con otras soluciones disponibles en el mercado de proveedores de red, como `Huawei` y `Cisco`, entre los más conocidos. Provee, entre otras, las funcionalidades de enrutamiento, cortafuegos, `VPN` y `VLAN`.

## Escenario

## Instalación

El proceso de instalación de `VyOS` es en extremo sencillo, basta con ejecutar el comando `install image` y aceptar (**recomendado**) o modificar las opciones que nos presenta el asistente; al concluir, reiniciar.

Una vez instalado `VyOS` dispone de dos modos de gestión, el modo operacional y el modo de configuración. En el primero es donde se ejecutan todos los comandos de verificación y monitoreo, y en el segundo, como su propio nombre indica; los comandos de configuración.

## Configuración

Acceder al modo de configuración a través del comando `configure`.

* Parámetros globales

```bash
set system host-name 'HOSTNAME'
set system name-server 'ISP_NAME_SERVER'
set system time-zone 'America/Havana'
set system ntp server 'NTP_SERVER_ADDRESS'
```

> **NOTA**: Si se utiliza versiones igual o inferior a la `1.1.8`, es recomendable ejecutar `delete system console device ttyS0`.

* Parámetros de interfaces de red

```bash
set interfaces ethernet eth0 vif 'VLAN_ID' address 'WAN_ADDRESS/CIDR'
set interfaces ethernet eth0 vif 'VLAN_ID' description 'WAN'
set interfaces ethernet eth0 vif 'VLAN_ID' address 'LAN_ADDRESS/CIDR'
set interfaces ethernet eth0 vif 'VLAN_ID' description 'LAN'
```

* Parámetros de enrutamiento

  - Para versiones <=1.1.8

  ```bash
  set system gateway-address 'ISP_WAN_ADDRESS'
  ```
  ó
  ```bash
  set protocols static route 0.0.0.0/0 next-hop 'ISP_WAN_ADDRESS'
  ```

  - Para versiones =>1.1.8

  ```bash
  set protocols static route 0.0.0.0/0 next-hop 'ISP_WAN_ADDRESS'
  ```

* Parámetros de seguridad

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
  set interface ethernet eth0 vif 'VLAN_ID' firewall local name OutSide
  ```

  - Definir un usuario distinto al por defecto

  ```bash
  set system login user USERNAME full-name 'Descripción del usuario'
  set system login user USERNAME authentication plaintext-password 'PASSWORD'
  set system login user USERNAME level 'admin'
  ```

  - Permitir acceso a través del protocolo `SSH`

  ```bash
  set service ssh listen-address 'LAN_ADDRESS'
  set service ssh port '22'
  ```

Aplicar la configuración, guardar los cambios para hacerlos permanentes en cada reinicio, y salir del modo de configuración.

```bash
commit && save
exit
```

> **NOTA**: Si se desea conocer todos los comandos de configuración establecidos, se debe ejecutar en el modo operacional, `show configuration commands`.

Mostrar los parámetros de configuración establecidos.

```bash
show configuration
```

## Conclusiones

La virtualización representa una de las tecnologías más eficientes para reducir los costos de infraestructura de informática y telecomunicaciones. Esto es porque la virtualización puede ser aplicada tanto a servidores como a redes. Además, también genera mayor eficiencia, agilidad y autonomía para la empresa cliente, con una inversión que no rebasa los presupuestos predefinidos.

## Referencias

* [Welcome to the VyOS project, a Linux-based network operating system](https://wiki.vyos.net/wiki/Main_Page)
* [VyOS User Guide](https://wiki.vyos.net/wiki/User_Guide)
* [VyOS Readthedocs](https://vyos.readthedocs.io/en/latest/)
* [VyOS KB](https://support.vyos.io/en/kb)
