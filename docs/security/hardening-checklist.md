# Hardening checklist — Homelab (wersja rozszerzona)

Zbiorcza lista kontrolna oraz odnośniki do szczegółowych przewodników dla poszczególnych usług.

Linki do przewodników szczegółowych:

- `docs/security/hardening-openvpn.md` — OpenVPN (serwer i klienci)
- `docs/security/hardening-opnsense.md` — OPNsense / firewall
- `docs/security/hardening-pihole.md` — Pi-hole (Docker)
- `docs/security/hardening-home-assistant.md` — Home Assistant
- `docs/security/hardening-synology.md` — Synology DSM
- `docs/security/hardening-wazuh.md` — Wazuh (SIEM)

Główne kroki (szybkie przypomnienie):

1. Aktualizacje
   - Regularne aktualizacje systemów (OS, firmware routerów, NAS, kontenery). Automatyzuj tam, gdzie to bezpieczne.
2. Backup i odzyskiwanie
   - Regularne backupy konfiguracji i danych (zaszyfrowane, z retencją i testami odtwarzania).
3. SSH
   - Wyłącz logowanie hasłem, używaj kluczy publicznych.
   - Wyłącz `root` login przez SSH.
   - Ogranicz dostęp do adresów IP lub wymuś jump-host / bastion.
4. VPN
   - Używaj certyfikatów klienta i serwera; utrzymuj CRL/OCSP i rotację certyfikatów.
   - Minimalne uprawnienia routingu (split-tunnel tylko jeśli konieczne).
5. Firewall
   - Zasada najmniejszych uprawnień (deny by default), logowanie incydentów.
   - Separacja VLAN dla IoT/CCTV/serwerów.
6. Usługi webowe
   - Reverse proxy z TLS, HSTS, silnymi cipherami i ograniczeniem dostępu (IP allowlist / auth).
   - Wymagaj uwierzytelnienia dla paneli (2FA tam, gdzie możliwe).
7. Monitoring i SIEM
   - Centralne logowanie (Wazuh), alerty i retencja logów; monitorowanie integrity.
8. Zarządzanie sekretami
   - Nie trzymać haseł i kluczy w repo. Używaj `sops`/`ansible-vault`/secrets manager.
9. Dostępy i konta
   - Zasada najmniejszego uprzywilejowania, audyt kont i rotacja haseł/kluczy.
10. Testy i audyty

- Regularne skany (vuln scan), audyty konfiguracji, testy odzyskiwania.

Następne kroki: szczegółowe instrukcje dla każdej usługi znajdują się w zamieszczonych plikach. Mogę przygotować Ansible playbooki i szablony konfiguracyjne po Twoim potwierdzeniu.
