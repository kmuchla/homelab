# Network Overview

Dokument opisuje topologię sieci w dwóch lokalizacjach oraz połączenie z VPS poprzez tunel VPN. Sieć nie wykorzystuje VLANów – segmentacja opiera się głównie na funkcjach OPNsense w Lokalizacji A oraz na logicznym podziale urządzeń w Lokalizacji B.

## 1. Lokalizacja A

**Podsieć:** `192.168.100.0/24`
**Gateway / Firewall:** OPNsense
**DHCP:** OPNsense

### 1.1 Główne urządzenia

| Urządzenie | Adres         | Rola                            |
| ---------- | ------------- | ------------------------------- |
| OPNsense   | 192.168.10X.X | Gateway, firewall, DHCP, DNS FW |
| CCTV DVR   | 192.168.10X.x | Monitoring                      |
| IoT        | 192.168.10X.x | Lokalne urządzenia oraz IoT     |

Adresy `192.168.100.x` są przykładowe – w repo nie są publikowane pełne listy hostów ani dane wrażliwe.

### 1.2 Routing i dostęp

- Cały ruch internetowy z lokalnych urządzeń przechodzi przez OPNsense → WAN ISP.
- OPNsense zestawia tunel VPN do VPS:
  - tunel: `10.8.0.0/24` (OpenVPN),
  - ruch administracyjny do VPS (SSH/HTTP(S)) przechodzi przez tunel.

### 1.3 DNS i filtrowanie

- OPNsense może pełnić funkcję resolvera DNS (Unbound).
- Ruch DNS z urządzeń CCTV może być kierowany:
  - bezpośrednio do DNS ISP lub
  - do resolvera na OPNsense (w zależności od konfiguracji).

### 1.4 Dostęp administracyjny

- GUI OPNsense dostępne wyłącznie:
  - z sieci `192.168.100.0/24` lub
  - z zaufanych adresów przez tunel VPN.

---

## 2. Lokalizacja B

**Podsieć:** `192.168.0.0/24`
**Gateway:** router ISP (domowy)
**DHCP:** router ISP

Lokalizacja B to główne środowisko homelabu z usługami, monitoringiem oraz urządzeniami użytkowymi.

### 2.1 Główne urządzenia

| Urządzenie       | Adres       | Rola                               |
| ---------------- | ----------- | ---------------------------------- |
| Router ISP       | 192.168.0.1 | Gateway, NAT, DHCP                 |
| Synology NAS     | 192.168.0.x | Storage, backup                    |
| Raspberry Pi 5   | 192.168.0.x | Host dla kontenerów (Pi-hole, HA)  |
| Pi-hole (Docker) | 192.168.0.x | DNS filtering / ad-blocking        |
| Home Assistant   | 192.168.0.x | Home automation                    |
| Hyper-V Host     | 192.168.0.x | Hypervisor (Wazuh, Kali i inne VM) |
| VM Wazuh Server  | 192.168.0.x | SIEM/IDS dla homelabu              |
| VM Kali Linux    | 192.168.0.x | Lab/pentest                        |
| Windows PC (2x)  | 192.168.0.x | Stacje robocze z agentami Wazuh    |
| Drukarki (2x)    | 192.168.0.x | Urządzenia biurowe                 |
| Urządzenia IoT   | 192.168.0.x | Oświetlenie, smart urządzenia itp. |

### 2.2 Routing i dostęp

Lokalizacja B korzysta z tras statycznych, które kierują ruch do zdalnych podsieci poprzez główną bramę `192.168.0.XX`.
Dodatkowo w środowisku działa drugi, zapasowy klient OpenVPN na Synology (`192.168.0.XX`), który **nie uczestniczy w routingu**, ale może zostać ręcznie aktywowany jako alternatywne połączenie VPN.

#### Statyczne trasy skonfigurowane na routerze:

