🔧 1) Skript vytvoří vše, co je potřeba pro migraci
Tvoje logika:

vytvoří CA (pokud není)

vytvoří server cert (pokud není)

podepíše

zkontroluje private-key (pokud existuje)

vyexportuje CA i server cert jako .p12 → kompletní záloha

vyexportuje klientské certy + klíče

vyexportuje .ovpn pro každého klienta

To znamená:

✔ Máš kompletní PKI zálohu
✔ Máš kompletní klientské konfigurace
✔ Máš vše, co potřebuješ pro obnovu serveru na jiném routeru
A to je přesně to, co jsi chtěl.

🛡️ 2) Skript už nic nepřepisuje, pokud to existuje
Tohle je klíčové — a ty to máš správně:

CA
rsc
if exists → skip + check private-key
SERVER cert
rsc
if exists → skip + check private-key
IP pool
rsc
if exists → update ranges
PPP profile
rsc
if exists → update addresses
OVPN server
rsc
if exists → update cert/profile/cipher
Klientské certy
rsc
if exists → skip
PPP secret
rsc
if exists → update password/profile
To znamená:

✔ Skript je idempotentní
✔ Skript nic nezničí
✔ Skript nic nepřegeneruje
✔ Skript jen opraví to, co má být správně
A to je přesně to, co chceš u produkčního skriptu.

📦 3) Ruční donastavení OVPN serveru ti skript nepřepíše
Ano — tohle je důležité.

Skript nastavuje jen:

certifikát

require-client-certificate=yes

cipher=aes256-cbc

default-profile

A to je vše.

Všechno ostatní (port, mode, auth, keepalive, netmask, IPv6, TLS-auth, push-routes, DNS, …) si můžeš nastavit ručně a skript ti to už nikdy nepřepíše.

To je přesně ta rovnováha:

✔ automatizace tam, kde to má být
✔ manuální nastavení tam, kde je to individuální
