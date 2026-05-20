# README: Configuració d’un proxy filtrat amb IPFire

Aquest document resumeix la pràctica realitzada per configurar un **proxy web amb filtrat de contingut** a IPFire, seguint els requisits del professor. S’han aplicat bloquejos per categories, dominis, URLs concretes, paraules clau i franges horàries.

## 🔧 Requisits complerts

| # | Requisit | Com s’ha implementat | Comprovació |
|---|----------|----------------------|--------------|
| 1 | Mostrar la URL bloquejada a la pàgina de denegació | Plantilla per defecte d’IPFire | En qualsevol bloqueig apareix la URL sol·licitada |
| 2 | Instal·lar llistes negres | *URL Filter → Maintenance* → descàrrega automàtica | Llistes actualitzades correctament |
| 3 | Bloquejar categories *bank* i *radio* | Marcar *financial* i *radio* a *Categorías bloqueadas* | `ing.es` (bank) i `ah.fm` (radio) bloquejats |
| 4 | Bloquejar dominis complets: `elnacional.cat`, `tecnocampus.cat` | *Lista Negra personalizada → Dominios bloqueados* | Accés denegat a tots dos dominis |
| 5 | Bloquejar una URL concreta d’un domini sense bloquejar la resta | *URLs bloquejades* → afegir `https://www.youtube.com/` | `youtube.com` bloquejat, altres dominis de Google funcionen |
| 6 | Bloquejar pàgines amb “anime”, excepte `animenewsnetwork.com` | *Frases bloquejades* = `anime`; *Lista Blanca* = `animenewsnetwork.com` | `animebusca.com` bloquejat, `animenewsnetwork.com` accessible |
| 7 | Bloqueig per hores | Regla de firewall amb restricció temporal (dimarts a divendres, 16-17h) | Fora d’aquest horari, el trànsit web es denega |

## 🖥️ Entorn de treball

- **IPFire** 2.29 (x86_64)
- Interfícies:
  - **GREEN**: `192.169.20.254/24` (xarxa interna)
  - **RED**: IP dinàmica (DHCP) per a connexió a Internet
- Client a GREEN: `192.169.20.1`

## 📁 Estructura de la configuració

```
IPFire
├── Advanced Web Proxy (mode transparent a GREEN, port 3128)
├── URL Filter
│   ├── Categories bloquejades: financial, radio
│   ├── Llista negra personalitzada (dominis)
│   ├── Llista d’URLs bloquejades (concretes)
│   ├── Llista de frases bloquejades: "anime"
│   └── Llista blanca personalitzada: animenewsnetwork.com
└── Firewall Rules
    └── Regla horària per a GREEN → RED (ports 80,443)
```

## ✅ Validació dels bloquejos

### Bloqueig per categoria
- Accés a `ing.es` → `ERR_TUNNEL_CONNECTION_FAILED`
- Accés a `ah.fm` → pàgina `ACCESS DENIED` d’IPFire

### Bloqueig de dominis sencers
- `elnacional.cat` i `tecnocampus.cat` → missatge denegat

### Bloqueig d’URL concreta
- `https://www.youtube.com/` → error DNS (proxy bloqueja la resolució)
- Altres pàgines del mateix domini (ex: `https://accounts.google.com`) funcionen normalment

### Bloqueig per paraula clau “anime”
- `animebusca.com`, `animefiguras.com` → error DNS / denegat
- `animenewsnetwork.com` → carrega amb normalitat

### Bloqueig horari
- Regla activa només de dimarts a divendres, 16:00–17:00
- Qualsevol intent fora d’aquest període rep bloqueig del tallafocs

## 📌 Observacions

- La pàgina de bloqueig per defecte d’IPFire **ja mostra la URL denegada** (no cal configuració addicional).
- Per al bloqueig d’una URL concreta, és important escriure la URL completa (incloent `https://`) i desar-la a la secció corresponent.
- La llista de frases bloquejades utilitza **expressió regular** simple. Amb el terme `anime` ja n’hi ha prou per interceptar qualsevol domini o URL que contingui aquesta cadena.
- El bloqueig per hores s’aplica a nivell de **firewall**, no del proxy. Això permet tallar completament la connexió fora de l’horari permès.

## 📎 Captures de pantalla

Les imatges referenciades a la guia detallada (`img/1.png` a `img/23.png`) il·lustren cada pas de la configuració i els resultats obtinguts.

---

**Data de realització:** Maig 2026 (d’acord amb la v20250320 de l’enunciat)  
**Autor:** Estudi de l’activitat de proxy amb filtrat de contingut.