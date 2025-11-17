# Lokalizacja A

## 1. Charakterystyka ogólna

- Typ: Dom (Lokalizacja A)
- Rola w homelabie: Sieć użytkowa + IoT + monitoring + dostęp zdalny przez VPN
- Dostęp między lokalizacjami: OpenVPN klient → VPS (centrala)
- Brzegowy router/firewall: Protectli VP4650 z OPNsense

---

## 2. WAN / Internet

- Dostawca: światłowód (PPPoE)
- Urządzenie operatora: ONT w trybie bridge
- OPNsense zestawia PPPoE (dynamiczne publiczne IP)
- Zdalny dostęp do sieci realizowany przez OpenVPN → VPS

---

## 3. Interfejsy OPNsense

Konfiguracja z panelu OPNsense:

| Interfejs | Nazwa systemowa | Fizyczne urządzenie | Przeznaczenie                                                    |
| --------- | --------------- | ------------------- | ---------------------------------------------------------------- |
| LAN       | lan             | igc0                | Sieć LAN `192.168.100.0/24`                                      |
| WAN       | wan             | pppoe0 (igc1)       | Połączenie z ISP (PPPoE)                                         |
| ovh       | opt3            | ovpnc2              | OpenVPN klient (tunel do VPS)                                    |
| switch    | opt4            | igc4                | Uplink do switcha / okablowania strukturalnego (gniazdka w domu) |
| x95       | opt2            | igc3                | Połączenie do TP-Link Deco (AP mode)                             |

---

## 4. Sieci / VLAN

### 4.1. LAN (główna)

- Adresacja: `192.168.100.0/24`
- DHCP: OPNsense
- Typ ruchu: komputery, IoT, drukarka, UPS

### 4.2. Monitoring CCTV

- Adresacja: `192.168.102.0/24`
- Urządzenia: 2× NVR + kamery IP
- Uzasadnienie: separacja ruchu wideo od LAN
- Uwaga: VLAN/oddzielna sieć — zgodnie z konfiguracją przełącznika

(Jeśli VLAN tag jest znany — dopiszemy w sekcji ip-plan.)

---

## 5. Wi-Fi

- System: TP-Link Deco
- Tryb: Access Point
- DHCP & routing: wyłącznie OPNsense
- Interfejs połączeniowy: `opt2 (x95)`
- Przeznaczenie:
  - sieć użytkowa
  - IoT (ok. 30 urządzeń)

---

## 6. Urządzenia końcowe

- 2× komputery użytkowe
- 1× drukarka
- ok. 30× urządzeń IoT (sensory, oświetlenie, gniazda, AGD smart)
- UPS (zarządzany sieciowo)
- 2× NVR + kamery IP (w sieci `192.168.102.0/24`)

---

## 7. VPN – połączenie z VPS

- Typ: OpenVPN
- Rola OPNsense: klient (`ovpnc2`)
- Cele:
  - stały dostęp z zewnątrz (VPS jako „hub” z publicznym adresem)
  - routowanie ruchu między Lokalizacją A, Lokalizacją B i VPS

### Sieci prawdopodobnie routowane:

> Zaktualizujemy po weryfikacji konfiga VPS i OPNsense.

- `192.168.100.0/24` (LAN A)
- `192.168.102.0/24` (CCTV — jeśli dodana do tunelu)
- tunel: `10.8.0.0/24` (typowe dla OpenVPN) — szczegóły punktów końcowych zostały zamaskowane w dokumentacji publicznej
