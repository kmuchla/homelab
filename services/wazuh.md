# Wazuh – SIEM

## 1. Architektura

### Serwer Wazuh

- Host: VM Linux na Hyper-V
- Adres: `192.168.0.X` (zamaskowano)
- Lokalizacja: B

### Agenci

- VPS (OpenStack) – wazuh-agent
- Windows / Linux w obu lokalizacjach (do rozbudowy)

## 2. Dostęp

- UI przez tunel (HTTPS)
- Porty: 55000, 55002

## 3. Funkcje

- log collection
- file integrity monitoring
- detection rules
- integracja z NAS, systemami w sieci

## 4. Monitoring VPS

- wazuh-agent.service aktywny
- logi systemowe i OpenVPN

## 5. TODO

- integracja z OPNsense (API + Wazuh module)
