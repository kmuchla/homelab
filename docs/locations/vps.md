# VPS

## 1. Ogólny opis

- Provider: OVH (OpenStack Nova, AS16276 OVH SAS)
- Platforma: OpenStack (OpenStack Nova / OpenStack Foundation)
- Rola: Węzeł centralny (hub) dla VPN + hosting stron WWW
- Nazwa hosta: `vps-72224b55`

## 2. System operacyjny

- Dystrybucja: Ubuntu 25.04 "plucky"
- Kernel: Linux 6.14.0-35-generic (x86_64)
- Usługi systemowe (wybrane, aktywne):
  - `apache2.service` – Apache HTTP Server
  - `openvpn-server@server.service` – serwer OpenVPN
  - `mariadb.service` – MariaDB 11.4.7
  - `docker.service` + `containerd.service` – Docker / container runtime
  - `ssh.service` – serwer SSH
  - `wazuh-agent.service` – agent Wazuh

## 3. Sieć i adresacja

- Interfejs publiczny: `ens3`
- Publiczny adres IP: `57.128.226.X`
  (pełny adres: `57.128.226.10` – w dokumentacji maskowany do X)
- Brama domyślna: `57.128.226.1`

Routing (istotne wpisy):

- `default via 57.128.226.1 dev ens3`
- `10.8.0.0/24 dev tun0` – sieć tunelu OpenVPN
- `192.168.0.0/16 via 10.8.0.2 dev tun0` – ruch do sieci prywatnych przez klienta `zielona24b`
- `172.17.0.0/16 dev docker0` – sieć Docker

## 4. Usługi WWW (Apache)

Apache nasłuchuje na portach:

- HTTP: `*:80`
- HTTPS: `*:443`

### 4.1. VirtualHost – HTTPS (443)

- `57.128.226.10` – vhost domyślny (`000-ip-catchall.conf`)
- `kamil.muchla.pl` (`kamil-le-ssl.conf`)
- `msmistal.com` (`msmistal-le-ssl.conf`)
  - aliasy: `www.msmistal.com`, `msmistal.com.pl`, `www.msmistal.com.pl`,
    `msmistal.pl`, `www.msmistal.pl`, `msmistal.eu`, `www.msmistal.eu`
- `ventabe-bhp.pl` (`redirect-le-ssl.conf`)
  - aliasy: `www.ventabe-bhp.pl`, `ventabe.pl`, `www.ventabe.pl`
- `tischlerei-meblopank.com` (`tischlerei-com-le-ssl.conf`)
- `tischlerei-meblopank.pl` (`tischlerei-le-ssl.conf`)
- `ventabe.com` (`wordpress-le-ssl.conf`)
  - alias: `www.ventabe.com`

### 4.2. VirtualHost – HTTP (80)

- `default-only` (`000-default.conf`) – vhost domyślny
- `57.128.226.10` (`000-ip-catchall.conf`)
- `kamil.muchla.pl` (`kamil.conf`)
- `msmistal.com` (`msmistal.conf`) + te same aliasy, co na 443
- `ventabe-bhp.pl` (`redirect.conf`) + aliasy
- `tischlerei-meblopank.com` (`tischlerei-com.conf`)
- `tischlerei-meblopank.pl` (`tischlerei.conf`)
- `ventabe.com` (`wordpress.conf`)

## 5. OpenVPN – konfiguracja serwera

Interfejs tunelu:

- `tun0` – adres: `10.8.0.1/24`

Fragment konfiguracji serwera (`/etc/openvpn/server/server.conf`):

```text
server 10.8.0.0 255.255.255.0

push "route 192.168.100.0 255.255.255.0"
push "route 192.168.101.0 255.255.255.0"
push "route 192.168.102.0 255.255.255.0"
push "route 192.168.105.0 255.255.255.0"
push "route 192.168.0.0 255.255.255.0"
```
Stan serwera (/var/log/openvpn-status.log – uproszczony):
Klienci:
opnsense – Lokalizacja A – adres tunelu: 10.8.0.3
zielona24b – Lokalizacja B – adres tunelu: 10.8.0.2
Sieci za klientami (ROUTING_TABLE):
za opnsense:
192.168.100.0/24
192.168.101.0/24
192.168.102.0/24
192.168.103.0/24
192.168.104.0/24
192.168.105.0/24
za zielona24b:
192.168.0.0/24
hosty w tej podsieci (NAS, VM, itp.)
6. Wazuh agent
Na VPS działa wazuh-agent.service.
Rola:
monitorowanie logów systemowych
integracja z centralnym serwerem Wazuh w Lokalizacji B



