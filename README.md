# 🛡️ Arquitectura de Seguridad de Red y Defensa en Profundidad (Defense in Depth)

Este proyecto documenta el diseño, configuración y auditoría de una infraestructura de red segura utilizando el modelo de **Defensa en Profundidad**. El objetivo es demostrar cómo la implementación de múltiples capas de firewalls y políticas de mitigación protegen los activos críticos contra vulnerabilidades reales, movimientos laterales y explotación de configuraciones por defecto.

## 📋 Fundamentos del Diseño: La Importancia de una Red Bien Configurada

Las redes empresariales modernas no pueden depender de un único perímetro de seguridad. Las vulnerabilidades reales (como CVEs en servicios expuestos, ataques de *VLAN Hopping*, o suplantación de identidad mediante *ARP Spoofing*) requieren que la seguridad esté embebida en la propia arquitectura de ruteo y conmutación. Una red bien configurada aísla los fallos, limita el radio de explosión de un ataque y proporciona a los operadores NOC la visibilidad necesaria para reaccionar antes de que ocurra una exfiltración de datos.

---

## 🏗️ Topología del Laboratorio: Capas de Firewall (Cortafuegos)

El laboratorio emula un entorno corporativo dividiendo la seguridad en tres fronteras lógicas, implementando diferentes tipos de inspección y filtrado de tráfico.

### Capa 1: Perímetro y Borde (Network Edge Firewall)
La primera línea de defensa separa la red corporativa de internet (WAN).
*   **Tecnología:** Cisco IOS Edge Router.
*   **Implementación:**
    *   Configuración de **NAT/PAT** para ocultar el direccionamiento interno.
    *   **ACLs Extendidas (Access Control Lists):** Bloqueo explícito de tráfico entrante no solicitado (Spoofing de IPs, tráfico ICMP masivo para evitar ataques DoS).
    *   **Inspección de Estado (CBAC / Zone-Based Firewall conceptual):** Permitir el tráfico de retorno dinámicamente solo para las sesiones iniciadas desde la red interna.

### Capa 2: Segmentación Interna (Inter-VLAN Firewalling)
Un atacante que logra comprometer un equipo de la red de invitados no debe tener acceso a la red de servidores.
*   **Tecnología:** Switch Multicapa Cisco (L3) / Router on a Stick (ROAS).
*   **Implementación:**
    *   Aislamiento estricto de subredes mediante VLANs (VLAN de Gestión, VLAN de Usuarios, VLAN de Servidores DMZ).
    *   **ACLs Estándar y Extendidas Internas:** Reglas que permiten a los usuarios consumir servicios HTTP/HTTPS del servidor, pero bloquean el acceso SSH o ICMP hacia la VLAN de Gestión.

### Capa 3: Firewall de Host / Capa de Aplicación (Endpoint Security)
Si la seguridad de la red falla, el sistema operativo del servidor debe ser capaz de defenderse a sí mismo.
*   **Tecnología:** Servidor Linux (Debian/Ubuntu) con `iptables` / `UFW`.
*   **Implementación:**
    *   **Política Default Drop:** Todo el tráfico entrante es denegado por defecto.
    *   Apertura granular exclusivamente para puertos de producción (ej. 80, 443) y administración segura (ej. SSH en puerto no estándar 2222 solo desde la IP del administrador).
    *   **Prevención de Intrusos (Fail2ban):** Análisis de logs en tiempo real para bloquear dinámicamente IPs en el firewall local tras detectar patrones de ataque de fuerza bruta.

---

## 🦠 Mitigación de Vulnerabilidades Reales (L2 y L3)

Esta arquitectura no solo filtra puertos, sino que ataca problemas endémicos de la infraestructura de red aplicando "Hardening" avanzado:

*   **Ataques de Interceptación (Man-In-The-Middle / ARP Spoofing):**
    *   *Solución implementada:* Configuración de **Dynamic ARP Inspection (DAI)** y **DHCP Snooping** en los switches de acceso para garantizar que solo los servidores DHCP confiables asignen IPs y evitar la falsificación de tablas MAC.
*   **VLAN Hopping (Salto de VLAN):**
    *   *Vulnerabilidad:* Un atacante engaña al switch para negociar un enlace troncal (Trunk) y acceder a todas las redes.
    *   *Solución implementada:* Desactivación del protocolo DTP (`switchport nonegotiate`), configuración manual de puertos de acceso (`switchport mode access`) y asignación de puertos sin uso a una VLAN aislada (Agujero negro).
*   **Agotamiento de Direcciones MAC (MAC Flooding):**
    *   *Solución implementada:* **Port Security** con límite de direcciones MAC por puerto y acción restrictiva (`violation restrict` o `shutdown`) para prevenir que herramientas comprometan la tabla CAM del switch.
*   **Compromiso de Credenciales en Texto Plano:**
    *   *Solución implementada:* Erradicación de protocolos inseguros (Telnet, HTTP, SNMPv1/v2). Implementación obligatoria de **SSHv2**, encriptación de contraseñas de configuración (`service password-encryption` / `secret`) y **SNMPv3** con autenticación y cifrado para el monitoreo del NOC.

---

## 🛠️ Herramientas Utilizadas
*   **Diseño y Simulación:** Cisco Packet Tracer.
*   **Sistemas Operativos:** Cisco IOS 15.x, Debian/Ubuntu Linux.
*   **Seguridad y Diagnóstico:** `iptables`, `ufw`, `fail2ban`, Wireshark.

*   
