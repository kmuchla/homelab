# Homelab

Dokumentacja techniczna środowiska homelab obejmująca dwie lokalizacje, centralny VPS, tunel VPN oraz zestaw usług działających lokalnie i w chmurze. Repozytorium zawiera schematy, konfiguracje i opisy architektury sieci, bezpieczeństwa oraz usług.

---

## 1. Architektura

Środowisko składa się z trzech głównych części:

- **Lokalizacja A (192.168.1XX.0/24)**
  Firewall OPNsense, system CCTV, urządzenia IoT. Połączenie z VPS odbywa się przez tunel OpenVPN.

- **Lokalizacja B (192.168.0.0/24)**
  Główna lokalizacja homelabu z NAS, Hyper-V, Wazuh Server, Raspberry Pi 5 (Pi-hole, Home Assistant), komputerami, drukarkami i IoT. Statyczne trasy umożliwiają pełny dostęp do Lokalizacji A oraz do pozostałych podsieci.

- **VPS**
  Publiczny serwer z OpenVPN Server, Apache2 (wiele domen za **Cloudflare**), UFW oraz Wazuh Agent. Pełni funkcję centralnego VPN.

  ![Cloudflare](https://img.shields.io/badge/Cloudflare-F38020?logo=cloudflare&logoColor=white) wykorzystane usługi:

  - [Zero Trust Access](cloudflare/zero-trust.md),
  - [Cloudflare Tunnel](cloudflare/zero-trust.md),
  - [DNS Hosting](cloudflare/dns-setup.md),
  - [Universal SSL (Edge Certificates)](cloudflare/dns-setup.md#ssltls),
  - [Page Rules / Redirect Rules](cloudflare/dns-setup.md#page-rules),
  - [Cloudflare Turnstile](cloudflare/waf-concepts.md#4-turnstile),
  - [Proxy HTTPS (DNS proxied – orange cloud)](cloudflare/dns-setup.md#rekordy).

---

## 2. Sieć i routing

### 2.1 Podsieci

| Lokalizacja | Podsieć          | Gateway    |
| ----------- | ---------------- | ---------- |
| A           | 192.168.1XX.0/24 | OPNsense   |
| B           | 192.168.0.0/24   | Router ISP |
| VPN         | 10.8.0.0/24      | VPS        |

---

### 2.2 VPN

- Technologia: **OpenVPN (UDP 1194)**
- Architektura: **hub-and-spoke** z centralnym węzłem na VPS
- Sieć tunelu: **10.8.0.0/24**

OPNsense w Lokalizacji A oraz urządzenia w Lokalizacji B zestawiają połączenie z VPS poprzez tunel VPN. Dzięki statycznym trasom na routerze w Lokalizacji B, **wszystkie urządzenia w sieci 192.168.0.0/24 mają pełny dostęp do zasobów osiągalnych przez VPN**, w tym:

- urządzeń i CCTV w Lokalizacji A,
- pozostałych podsieci (100/1XX/1XX/1XX),
- usług i serwerów dostępnych na VPS,
- wszystkich hostów routowanych przez tunel.

Drugi tunel VPN na Synology (`192.168.0.X`) działa jako połączenie zapasowe i nie uczestniczy w routingu.

---

## 3. DNS

- Wszystkie urządzenia w Lokalizacji B korzystają z **Pi-hole jako głównego serwera DNS**.
- Pi-hole pełni funkcję:
  - filtracji DNS (blokowanie reklam i złośliwych domen),
  - lokalnego resolvera,
  - źródła statystyk zapytań DNS.
- OPNsense w Lokalizacji A może pracować jako resolver Unbound.

---

## 4. Monitoring i bezpieczeństwo

### 4.1 Monitoring (Wazuh)

W Lokalizacji B działa centralny **Wazuh Server** jako VM na Hyper-V.
Zbiera dane z agentów zainstalowanych na:

- Windows PC (2),
- Linux (host/VM w Lokalizacji B),
- Raspberry Pi 5,
- VPS,
- OPNsense.

Zakres monitoringu:

- logi systemowe,
- zmiany konfiguracji,
- logi zapór (UFW, OPNsense firewall),
- aktywność procesów,
- wykrywanie anomalii i incydentów.

### 4.2 Zabezpieczenia

- Ruch między lokalizacjami przechodzi wyłącznie przez VPN.
- VPS:
  - UFW ograniczone do IP Cloudflare na portach 80/443,
  - brak otwartych portów administracyjnych na WAN.
- OPNsense:
  - firewall, filtrowanie, brak ekspozycji usług do internetu.
- Usługi WWW są chronione przez Cloudflare.

---

## 5. Usługi

### 5.1 Web hosting (VPS)

- Apache2 z wieloma vhostami,
- domeny obsługiwane przez Cloudflare (DNS, SSL, page rules, zabezpieczenia).

### 5.2 Home automation (Lokalizacja B)

- Raspberry Pi 5 uruchamia:
  - Pi-hole (DNS filtering),
  - Home Assistant (automatyzacje domowe).

### 5.3 Monitoring

- Wazuh (SIEM/IDS),
- Pi-hole (DNS),
- logi OPNsense,
- logi VPS (systemd, UFW, Apache).

### 5.4 Security

- OpenVPN,
- firewall OPNsense,
- UFW,
- segmentacja logiczna,
- Cloudflare (origin protection).
