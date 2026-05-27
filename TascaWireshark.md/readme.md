# 🧭 Guia d'Anàlisi de Xarxa — Pràctica de Ciberseguretat i Anàlisi de Trànsit amb Wireshark

Aquest repositori conté l'informe tècnic de la **Pràctica d'Anàlisi de Xarxa** realitzada sobre l'entorn de laboratoris d'**EverPia**. L'objectiu d'aquesta tasca és auditar, filtrar i analitzar diferents protocols de xarxa (tals com ICMP, DNS, ARP, FTP, Telnet, SSH i SMTP) mitjançant l'ús de **Wireshark** a Kali Linux per entendre el comportament dels paquets i els riscos associats als protocols no xifrats.

---

## 🛠️ 1. Configuració Prèvia del Laboratori

Per poder realitzar la captura de paquets de manera correcta i simular un entorn d'auditoria real, es va configurar el programari de virtualització (**VirtualBox**) amb els següents paràmetres:

* **Configuració de l'Adaptador a Kali Linux:** * **Mode:** Adaptador en mode pont (*Bridged Adapter*).
* **Mode Promiscu:** `Permet-ho tot` (*Allow All*). Això és crític perquè la targeta de xarxa virtual de Kali pugui capturar paquets que no van dirigits exclusivament a ella.


* **Configuració de l'Adreçament IP intern:**
* **IP de Kali Linux:** `192.168.2.20`
* **Porta d'enllaç (Gateway):** `192.168.2.254`
* **Servidor DNS:** `8.8.8.8` (Google DNS)
* **Verificació:** Es va realitzar un `ping` correcte cap al Gateway (`192.168.2.254`) per comprovar la connectivitat de la xarxa.



---

## 🔍 2. Desenvolupament de l'Activitat i Respostes Tècniques

### 🔹 1. Anàlisi del Protocol ICMP (Ping)

Es va iniciar la captura a Wireshark i es va llançar una comanda `ping` al gateway. Els paquets ICMP treballen amb codis de tipus segons la seva funció:

