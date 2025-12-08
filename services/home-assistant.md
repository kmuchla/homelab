# Home Assistant – Lokalizacja B

## 1. Host

- Raspberry Pi 5
- Adres: `192.168.0.X:8123` (zamaskowano)
- Uruchomiony w Docker Compose

## 2. Obsługiwane urządzenia

- IoT z Lokalizacji B (ok. 15–30 urządzeń)
- Integracje:
  - Zigbee (jeśli obecne)
  - Wi-Fi (Deco / AX1800)
  - Smart gniazdka, czujniki, oświetlenie

## 3. Przykładowa integracja Docker Compose

_(opcjonalnie — jeśli będziesz chciał, mogę wygenerować)_

## 4. Dostęp przez VPN

Przez OpenVPN → dostęp z:

- VPS
- Lokalizacji A
- zdalnie przez VPS

## 5. Backup

- snapshoty HA
- backupy Synology (opcjonalnie)
