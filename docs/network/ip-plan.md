# Plan adresacji IP – Homelab

## 1. Przegląd

Wszystkie lokalne sieci w homelabie mieszczą się w przestrzeni adresowej `192.168.0.0/16`, z podziałem na /24 według lokalizacji i VLAN-ów.

## 2. Lokalizacja A

### 2.1. Sieci

- `192.168.100.0/24` – główna sieć LAN
  - komputery użytkowe
  - drukarka
  - IoT (~30 urządzeń)
  - UPS
- `192.168.102.0/24` – CCTV
  - 2× rejestrator NVR
  - kamery IP

### 2.2. Sieci zarezerwowane / planowane

- `192.168.101.0/24` – VLAN (zarezerwowany)
- `192.168.103.0/24` – VLAN (zarezerwowany)
- `192.168.104.0/24` – VLAN (zarezerwowany)
- `192.168.105.0/24` – VLAN (zarezerwowany)

(widoczne w konfiguracji OpenVPN i tablicy routingu VPS)

## 3. Lokalizacja B

### 3.1. Sieci

- `192.168.0.0/24` – główna sieć LAN
  - NAS Synology `192.168.0.X` (zamaskowano)
  - Hyper-V host `192.168.0.X` + VM Linux `192.168.0.X` (Wazuh Server) (zamaskowano)
  - Raspberry Pi `192.168.0.X` (Pi-hole, Home Assistant, OpenVPN client) (zamaskowano)
  - drukarki (`192.168.0.X`) (zamaskowano)
  - monitoring `192.168.0.X` (zamaskowano)
  - routery podrzędne, switche, IoT
- `172.17.0.0/16` – sieć Docker na VPS (wewnętrzna, nie routowana przez VPN)

## 4. VPS

- Publiczna sieć:
  - IP: `57.128.226.X` (pełny adres usunięty z publicznej dokumentacji)
  - brama: `57.128.226.X` (zamaskowano)
- Sieć tunelu VPN:
  - `10.8.0.0/24`
  - VPS: `10.8.0.X` (zamaskowano)
  - Lokalizacja B (`zielona24b`): `10.8.0.X` (zamaskowano)
  - Lokalizacja A (`opnsense`): `10.8.0.X` (zamaskowano)

## 5. Konwencje

- Adresy z końcówką `.1` – zwykle gateway dla danego VLAN / podsieci.
- Adresy serwerów kluczowych:
  - NAS: `192.168.0.X` (zamaskowano)
  - Hyper-V host: `192.168.0.X` (zamaskowano)
  - Wazuh Server VM: `192.168.0.X` (zamaskowano)
  - Raspberry Pi (Pi-hole + HA + OpenVPN): `192.168.0.X` (zamaskowano)
- Adresacja IoT, drukarek i routerów podrzędnych dokumentowana w osobnym pliku, jeśli będzie potrzebne szczegółowe inventory.
