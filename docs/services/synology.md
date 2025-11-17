# Synology DS224+

## 1. Specyfikacja

- Adres: `192.168.0.X` (zamaskowano)
- Dostępna przez VPN (A ↔ B ↔ VPS)

## 2. Usługi

- File Services (SMB, NFS)
- Virtual Machine Manager
  - Kali Linux VM – `192.168.0.X` (zamaskowano)
- Backup:
  - Time Machine (MacBook)
  - HA snapshots (opcjonalnie)
- OpenVPN Client (drugi tunel)

## 3. Integracje

- widoczny z obu lokalizacji
- backupy IoT / HA / systemów
- miejsce pod logi Wazuh (opcjonalnie)

## 4. TODO

- dokumentacja przekierowań portów (AX1800)
- plan podłączenia Synology do VLAN-ów (jeśli pojawią się w B)
