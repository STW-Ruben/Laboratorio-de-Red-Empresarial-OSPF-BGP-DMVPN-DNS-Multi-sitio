# Laboratorio de Red Empresarial: OSPF + BGP + DMVPN + DNS Multi-sitio

## 🎯 El reto
Diseñar y desplegar una red corporativa multi-sucursal (matriz + 3 sucursales: SDQ, STI y PUJ) donde **solo un sitio tenga salida a Internet propia**, y que el resto de las sucursales puedan salir a la web usando ese único enlace, sin necesidad de contratar circuitos de Internet independientes en cada oficina. Todo esto sobre un backbone jerárquico tipo MPLS (OSPF + BGP) con seguridad de extremo a extremo vía DMVPN.

## 🧠 Arquitectura implementada

**Backbone (Core/WAN):**
- **OSPF multi-área** como IGP interno, con autenticación MD5 (IPv4) e IPSec/SHA1 (IPv6) en cada enlace punto a punto, y `router-id` fijo por dispositivo.
- **BGP (eBGP)** entre el router de borde de la sucursal SDQ (AS 65001) y el ISP/HQ (AS 65000), con autenticación MD5 en la sesión y filtrado estricto mediante **prefix-lists y route-maps** (nada se anuncia "a ciegas").
- Un **route-reflector (HQ-RR)** en la matriz para escalar el iBGP/OSPF entre los tres HQ-Rx sin necesidad de mallar todas las sesiones.

**Internet compartido entre sucursales (el punto clave):**
- Únicamente **BPD-SDQ** tiene el circuito físico a Internet (NAT overload + peering BGP con el ISP).
- Ese router inyecta una **ruta por defecto (`default-originate` / `default-information originate`)** hacia el resto del backbone OSPF.
- Resultado: **STI y PUJ navegan a través del enlace de SDQ sin tener su propio ISP**, demostrando failover/centralización de salida a Internet y ahorro de costos de circuitos redundantes.

**DMVPN (seguridad sobre el transporte):**
- **Hub** en FW-SDQ, **spokes** en FW-STI y FW-PUJ, usando **mGRE + NHRP** para el descubrimiento dinámico de túneles.
- Cifrado con **IPsec (IKEv2, AES-256, SHA-256, DH grupo 14)** en modo transporte, protegiendo tanto IPv4 como IPv6 (dual-stack real, no solo v4).

**VPN site-to-site como respaldo (STI ↔ PUJ):**
- Además del hub-spoke, hay un **túnel GRE sobre IPsec dedicado y directo entre las sucursales STI y PUJ** (perfil criptográfico independiente al del DMVPN, con su propio par IKEv2 y llave precompartida).
- Funciona como **plan B ante una caída del DMVPN** (falla del hub SDQ o de la nube NHRP): si el hub deja de estar disponible, el tráfico entre STI y PUJ sigue fluyendo por esta ruta directa en vez de depender del hub.
- Integrado a OSPF como un enlace más, así que el failover es automático por costo/routing, sin intervención manual.

**DNS corporativo (BIND9):**
- Servidor DNS autoritativo/recursivo (`popular.com.do`) en dual-stack, con `allow-query`/`allow-recursion` restringidos a las redes internas (no es resolver abierto) y `forward only` hacia 8.8.8.8.

**Otras tecnologías del laboratorio** (sin entrar en detalle para no extender demasiado):
DHCP dual-stack por VLAN, port-security, DHCP snooping + ARP inspection, VTP en modo transparente, SSHv2 con protección anti fuerza bruta, y ACLs de firewall entre sitios.

## ✅ Resultado
Una red convergida y resiliente: **un solo punto de salida a Internet compartido entre sucursales**, DMVPN como transporte cifrado principal con un **túnel site-to-site de contingencia entre STI y PUJ**, resolución de nombres centralizada y alta disponibilidad de enrutamiento con OSPF+BGP. Laboratorio probado y validado con capturas de tráfico y pruebas funcionales end-to-end.

---
*#Networking #CiscoIOS #BGP #OSPF #DMVPN #DNS #IPv6 #Ciberseguridad #InfraestructuraDeRed*
