# Instalación y puesta en marcha de ISC-DHCP en Ubuntu Server

## Autor

- [q3it](https://www.blogger.com/profile/16118326082555553765)

## Resumen

DHCP son las siglas de Dynamic Host Configuration Protocol (protocolo de configuración dinámica de host). Se trata de un protocolo de red utilizado para asignar y gestionar automáticamente direcciones IP (Protocolo de Internet), máscaras de subred, direcciones de pasarela y otros parámetros de configuración de red a los dispositivos de una red TCP/IP. DHCP trabaja en nuestros hogares y conecta nuestros dispositivos, a menudo sin que nos demos cuenta. Asigna direcciones IP privadas a nuestros ordenadores, smartphones, dispositivos Wi-Fi y gadgets, todos los cuales utilizan una única red. El servicio DHCP simplifica el proceso de configuración de los parámetros de red para los dispositivos y ayuda a garantizar un uso eficiente de las direcciones IP dentro de una red. Llevaría mucho tiempo configurarlas manualmente, así que DHCP lo hace sin esfuerzo, la mayoría de las veces sin que pensemos en ello.

En las [referencias](#referencias), encontrará enlaces a sitios de `Internet` que pueden ayudar.

## Escenario

<p align="center">
<img src="/img/Screenshot.png">
</p>

Se crearán dos máquinas virtuales: Una con Ubuntu server que nos servirá como DHCP y otra con Debian que será el cliente. En el diagrama de red tendrémos una idea de como se van a comunicar las máquinas.

Debian estará en una red interna que se conectará a Ubuntu Server, y para ello tendrá un adaptador de red sin conexión a internet. Ubuntu Server por su lado se le configurará un adaptador como red interna y otro que hará de router para tener acceso a internet.

## Administración del servidor

El servidor `Ubuntu` utilizá los siguientes parámetros de configuración de red:

* Dirección `IP` del servidor con conexión wan: `192.168.210.145`
* Dirección `IP` del servidor con conexión lan: `192.168.277.133`
* Puerta de enlace del servidor: `192.168.277.1`
* Red interna: `192.168.277.0/24`
* Red externa: `192.168.210.0/24`

### Ajustes de los parámetros de red Ubuntu Server

La configuración de las tarjetas de red del servidor se encuentran detalladas en el repositorio [Router Server](https://github.com/q3it/Router-Ubuntu-Server).

### Instalación y configuración de la herramienta

Ejecutamos el comando `apt install isc-dhcp-server` en consola.

- Editamos los dos ficheros de configuración: `/etc/dhcp/dhcpd.conf` y `/etc/default/isc-dhcp-server`.

> [!NOTE]
> Anexo en este repositorio se encuentran los ficheros completos y el punto exacto dónde se modifican.

```ruby
nano /etc/dhcp/dhcpd.conf

# Red interna práctica DHCP
group qbit{
subnet 192.168.227.0 netmask 255.255.255.0 {
	range 192.168.227.140 192.168.227.150;
	option domain-name-servers 192.168.227.133;
	option domain-name "clockwork.local";
	option subnet-mask 255.255.255.0;
	option routers 192.168.227.133;
	option broadcast-address 192.168.227.255;
	}
}
```

```ruby
nano /etc/default/isc-dhcp-server

# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#	Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACESv4="ens256"
INTERFACESv6=""
```

- Reiniciamos el servicio.

```bash
service isc-dhcp-server restart
```

-  Comprobamos el estado.

```bash
service isc-dhcp-server status
```
## Consultamos IP Debian

- En la máquina `Debian` ejecutamos un `ip a`.

<p align="center">
<img src="/img/ip.png">
</p>

- Vemos las conexiones de red.

<p align="center">
<img src="/img/red.png">
</p>

## Ip estática en Debian.

- Para que la máquina Debian siempre tenga la misma IP asignada desde DHCP Server, aplicamos los siguientes cambios.

```ruby
nano /etc/dhcp/dhcpd.conf

host debi{
        hardware ethernet 00:0C:29:CC:92:EE;
        fixed-address 192.168.227.140;
}
```

> [!TIP]
> El desarrollo de esta práctica la encontraras en mi [blog](https://www.thequbit.net/2024/09/dhcp-ubuntu-server.html).

## Conclusiones

DHCP automatiza y gestiona de forma centralizada las direcciones IP, en lugar de requerir que los administradores de red asignen las direcciones por su cuenta bajo el riesgo de asignar direcciones idénticas, lo que generará un posible conflicto.

## Referencias

* [DHCP Wiki](https://es.wikipedia.org/wiki/Protocolo_de_configuraci%C3%B3n_din%C3%A1mica_de_host)
* [ISC](https://www.isc.org/dhcp/)
* [DHCP Ubuntu Server](https://help.ubuntu.com/community/isc-dhcp-server)
