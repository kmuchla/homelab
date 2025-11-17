# Hardening — Synology DSM

Lista praktyk zabezpieczających serwery Synology (DSM).

1. Aktualizacje i pakiety

- Aktualizuj DSM i pakiety regularnie. Włącz automatyczne powiadomienia.

2. Konta i uprawnienia

- Używaj kont z minimalnymi uprawnieniami.
- Wyłącz konto `admin` (użyj innego konta z uprawnieniami sudo).
- Włącz 2FA dla kont administracyjnych.

3. Sieć

- Włącz firewall w DSM i ogranicz dostęp do usług tylko do zaufanych źródeł.
- Użyj VPN do zdalnego zarządzania zamiast wystawiania portów.

4. Folders & Services

- Ogranicz dzielenie zasobów SMB/NFS do grup i użytkowników.
- Włącz dostęp tylko po SMB v2+ i wymuś silne szyfrowanie.

5. Snapshoty i backup

- Włącz snapshoty Btrfs (jeśli dostępne) i zewnętrzne backupy zaszyfrowane.

6. Audyt i logi

- Włącz i centralizuj logi; integruj z Wazuh lub syslog server.

7. Aplikacje kontenerowe

- Jeżeli uruchamiasz kontenery (Docker/Container Manager), ogranicz uprawnienia i używaj sieci izolowanych.

8. Dodatkowe ustawienia

- Włącz Auto Block / Account Protection opcje.
- Skonfiguruj limity sesji i timeouty dla GUI.
