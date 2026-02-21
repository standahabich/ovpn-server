# MikroTik OpenVPN Automation Script

## Overview
Tento skript automatizuje kompletní přípravu OpenVPN serveru na MikroTiku včetně:

- generování CA certifikátu
- generování serverového certifikátu
- generování klientských certifikátů
- exportu `.p12` záloh pro migraci
- exportu `.ovpn` konfiguračních souborů pro klienty
- vytvoření/úpravy IP poolu
- vytvoření/úpravy PPP profilu
- vytvoření/úpravy OVPN serveru
- vytvoření/úpravy PPP secretů

Skript je idempotentní — pokud něco existuje, nepřepisuje to, pouze opraví kritické parametry (certifikát, IP pool, profil).  
Je navržen tak, aby:

- jednorázově vytvořil vše potřebné
- vyexportoval zálohy
- vyexportoval klienty
- a poté už konfiguraci serveru nepřepisoval

Díky tomu je možné server dále ručně upravit bez rizika, že skript změny zničí.

---

## Naming konvence certifikátů
Skript používá jasnou a škálovatelnou strukturu názvů:

| Typ certifikátu | Formát Common Name |
|-----------------|--------------------|
| CA              | `ovpn_<serverName>_ca` |
| Server          | `ovpn_<serverName>_server` |
| Client          | `ovpn_<serverName>_client_<user>` |

Výhody:

- přehlednost
- snadné filtrování certifikátů
- možnost více OVPN serverů na jednom routeru
- čisté logování a správa

---

## Co skript dělá

### Certifikáty
- vytvoří CA certifikát (pokud neexistuje)
- vytvoří server certifikát (pokud neexistuje)
- vytvoří klientské certifikáty (pokud neexistují)
- podepíše je CA
- opraví názvy importovaných certifikátů
- ověří, že CA i server certifikát mají privátní klíč
- exportuje CA + server certifikát jako `.p12`
- exportuje klientské certy + klíče jako PEM

### OpenVPN server
- vytvoří nebo upraví IP pool
- vytvoří nebo upraví PPP profil
- vytvoří nebo upraví OVPN server (certifikát, cipher, require-client-certificate)
- vytvoří nebo upraví PPP secret pro každého klienta

### Export klientů
Pro každého klienta skript:

- exportuje certifikát a klíč
- vytvoří `.ovpn` konfigurační soubor
- pojmenuje ho jako `<serverAddress>_<clientName>.ovpn`
- uklidí dočasné soubory

---

## Co skript nedělá
- nepřepisuje ruční nastavení OVPN serveru (port, auth, TLS-auth, push-routes, DNS, atd.)
- nepřegenerovává existující certifikáty
- nemaže existující klienty
- nemění nic, co není explicitně v jeho správě

Po prvním spuštění můžeš server libovolně doladit ručně a skript ti to už nikdy nepřepíše.

---

## Migrace na jiný router

1. Spusť skript na původním routeru
2. Získej exportované `.p12` soubory (CA + server cert)
3. Získej `.ovpn` soubory pro klienty
4. Na novém routeru importuj `.p12`
5. Spusť skript znovu — automaticky:
   - opraví názvy certifikátů
   - ověří private-key
   - nastaví certifikát na OVPN server
   - vytvoří pool, profil, PPP secrets
6. Ručně nastav port, TLS-auth, push-routes, atd.

Hotovo — klienti se připojí bez změny konfigurace.

---

## Requirements
- RouterOS 7.x
- MikroTik s podporou OpenVPN serveru
- SSH/WinBox pro spuštění skriptu
- `.p12` exporty pro migraci (pokud přenášíš server)

---

## Notes
- Skript obsahuje ochranu proti paralelnímu běhu
- Všechny operace jsou idempotentní
- Naming konvence je navržena pro multi-server prostředí
- Exporty jsou ukládány do `/file`
