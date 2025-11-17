# Overview — Homelab

Krótki przegląd architektury oraz celów tego repozytorium.

Cel repozytorium

- Dokumentacja prywatnego homelabu i konfiguracji sieciowych.
- Materiał demonstracyjny do portfolio (pierwsza wersja, PL).

Główne komponenty

- Lokalizacja A: OPNsense (firewall/router), VLANy, monitoring, CCTV.
- Lokalizacja B: NAS (Synology), Hyper-V host, Raspberry Pi (Pi-hole, Home Assistant).
- VPS (OVH): OpenVPN server (hub), serwisy WWW (Apache) i agent Wazuh.

Jak korzystać z repo

- Zacznij od `docs/overview.md`, następnie `docs/network/topology.md` i `docs/network/ip-plan.md`.
- Konfiguracje usług znajdują się w `docs/services/`.
- Checklisty bezpieczeństwa i hardening w `docs/security/`.

Uwaga

- Repo zawiera jedynie dokumenty i przykładowe pliki konfiguracyjne bez sekretów.
- Wersja początkowa — sugerowane dalsze usprawnienia: inventory (Ansible/NetBox), edytowalne diagramy (Mermaid/draw.io), CI (linting dokumentacji).