* **Echo Request (Petició d'eco):** El camp *Type* és **`8`**.
* **Echo Reply (Resposta d'eco):** El camp *Type* és **`0`**.

### 🔹 2. Captura del Trànsit de la Màquina Física (Amfitrió)

* **IP de l'Amfitrió:** `172.0.2.244`
* **Acció:** Es va iniciar la captura a Kali i es va navegar per internet des de la màquina física.
* **Filtre utilitzat:** `ip.addr == 172.0.2.244`
* **Resultat:** **Sí, és possible capturar-lo.** Es van visualitzar paquets UDP provinents dels servidors de Google destinats a la IP de la màquina amfitriona a causa de la configuració del mode promiscu.

### 🔹 3 i 4. Resolució de Noms DNS (Petició i Resposta)

Es va executar la comanda `nslookup www.xtec.cat` des de la consola de Kali aplicant el filtre de Wireshark `dns`.

* **Petició (Query):** El paquet *Standard query* realitza una consulta del **registre de tipus A** (Adreça IPv4) per al domini `www.xtec.cat`.
* **Resposta (Answer):** Al desglossar el camp *Answers* del paquet de resposta, es va determinar que la IP pública de `www.xtec.cat` és **`83.247.151.214`**.

### 🔹 5 i 6. Protocol ARP (Resolució d'Adreces MAC)

* **MAC del Gateway (Local):** Aplicant el filtre `arp`, es va interceptar el paquet de resposta on la MAC d'origen (*Src*) de la porta d'enllaç era `08:00:27:be:1b:a8`. Analitzant el OUI (els 3 primers bytes `08:00:27`), es confirma que el fabricant de la targeta és **PCS Systemtechnik GmbH (Oracle VirtualBox)**.
* **MAC de 192.168.1.1 (Fitxer `captura1.pcapng`):** Aplicant el filtre `arp and ip.addr == 192.168.1.1`, s'observa que l'adreça física corresponent és **`d4:76:ea:0f:fd:58`**.

### 🔹 7. Auditoria de Seguretat sobre FTP (Trànsit en Text Clar)

Analitzant el fitxer `captura1.pcapng` amb el filtre `ftp` i seguint el flux TCP (*Follow TCP Stream*):

* **Contrasenya de l'usuari:** Es va interceptar la comanda `PASS contra`, exposant que el password és **`contra`**.
* **Fitxer transferit:** Es va localitzar la comanda `RETR README.txt`, de manera que el fitxer descarregat és **`README.txt`**.
* *Conclusió:* FTP és un protocol insegur ja que viatja sense xifrar.

### 🔹 8 i 9. Anàlisi del Protocol Telnet

* **Flux analitzat:** `tcp.stream eq 333` (corresponent al trànsit de Telnet a la IP destí `94.142.241.111`).
* **Visualització de l'usuari:** Es va reconstruir la sessió on es mostrava un dibuix artístic en format **ASCII d'una nau espacial**. Els caràcters utilitzats per a la seva composició són: `O`, `=`, `<`, `8`, `0`, `E`, `I`, `_` i espais en blanc.
* **Resolució del Domini:** La IP destí `94.142.241.111` no va retornar cap resolució a un domini conegut.

### 🔹 10 i 11. Auditoria de Seguretat sobre SSH

A la mateixa `captura1.pcapng` utilitzant el filtre `ssh`:

* **IP del Servidor SSH:** `205.166.94.17`
* **Anàlisi del paquet de 326 bytes (`frame.len == 326`):** S'observa el paquet numeri `24104`. A diferència del Telnet o l'FTP, el contingut es mostra completament com a ***Encrypted*** (Xifrat). No es pot llegir absolutament res del codi ni de les dades de l'usuari.
* *Conclusió:* SSH garanteix la confidencialitat de les dades.

### 🔹 12. Intercepció de Correu Electrònic SMTP (`captura2.pcapng`)

Aplicant el filtre `smtp` i seguint el flux TCP de la sessió de correu electrònic:

* **Missatge robat:** S'ha extret el contingut del text del correu, el qual deia exactament: **`mensaje ultrasectro para el administrador`**.
* **Fitxers adjunts:** Després de revisar l'arbre de protocols de la sessió, es confirma que **no hi havia cap fitxer adjunt** en l'enviament.

---

## 📊 Taula Resum de Protocols i Seguretat

A continuació es presenta una taula corporativa per als enginyers d'EverPia que resumeix el nivell de seguretat observat a la pràctica:

| Protocol | Port per defecte | Estat de la Informació | Risc de Seguretat | Recomanació d'EverPia |
| --- | --- | --- | --- | --- |
| **ICMP** | N/A | Text clar (Codis) | Baix (Mapeig de xarxa) | Monitoritzar pings massius |
| **DNS** | 53 | Text clar | Mitjà (DNS Spoofing / Hijacking) | Implementar DNSSEC / DoH |
| **ARP** | N/A | Text clar | Mitjà (ARP Spoofing) | Configurar taules ARP estàtiques |
| **FTP** | 21 | 🔴 **Text clar (Sense xifrar)** | Alt (Robatori de credencials i fitxers) | Migrar immediatament a **SFTP** |
| **Telnet** | 23 | 🔴 **Text clar (Sense xifrar)** | Alt (Intercepció de comandes) | Migrar immediatament a **SSH** |
| **SSH** | 22 | 🟢 **Xifratge fort (AES/RSA)** | Molt Baix | Protocol estàndard segur |
| **SMTP** | 25 | 🔴 **Text clar (Sense xifrar)** | Alt (Lectura de correus privats) | Forçar l'ús de **SMTPS (Port 465/587)** |

---

*Document de treball intern de la base de coneixement de ciberseguretat d'EverPia.* 🔐