| Sieć docelowa    | Maska         | Brama        | Interfejs | Opis                               |
| ---------------- | ------------- | ------------ | --------- | ---------------------------------- |
| 192.168.10X.0/24 | 255.255.255.0 | 192.168.0.XX | LAN       | Dostęp do Lokalizacji A .10X       |
| 192.168.10X.0/24 | 255.255.255.0 | 192.168.0.XX | LAN       | Dostęp do Lokalizacji A .10X       |
| 192.168.10X.0/24 | 255.255.255.0 | 192.168.0.XX | LAN       | Dostęp do Lokalizacji A .10X       |
| 10.8.0.0/24      | 255.255.255.0 | 192.168.0.XX | LAN       | Zapasowy klient OpenVPN (Synology) |

#### Wyjaśnienie roli obu hostów

**192.168.0.10 – główna brama routingu**

- obsługuje cały ruch do zdalnych podsieci:
  `192.168.1XX/1XX/1XX/1XX`,
- umożliwia pełny dostęp z Lokalizacji B do Lokalizacji A,
- prowadzi ruch tunelowany przez główny klient VPN.

**192.168.0.2 – zapasowy VPN na Synology**

- działa jako _drugi, alternatywny klient OpenVPN_,
- **nie realizuje żadnych tras** i nie jest bramą dla ruchu sieciowego,
- jest gotowy do użycia jako awaryjny tunel w przypadku problemów z głównym połączeniem.

#### Przepływ ruchu (aktywny)

1. Urządzenie w `192.168.0.0/24` → router ISP
2. Router wg tras statycznych kieruje ruch zdalny → `192.168.0.XX`
3. `192.168.0.XX` → tunel VPN → VPS
4. VPS → tunel → OPNsense (Lokalizacja A)
5. OPNsense kieruje pakiety do urządzeń w `192.168.100.0/24`

#### Zapasowy VPN (nieaktywny w routingu)

- Synology (`192.168.0.XX`) utrzymuje _niezależne, alternatywne połączenie VPN_,
- nie przyjmuje ruchu z routera i nie prowadzi żadnych tras,
- może zostać włączony według potrzeb jako redundantne połączenie do VPS.

---

### 2.3 DNS

- Wszystkie urządzenia w Lokalizacji B korzystają z Pi-hole jako **głównego i jedynego** serwera DNS.
- Router ISP przekazuje zapytania DNS do Pi-hole lub wymusza jego użycie poprzez konfigurację DHCP.
- Pi-hole pełni funkcję:
  - filtracji DNS (blokowanie reklam, trackerów, złośliwych domen),
  - lokalnego resolvera dla urządzeń w 192.168.0.0/24,
  - źródła statystyk zapytań DNS dla całej lokalizacji.
- Dla urządzeń IoT, komputerów, serwerów i VM – Pi-hole jest zawsze serwerem DNS zdefiniowanym w ustawieniach DHCP.

### 2.4 Monitoring i bezpieczeństwo

- Centralnym systemem monitoringu jest **Wazuh Server** uruchomiony jako VM na Hyper-V w Lokalizacji B.
- Wazuh zbiera logi z agentów zainstalowanych na:
  - Windows PC (2 urządzenia),
  - Linux (VM),
  - Raspberry Pi 5 (Pi-hole / Home Assistant),
  - VPS (Ubuntu Server),
  - OPNsense w Lokalizacji A.

---

## 3. VPS

**Public IP:** `XXX.XXX.XXX.XXX`
**Tunel VPN:** `10.8.0.0/24` (OpenVPN)
**System:** Ubuntu Server
**Usługi:**

- OpenVPN Server – centralny punkt połączeń VPN,
- Apache2 – hosting wielu vhostów (kilka domen),
- UFW – firewall z ograniczeniem dostępu do portów 80/443 wyłącznie z IP Cloudflare,

### 3.1 Rola w topologii

- Węzeł centralny dla tuneli VPN z lokalizacji A oraz B.
- Serwer www za Cloudflare (reverse proxy/CDN).
- Punkt pośredni dla ruchu między lokalizacjami:
  - Lokalizacja B → VPS → OPNsense (Lokalizacja A).

---

## 4. Sieć VPN

**Technologia:** OpenVPN
**Tryb:** site-to-site / remote access
**Podsieć tunelu:** `10.8.0.0/24`
**Port:** 1194/udp
