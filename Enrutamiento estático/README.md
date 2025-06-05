# Enrutamiento estático

### Objetivo

Tenemos como objetivo principal poder conectar 2 subredes LAN. La idea es hacer que 2 PC's puedan hacer un `ping` entre sí.

### Requerimientos

- 2 Routers
- 2 PC's
- 2 Switches
- 5 Cables ethernet (RJ-45)
- 1 Cable de consola

### 🔥 Parte 1: Conexión de routers, switches, y PC's

Primero deberemos conectar, mediante cables ethernet(RJ-45), los switches, los routers y las PC's.

Seria de la siguiente forma:

![pkt](/img/Captura%20de%20pantalla%202025-06-03%20145351.png)

El cable de router a router es cruzado, pero hoy con los routers nuevos no hacen falta. De igual forma, el packet tracer pide cable cruzado.

### 🔥 Parte 2: Configurar las interfaces de red al switch
Primero debemos configurar las interfaces de red desde el router. Esto podemos hacerlo con un cable consola, conectarlo desde la PC al router. Debería aparecer algo así:

```yml
Router>
```

Supongamos que queremos configurar la LAN A, desde la terminal, debemos configurar las interfaces de red con sus IP. Primero vamos a hacer la interfaz que va al switch. Esto podemos hacerlo introduciendo los siguientes comandos:

```yml
Router> enable
Router# configure terminal
Router(config)# interface FastEthernet 0/1
Router(config-if)# ip add 192.168.3.1 255.255.255.0 // Su IP con máscara de 24 bits.
Router(config-if)# no sh
Router(config-if)# end
Router# wr // Guardamos.
```

Luego de guardar, debería quedar algo así:
```yml
Router# show running-config // Para ver lo que configuramos.

interface FastEthernet0/1
 ip address 192.168.3.1 255.255.255.0
 duplex auto
 speed auto
```

Esto hay que repetirlo con el otro router, sólo que habría que hacerlo con su IP correspondiente. En este caso sería así:

```yml
Router> enable
Router# configure terminal
Router(config)# interface FastEthernet 0/1
Router(config-if)# ip add 172.16.4.1 255.255.255.0
Router(config-if)# no sh
Router(config-if)# end
Router# wr
Router# show running-config

interface FastEthernet0/1
 ip address 172.16.4.1 255.255.255.0
 duplex auto
 speed auto
```

### 🔥 Parte 3: Configuración de router a router (IP de salto)

Estamos en el Router 0 y configuramos las IP de salto:

```yml
Router> enable
Router# configure terminal
Router(config)# interface FastEthernet 0/0
Router(config-if)# ip add 10.10.10.1 255.255.255.252
Router(config-if)# no sh
Router(config-if)# end
Router# wr
```

Pasamos al router 1 y hacemos lo mismo con otra IP de salto que en el router 0:

```yml
Router> enable
Router# configure terminal
Router(config)# interface FastEthernet 0/0
Router(config-if)# ip add 10.10.10.2 255.255.255.252
Router(config-if)# no sh
Router(config-if)# end
Router# wr
```

### 🔥 Parte 4: Configuración DHCP

Para que las PC's tengan una interfaz de red asignada automáticamente, podemos configurar DHCP mediante el router. Para ello, debemos poner los siguientes comandos:

```yml
ip dhcp excluded-address 172.16.4.250 172.16.4.254
ip dhcp pool LAN_B
network 172.16.4.0 255.255.255.0
default-router 172.16.4.1
dns-server 8.8.8.8
```

Luego de configurar el router 1 Configuramos el DHCP del router 0

```yml
ip dhcp excluded-address 192.168.3.250 192.168.3.254
ip dhcp pool LAN_A
network 192.168.3.0 255.255.255.0
default-router 192.168.3.1
dns-server 8.8.8.8
```

### 🔥 Parte 5: Enrutamiento estático

Para que las PC's puedan comunicarse entre sí debemos configurar el enrutamiento entre los routers. Para ello, debemos poner los siguientes comandos desde la LAN A: 

```yml
Router> enable
Router# configure terminal
Router(config)# ip route 172.16.4.0 255.255.255.0 10.10.10.2
Router(config)# end
```

Hacemos lo mismo a la inversa desde la LAN B:

```yml
Router> enable
Router# configure terminal
Router(config)# ip route 192.168.3.0 255.255.255.0 10.10.10.1
Router(config)# end
```

### Conclusión

Con esta guía que acabamos de hacer, desde la PC1 deberíamos poder hacer ping a la PC2 y viceversa:

```yml
C:\>ping 192.168.3.2

Pinging 192.168.3.2 with 32 bytes of data:

Reply from 192.168.3.2: bytes=32 time<1ms TTL=126
Reply from 192.168.3.2: bytes=32 time=9ms TTL=126
Reply from 192.168.3.2: bytes=32 time<1ms TTL=126
Reply from 192.168.3.2: bytes=32 time<1ms TTL=126

Ping statistics for 192.168.3.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 9ms, Average = 2ms
```

```yml
C:\>ping 172.16.4.2

Pinging 172.16.4.2 with 32 bytes of data:

Request timed out.
Reply from 172.16.4.2: bytes=32 time<1ms TTL=126
Reply from 172.16.4.2: bytes=32 time<1ms TTL=126
Reply from 172.16.4.2: bytes=32 time=1ms TTL=126

Ping statistics for 172.16.4.2:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms
```

Además, si una PC levanta una página web con Live Server podemos verla desde la otra máquina copiando la IP y el puerto en un navegador. Por ejemplo:

![grupo](img/image.png)
