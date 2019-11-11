# Instalar y configurar VyOS como router en un entorno virtualizado con Proxmox

## Autor

- [Ixen Rodríguez Pérez - kurosaki1976](ixenrp1976@gmail.com)

## Introducción

`VyOS` es un sistema operativo de red de código abierto basado en `Debian GNU/Linux`. `VyOS` proporciona una plataforma de enrutamiento gratuita que compite directamente con otras soluciones disponibles en el mercado de proveedores de red, como Huawei y Cisco, entre los más conocidos. Provee, entre otras, las funcionalidades de enrutamiento, cortafuegos, `VPN` y `VLAN`. 

## Instalación

```bash
install image
```

## Configuración

Acceder al modo de configuración a través del comando `configure`.

* Parámetros globales

```bash
set system host-name 'HOSTNAME'
set system name-server 'ISP_NAME_SERVER'
set system time-zone 'America/Havana'
set system ntp server 'NTP_SEVER_ADDRESS'
```

> **NOTA**: Si se está utilizando la versión 1.1.8 es recomendable ejecutar `delete system console device ttyS0`.

* Parámetros de interfaces de red

```bash
set interfaces ethernet eth0 vif 'VLAN_ID' address 'WAN_ADDRESS/CIDR'
set interfaces ethernet eth0 vif 'VLAN_ID' description 'WAN'
set interfaces ethernet eth0 vif 'VLAN_ID' address 'LAN_ADDRESS/CIDR'
set interfaces ethernet eth0 vif 'VLAN_ID' description 'LAN'
```

* Parámetros de enrutamiento

```bash
set system gateway-address 'ISP_WAN_ADDRESS'
set protocols static route 0.0.0.0/0 next-hop 'ISP_WAN_ADDRESS'
```

* Parámetros globales

```bash
set service ssh listen-address 'LAN_ADDRESS'
set service ssh port '22'
```

* Parámetros de cortafuegos

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

* Parámetros de seguridad

```bash
set system login user USERNAME full-name 'User Full Name'
set system login user USERNAME authentication plaintext-password 'PASSWORD'
set system login user USERNAME level 'admin'
```

Aplicar la configuración, guardar los cambios para hacerlos permanentes en cada reinicio, y salir del modo de configuración.

```bash
commit && save
exit
```

> **NOTA**: Si se desea conocer todos los parámetros de configuración establecidos, se debe ejecutar en el modo de operación el comando `show configuration commands`.

## Referencias

* [User Guide](https://wiki.vyos.net/wiki/User_Guide)
