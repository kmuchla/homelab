# Hardening — Wazuh (server i agenci)

Praktyczne wskazówki dla instalacji Wazuh (server + agenci) oraz integracji z homelab.

1. Separacja usług

- Uruchom serwer Wazuh na dedykowanym hoście/VM z ograniczonymi uprawnieniami.

2. TLS i uwierzytelnianie

- Włącz TLS dla komunikacji agenta z serwerem (wazuh-agent <-> manager).
- Używaj certyfikatów lub kluczy o odpowiedniej sile.

3. Zarządzanie agentami

- Rejestruj i autoryzuj agentów ręcznie lub przez automatyzację z audytem.
- Utrzymuj listę zaufanych agentów i procedury revoke.

4. Logi i retencja

- Ustaw polityki retencji logów; przechowuj dłuższy okres metadanych i krótszy pełnych logów.
- Agreguj logi z ważnych hostów (firewall, VPN, NAS).

5. Alerting i playbooks reagowania

- Skonfiguruj krytyczne reguły detekcji i testuj playbooki reagowania (np. odcinanie połączeń, blokady IP).

6. Monitoring stanu serwera

- Monitoruj wykorzystanie dysku i bazy danych (Elasticsearch / OpenSearch) oraz health Wazuh components.

7. Backup i odzyskiwanie

- Backup konfiguracji managera Wazuh i database (elastic indices) oraz kluczy certyfikatów.

8. Aktualizacje

- Testuj aktualizacje agenta i managera przed wdrożeniem do produkcji.
