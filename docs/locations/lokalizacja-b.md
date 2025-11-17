# Lokalizacja B

## 1. Charakterystyka ogólna

- Typ: Dom (Lokalizacja B)
- Rola: Sieć użytkowa, IoT, serwery, NAS, wirtualizacja
- Główna podsieć: `192.168.0.0/24`
- Połączenie z pozostałymi lokalizacjami: OpenVPN → VPS (piwnica – Raspberry Pi)

---

## 2. Topologia WAN / Routery

Lokalizacja B zawiera **dwa poziomy routerów**:

### 2.1. Router dostawcy internetu (Poziom 1)

- Adres: `192.168.1.1`
- Tryb: router NAT
- Konfiguracja DMZ: cały ruch przekazywany do routera użytkownika (poziom 2)

### 2.2. Główny router użytkownika – AX1800 Wi-Fi 6 (Poziom 2)

- DHCP: **TAK** (główna podsieć `192.168.0.0/24`)
- Rola: centrum dystrybucji sieci kablowej i Wi-Fi
- Do routera podpięte są 5 fizycznych przewodów, z czego znane są cztery:

| Port | Lokalizacja   | Końcowe urządzenia                  | Adresy / uwagi                                                                   |
| ---- | ------------- | ----------------------------------- | -------------------------------------------------------------------------------- |
| 1    | Salon         | Deco X20 (mesh) + Deco X20 (piętro) | `.199`, `.198`                                                                   |
| 2    | Pokój Szymona | Switch (adres nieustalony)          | urządzenia IoT                                                                   |
| 3    | Piwnica       | Router (adres nieustalony)          | NAS + Raspberry Pi                                                               |
| 4    | Biuro         | Router Archer C7 `192.168.0.16`     | drukarka `.3`, monitoring `.5`, kolejny router (IP?) + druga drukarka `.123`, PC |
| 5    | Nieustalony   | wymaga weryfikacji                  | -                                                                                |

---

## 3. Sieć bezprzewodowa

### 3.1. Deco X20 (mesh)

- Tryb: Access Point
- Adresy:
  - Parter: `192.168.0.199`
  - Piętro: `192.168.0.198`
- DHCP: wyłączony
- SSID: takie samo jak router główny (jedna domena broadcastowa)

### 3.2. Router w biurze (Archer C7)

- Adres: `192.168.0.16`
- Funkcja: dodatkowy AP + switch
- Urządzenia:
  - Drukarka `192.168.0.3`
  - Rejestrator monitoringu `192.168.0.5`
  - Kolejny router (adres nieznany)
    - Drukarka `192.168.0.123`
    - PC (brak adresu)

---

## 4. Hosty i wirtualizacja

### 4.1. Komputer Windows 11 (192.168.0.4)

- Rola: host Hyper-V
- VM:
  - Linux (192.168.0.7)
  - Zainstalowane usługi:
    - **Wazuh Server**

### 4.2. Raspberry Pi 5 (192.168.0.10)

- Docker + Docker Compose
- Usługi:
  - **Pi-hole** (`192.168.0.10/admin/login`) — lokalny DNS dla Lokalizacji B
  - **Home Assistant** (`192.168.0.10:8123`) — obsługa IoT lokalnych
- Dodatkowe oprogramowanie:
  - Klient **OpenVPN** → VPS (tunel między lokalizacjami)
- Rola pomocnicza:
  - Punkt integracji IoT + DNS + tunel VPN

### 4.3. Synology DS224+ (192.168.0.2)

- Dostęp z obu lokalizacji (routing do doprecyzowania)
- Użytkownicy + udziały plików
- Funkcje:
  - **Virtual Machine Manager**:
    - Kali Linux VM (`192.168.0.151`)
  - **Backup Time Machine** (MacBook)
  - **Systemowy serwer OpenVPN** (zapasowy)
    - Port forwarding skonfigurowany na Routerze z sekcji 2.2.

---

## 5. Monitoring

- Rejestrator główny: `192.168.0.5`
- Drugi monitoring za kolejnym routerem (drukarka `.123`, PC)
- Wszystko działa w jednym segmencie `192.168.0.0/24`

---

## 6. IoT

- Lokalizacja: rozproszone (Piwnica, Pokój Szymona, Biuro)
- Adresacja: `192.168.0.0/24`
- Liczba urządzeń: kilkanaście (szacowane: 15–30)
- Integracja: Home Assistant na Raspberry Pi

---

## 7. VPN – połączenie do VPS

- Klient VPN: Raspberry Pi (`openvpn-client`)
- Tunel: `10.8.x.x` (do potwierdzenia)
- Cel:
  - Połączenie z Lokalizacją A
  - Dostęp do NAS z obu lokalizacji
  - Zdalny dostęp do usług w LAN B przez VPS

---

## 8. Elementy wymagające weryfikacji (TODO)

1. Adres IP **switcha w Pokoju Szymona**
2. Adres IP **routera w Piwnicy**
3. Adres IP **routera w drugim biurze**
4. Adres IP urządzenia z gniazda nr 5 w routerze AX1800
5. Zakresy sieci tunelowej OpenVPN między VPS ↔ Lok B
