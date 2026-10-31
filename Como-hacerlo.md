# Cómo hacer las prácticas en Packet Tracer

Guía con los pasos y las direcciones IP que pide la lectura. Está pensada para
ejecutarla con el programa abierto, en **modo Realtime** (no Simulation).

---

## 0 · Antes de arrancar

1. Abrí Packet Tracer.
2. **File → Save As** y guardá el archivo como `.pkt` **antes de tocar nada**. Así,
   si algo sale mal, tenés de dónde volver.
3. Fijate qué versión tenés: **Help → About Packet Tracer**. Importante para el
   final de esta guía.

---

## 1 · Tres PC y un switch

| Paso | Qué hacer | Dónde |
|---|---|---|
| 1 | Arrastrá un switch **2950-24** | Network Devices → Switches |
| 2 | Arrastrá tres **PC-PT** | End Devices → End Devices |
| 3 | Cable **Copper Straight-Through** desde el PC al switch | Connections (el ícono del rayo) |

Para conectar: elegí el cable, hacé clic en el PC y elegí **FastEthernet0**, después
clic en el switch y elegí **FastEthernet0/1**. Repetí con los otros dos PC en
`0/2` y `0/3`.

**Las IPs** (clic en cada PC → Desktop → IP Configuration):

| Equipo | IP | Máscara | Gateway |
|---|---|---|---|
| PC0 | 192.168.1.2 | 255.255.255.0 | 192.168.1.1 |
| PC1 | 192.168.1.3 | 255.255.255.0 | 192.168.1.1 |
| PC2 | 192.168.1.4 | 255.255.255.0 | 192.168.1.1 |

> El gateway `192.168.1.1` no existe en esta práctica porque no hay router. La guía
> lo pide igual; no molesta, porque el ping es entre equipos de la misma subred.

**Verificar:** esperá a que los triángulos de los cables se pongan **verdes**, después
en PC0 → Desktop → Command Prompt:

```
ipconfig
ping 192.168.1.3
```

Tienen que responder los 4 paquetes. Si el primero falla, es normal: es el ARP.

---

## 2 · Red inalámbrica con Access Point

| Paso | Qué hacer | Dónde |
|---|---|---|
| 1 | Arrastrá un **AccessPoint-PT** | Network Devices → Wireless Devices |
| 2 | Arrastrá dos PC con tarjeta inalámbrica | Custom Made Devices |
| 3 | Esperá: se conectan solos al AP | — |

IPs: `192.168.1.11` y `192.168.1.12`, máscara `255.255.255.0`, gateway `192.168.1.1`.

Ojo: **que aparezca la conexión no significa que haya comunicación**. Hay que cargar
las IPs igual. Verificá con `ping 192.168.1.11` desde la otra PC.

---

## 3 · Híbrida: switch + Access Point

Es la práctica 1 y la 2 juntas, con las mismas IPs. Lo único distinto:

**Los dos concentradores tienen que estar unidos por un cable.** Conectá el
**switch** con el **Access Point** usando un **cable directo** (Copper
Straight-Through). Sin ese cable, la red cableada y la inalámbrica no se ven.

---

## 4 · Router inalámbrico con dos subredes

Acá cambia todo: la red cableada usa **IP estática** y la inalámbrica **DHCP**, y
son dos subredes distintas.

### Red cableada (estática) — `192.168.2.0`

| Equipo | IP | Máscara | Gateway |
|---|---|---|---|
| PC0 | 192.168.2.2 | 255.255.255.0 | 192.168.2.1 |
| PC1 | 192.168.2.3 | 255.255.255.0 | 192.168.2.1 |

El cable del **switch** va al puerto **INTERNET** del router inalámbrico (así lo
indica la guía, aunque parezca raro). Sobre ese puerto se carga la dirección de
gateway de la red cableada.

### Red inalámbrica (DHCP) — `192.168.1.0`

Clic en el router → pestaña **GUI**. Ahí se carga:

| Campo | Valor |
|---|---|
| IP Address | 192.168.1.1 |
| Subnet Mask | 255.255.255.0 |
| DHCP Enabled | **Sí** |
| Start IP Address | 192.168.1.100 |
| Maximum Number of Users | 50 |
| Rango que reparte | 192.168.1.100 – 192.168.1.149 |

Las PC inalámbricas van con **DHCP** (Desktop → IP Configuration → DHCP), no con IP
fija.

