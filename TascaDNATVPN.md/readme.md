# README.md - Activitat AA4: NAT i VPN

Aquest document descriu l’activitat **AA4: NAT i VPN** del curs *Serveis de xarxa* (CFGM SMX). L’objectiu és configurar i comparar dos mètodes d’accés remot a serveis interns: **Destination NAT (DNAT)** i **Virtual Private Network (VPN)**.

## Escenari de pràctiques

Es disposa de tres màquines virtuals:

| Màquina | Rol | Xarxes |
|---------|-----|--------|
| **IPFire** | Router / firewall | eth0: xarxa NAT (exterior) - DHCP<br>eth1: xarxa interna (192.169.x.0/24) |
| **Zorin** | Servidor intern | Xarxa interna (192.169.x.0/24) |
| **Client** | Equip exterior | Xarxa NAT (mateixa subxarxa que eth0 d’IPFire) |

L’IPFire actua com a passarel·la. El Zorin no té accés directe des de l’exterior; inicialment només és accessible des de la xarxa interna.

## Objectius

1. **Destination NAT (DNAT)**  
   - Exposar els serveis SSH (port 22) i HTTP (port 80) del Zorin a l’exterior.  
   - Configurar regles de *port forwarding* a l’IPFire.  
   - Comprovar l’accés des del Client utilitzant l’adreça “pública” de l’IPFire.

2. **Virtual Private Network (VPN) amb OpenVPN**  
   - Configurar el servidor VPN a l’IPFire (generació de certificats, creació d’una connexió Host-to-net).  
   - Instal·lar i configurar el client OpenVPN a la màquina Client.  
   - Connectar el Client a la xarxa interna mitjançant VPN.  
   - Accedir als serveis del Zorin utilitzant la seva adreça privada (192.169.x.1).

3. **Comparativa**  
   - Analitzar els avantatges i inconvenients de cada solució (seguretat, transparència, facilitat d’ús, etc.).

## Tasques detallades

### 1. Preparació inicial

- Comprovar que tant el Zorin com el Client tenen connectivitat a Internet (ping a 8.8.8.8).  
- Al Zorin, instal·lar els paquets `apache2` i `openssh-server`.  
- Modificar l’arxiu `index.html` del servidor web perquè mostri un missatge personalitzat (p. ex., el nom de l’alumne).  
- Verificar localment que els serveis funcionen (curl a localhost, ssh localhost).

### 2. Configuració de Destination NAT (DNAT)

- Accedir a la interfície web d’administració de l’IPFire.  
- Anar a **Firewall > Firewall Rules** i crear dues regles noves:  
  - **Regla SSH**: redirigeix el port 22 de la interfície RED cap al port 22 del Zorin (IP interna).  
  - **Regla HTTP**: redirigeix el port 80 de la interfície RED cap al port 80 del Zorin.  
- Aplicar els canvis.  
- Des del Client (exterior), connectar-se a l’adreça IP de la interfície RED de l’IPFire:  
  - Navegador: `http://<IP_IPFire>` → ha de mostrar la pàgina personalitzada.  
  - Terminal: `ssh usuari@<IP_IPFire>` → ha de permetre l’accés al Zorin.

### 3. Configuració de VPN (OpenVPN)

#### Al servidor IPFire

- Accedir a **Servicios > OpenVPN > Autoridades de Certificado**.  
- Generar el certificat root i el certificat del servidor (omplir les dades organitzatives).  
- Anar a **OpenVPN > Control y Status de conexión** i fer clic a **Agregar**.  
- Triar el tipus *Conexión de un equipo a la red* (Host-to-net).  
- Assignar un nom a la connexió (p. ex., “Serveis20”).  
- Escollir un rang d’adreces IP dinàmiques per als clients (p. ex., 10.53.73.0/24).  
- A les opcions avançades, permetre l’accés del client a la xarxa **Green** (la LAN interna).  
- Configurar l’autenticació: nom d’usuari, organització, contrasenya PKCS12.  
- Guardar la connexió i, tot seguit, descarregar els fitxers de configuració del client (`.ovpn` i `.p12`).

#### Al client (Windows o Linux)

- Editar el fitxer `hosts` del sistema operatiu per afegir una entrada que resolgui el nom del servidor VPN (el mateix que es va posar a IPFire, p. ex., `192.169.2.254   aran20.foodlogistic.test`).  
- Instal·lar el programari client d’OpenVPN (versió Community).  
- Copiar els fitxers `.ovpn` i `.p12` descarregats a una carpeta accessible.  
- Des del gestor de xarxa, importar el fitxer `.ovpn` com a nova connexió VPN.  
- A la configuració d’autenticació, seleccionar *Password with Certificates (TLS)*, indicar l’usuari creat i associar el fitxer `.p12` com a certificat/clau.  
- Connectar la VPN. L’estat ha de passar a “connectat”.

#### Comprovacions amb VPN activa

- Un cop establerta la connexió VPN, el Client obté una IP dins del rang assignat (p. ex., 10.53.73.x).  
- El Client pot accedir als serveis del Zorin utilitzant la seva **adreça IP privada** (192.169.2.1):  
  - `curl http://192.169.2.1` → mostra la pàgina personalitzada.  
  - `ssh usuari@192.169.2.1` → accés al terminal del Zorin.

### 4. Comparativa DNAT vs VPN

| Aspecte | DNAT | VPN |
|---------|------|-----|
| **Nivell d’accés** | Només als ports específics (22, 80). | Accés complet a tota la xarxa interna (com si fos un equip local). |
| **Seguretat** | Cada servei exposat individualment; cal controlar vulnerabilitats per port. | Xifrat extrem a extrem, autenticació mitjançant certificats. |
| **Transparència** | L’usuari es connecta a l’adreça pública del router. | L’usuari obté una IP interna i pot usar noms o IPs privades. |
| **Configuració** | Senzilla (regles de port forwarding). | Més complexa (PKI, certificats, configuració client). |
| **Cas d’ús típic** | Exposar serveis concrets a Internet (web, SSH). | Teletreball, accés segur a tota la LAN. |

## Lliurament

L’alumne ha de presentar un document (PDF o Markdown) que inclogui:
- Captures de pantalla de cada pas significatiu (comprovacions de connectivitat, regles DNAT, generació de certificats, connexió VPN, accés als serveis).
- Una breu explicació de cada captura.
- Una taula comparativa entre DNAT i VPN.

