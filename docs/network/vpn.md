# Architektura VPN

## 1. Model połączeń

Topologia: **hub-and-spoke**

- **Hub**: VPS (OVH, Ubuntu 25.04, OpenVPN server)
- **Spoke 1**: Lokalizacja A (OPNsense, klient `opnsense`)
- **Spoke 2**: Lokalizacja B (klient `zielona24b`, na Raspberry Pi / routerze)

Tunel oparty o OpenVPN (tunel warstwy 3, TUN).

## 2. Sieć tunelu

- Sieć tunelu: `10.8.0.0/24`
- Adresy:
  - VPS (serwer): `10.8.0.1`
  - Lokalizacja B (`zielona24b`): `10.8.0.2`
  - Lokalizacja A (`opnsense`): `10.8.0.3`

## 3. Sieci prywatne za każdym węzłem

### 3.1. VPS

- brak dodatkowych sieci prywatnych – pełni rolę routera/huba
- posiada trasę:
  - `192.168.0.0/16 via 10.8.0.2 dev tun0`

### 3.2. Lokalizacja A (OPNsense – `opnsense`)

Sieci ogłaszane przez OpenVPN (zgodnie ze status log):

- `192.168.100.0/24` – główna sieć LAN
- `192.168.101.0/24` – zarezerwowana (np. przyszłe VLAN)
- `192.168.102.0/24` – CCTV
- `192.168.103.0/24` – zarezerwowana / przyszłe użycie
- `192.168.104.0/24` – zarezerwowana / przyszłe użycie
- `192.168.105.0/24` – zarezerwowana / przyszłe użycie

### 3.3. Lokalizacja B (`zielona24b`)

Sieci ogłaszane przez OpenVPN:

- `192.168.0.0/24` – główna sieć LAN (NAS, Hyper-V, RPi, drukarki, monitoring, IoT)

## 4. Routing z perspektywy VPS

Wybrane wpisy z tablicy routingu:

- `10.8.0.0/24 dev tun0` – sieć tunelu
- `192.168.0.0/16 via 10.8.0.2 dev tun0` – domyślnie cała przestrzeń 192.168.x.x kierowana do Lokalizacji B
- Dodatkowo OpenVPN obsługuje sieci z Lokalizacji A na poziomie wewnętrznej tablicy routingu serwera.

W praktyce:

- Ruch z internetu → VPS → (po filtrach firewall) → do `192.168.100.0/24` lub `192.168.0.0/24`
- Ruch między Lokalizacją A i B:
  - A → tunel → VPS → tunel → B
  - B → tunel → VPS → tunel → A

## 5. Zastosowania tunelu

- zdalny dostęp do:
  - NAS Synology (`192.168.0.2`)
  - Wazuh Server (`192.168.0.7`)
  - urządzeń w Lokalizacji A (`192.168.100.0/24`, `192.168.102.0/24`, itp.)
- centralny punkt ekspozycji usług (VPS z publicznym IP)
- monitoring bezpieczeństwa (Wazuh agent na VPS)