**Verificar:** desde la PC cableada `192.168.2.2` hacé ping a la inalámbrica
`192.168.1.102`.

---

## 5 · WEP en la red inalámbrica

1. Clic en el router → **Config** → sección **Wireless Security**.
2. Elegí **WEP** y poné una clave de **10 caracteres como mínimo**.
3. En **cada PC inalámbrica**: Desktop → **PC Wireless** → pestaña **Connect** →
   Security → WEP, y la **misma** clave.

Si la clave no coincide, el equipo no se conecta: es el comportamiento esperado.

---

## 6 · Configurar RIP

RIP es ruteo **dinámico**: los routers se avisan solos qué redes conocen, en vez de
que vos escribas las rutas a mano.

**Ojo con esto:** el **Router Inalámbrico** de las prácticas anteriores es un router
doméstico: tiene interfaz gráfica (GUI) pero **no tiene consola IOS**, así que no
soporta RIP. Para RIP hay que usar routers Cisco: **2911** o **1941**.

### Topología de ejemplo

Dos routers unidos entre sí, cada uno con su LAN:

| Router | Interfaz | IP | Para qué |
|---|---|---|---|
| R1 | GigabitEthernet0/0 | 10.0.0.1 / 255.255.255.252 | enlace hacia R2 |
| R1 | GigabitEthernet0/1 | 192.168.1.1 / 255.255.255.0 | LAN 1 |
| R2 | GigabitEthernet0/0 | 10.0.0.2 / 255.255.255.252 | enlace hacia R1 |
| R2 | GigabitEthernet0/1 | 192.168.2.1 / 255.255.255.0 | LAN 2 |

El cable entre los dos routers va **cruzado** (Copper Cross-Over) si usás
GigabitEthernet.

Las PC: `192.168.1.10` con gateway `192.168.1.1`, y `192.168.2.10` con gateway
`192.168.2.1`.

### Comandos en R1

Clic en el router → pestaña **CLI** → enter para llegar al `>`:

```
enable
configure terminal

interface GigabitEthernet0/0
 ip address 10.0.0.1 255.255.255.252
 no shutdown
exit

interface GigabitEthernet0/1
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit

router rip
 version 2
 no auto-summary
 network 10.0.0.0
 network 192.168.1.0
 passive-interface GigabitEthernet0/1
exit

end
write memory
```

### Comandos en R2

Lo mismo, cambiando las direcciones:

```
enable
configure terminal

interface GigabitEthernet0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit

interface GigabitEthernet0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
exit

router rip
 version 2
 no auto-summary
 network 10.0.0.0
 network 192.168.2.0
 passive-interface GigabitEthernet0/1
exit

end
write memory
```

### Cómo saber si funcionó

En cada router:

```
show ip protocols
show ip route
```

En `show ip protocols` tiene que aparecer **Routing Protocol is "rip"**. En
`show ip route` la red del otro router tiene que salir con una **R** adelante: eso
significa que la aprendió por RIP.

Y la prueba final: desde la PC de la LAN 1, `ping 192.168.2.10`.

### Los cuatro errores que siempre aparecen

1. **Falta `version 2`.** RIP versión 1 no manda la máscara de subred, así que con
   subredes no funciona.
2. **Falta `no auto-summary`.** Sin eso el router resume las redes y se pierden las
   subredes.
3. **`network` con máscara.** En RIP va la dirección de red **sola**:
   `network 192.168.1.0`, no `network 192.168.1.0 255.255.255.0`.
4. **Las PC sin gateway.** El ping falla aunque RIP esté perfecto, porque la PC no
   sabe a quién mandarle el tráfico de otra red.

## 7 · El tema de las versiones (importante)

Un archivo `.pkt` **guardado con una versión más nueva no abre en una más vieja**.
Packet Tracer no tiene "guardar como versión anterior".

Entonces:

- Si la facultad tiene **la misma versión o más nueva** que la tuya, el archivo abre.
- Si tiene **una más vieja**, no abre. La solución es instalar esa versión en tu PC
  (según el instructivo de la cátedra, dos versiones conviven sin problema) y
  guardar el `.pkt` desde ahí.
- Para saber qué versión te pide un archivo: intentá abrirlo. Si es más nuevo, el
  programa te lo dice con un cartel.

**Recomendación:** preguntá en la cátedra qué versión usan **antes** de entregar, y
guardá el `.pkt` con esa. Es el error más común y no se arregla después.
