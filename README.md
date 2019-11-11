# Instalar y configurar VyOS como router en un entorno virtualizado con Proxmox

## Autor

- [Ixen Rodríguez Pérez - kurosaki1976](ixenrp1976@gmail.com)

## Instalación

```bash
install image
```

## Configuración

* Parámetros globales

```bash
configure

# global settings
set system host-name 'HOSTNAME'
set system name-server 'ISP_NAME_SERVER'
set system time-zone 'America/Havana'
set system ntp server 'NTP_SEVER_ADDRESS'
delete system console device ttyS0
```

* Parámetros globales

```bash
configure

# interface settings
set interfaces ethernet eth0 vif VID address 'WAN_ADDRESS/CIDR'
set interfaces ethernet eth0 vif VID description 'WAN'
set interfaces ethernet eth0 vif VID address 'LAN_ADDRESS/CIDR'
set interfaces ethernet eth0 vif VID description 'LAN'
```

* Parámetros de enrutamiento

```bash
configure

# route settings
set system gateway-address 'ISP_WAN_ADDRESS'
set protocols static route 0.0.0.0/0 next-hop 'ISP_WAN_ADDRESS'
```

* Parámetros globales

```bash
configure

# service settings
set service ssh listen-address 'LAN_ADDRESS'
set service ssh port '22'
```

* Parámetros de cortafuegos

```bash
configure

# firewall settings
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
set interface ethernet eth0 vif VID firewall local name OutSide
```

* Parámetros de seguridad

```bash
configure

# security settings
set system login user USERNAME full-name 'User Full Name'
set system login user USERNAME authentication plaintext-password 'PASSWORD'
set system login user USERNAME level 'admin'
```

Aplicar la configuración y guardarlos para hacerlos permanentes en cada reinicio.

* Parámetros globales

```bash
commit && save
exit
```
