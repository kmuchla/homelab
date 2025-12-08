# Pi-hole – Lokalizacja B

## 1. Host

- Raspberry Pi 5
  -- Adres: `192.168.0.X` (zamaskowano)
- Docker + Docker Compose

## 2. Usługi

- Pi-hole (DNS + filtr reklam)
  -- Dostęp przez panel: `http://192.168.0.X/admin` (zamaskowano)

## 3. Konfiguracja sieci

- Pi-hole działa jako DNS **tylko dla Lokalizacji B**
  -- AX1800 router: wpisane DNS = IP RPi5 (`192.168.0.X`) (zamaskowano)
- Upstream DNS:
  - Cloudflare (1.1.1.1)
  - DNSSEC: włączony

## 4. Integracja

- Home Assistant (dla detekcji urządzeń)
- OpenVPN — widoczność z Lokalizacji A

## 5. Logi / monitoring

- `docker logs pihole`
- dzienny log DNS: `/etc/pihole/pihole.log`
