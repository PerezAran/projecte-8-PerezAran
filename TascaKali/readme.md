# Projecte d'Exploració de Xarxa amb Kali Linux

Aquest repositori conté la documentació tècnica i les evidències de l'activitat d'escaneig i descobriment de hosts en una xarxa local utilitzant eines de seguretat en Kali Linux.

## 📋 Descripció de la Tasca
L'objectiu principal d'aquesta activitat és utilitzar i comparar diferents eines de reconeixement de xarxa per identificar dispositius actius, serveis i sistemes operatius dins d'una subxarxa específica.

S'han treballat principalment dues eines clau:
- **Netdiscover**: Per al descobriment basat en el protocol ARP (Capa 2).
- **Nmap**: Per a un escaneig més profund basat en protocols de Capa 3 i 4.

## 🛠️ Eines Utilitzades
- **Sistema Operatiu**: Kali Linux
- **Eines d'escaneig**:
    - `netdiscover` (modes actiu i passiu)
    - `nmap` (ping scan, detecció d'OS i serveis)

## 📂 Estructura del Projecte
- `guia_tecnica.md`: Documentació detallada de tots els passos seguits.
- `img/`: Carpeta que conté les captures de pantalla (evidències) numerades del procés.

## 🚀 Resum de l'Execució
1. **Fase 1**: Verificació inicial de les eines.
2. **Fase 2**: Escaneig de xarxa amb Netdiscover (Comparativa entre enviament de paquets actius i escolta passiva).
3. **Fase 3**: Ús de Nmap per al mapatge de la xarxa i intent d'identificació de sistemes remots.
4. **Fase 4**: Anàlisi comparativa de resultats.

## 📊 Resultats Clau
- Es van identificar fins a **11 hosts** en mode actiu amb Netdiscover.
- Es va determinar que el mode passiu és més discret però menys exhaustiu.
- Nmap va permetre una anàlisi més tècnica, tot i que algunes configuracions de seguretat van bloquejar la detecció del sistema operatiu.

---
*Aquesta tasca s'ha realitzat com a part del mòdul de seguretat de sistemes microinformàtics i xarxes (SMiX).*