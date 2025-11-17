# Hardening — OpenVPN (serwer i klienci)

Ten dokument zawiera najlepsze praktyki dla serwera OpenVPN oraz klientów (bez ujawniania prywatnych certyfikatów/kluczy).

1. Architektura i separacja

- Utrzymuj OpenVPN w dedykowanym, minimalnym hostie (VPS) lub kontenerze.
- Ogranicz dostęp do paneli zarządzania hosta tylko z zaufanych adresów (np. SSH przez jump-host).

2. TLS / certyfikaty

- Używaj PKI: osobny CA, serwerowe i klientowskie certyfikaty.
- Ustaw długi okres życia CA, ale rotuj certyfikaty klientów częściej.
- Wykorzystaj silne ustawienia TLS (np. TLS 1.2/1.3, ECDSA lub RSA 3072+).
- Włącz TLS-auth / TLS-crypt (dla HMAC na pakietach lub pełne szyfrowanie nagłówków).

3. Konfiguracja serwera (przykład bez sekretów)

```
port 1194
proto udp
dev tun
server 10.8.0.0 255.255.255.0
keepalive 10 120
persist-key
persist-tun
tls-version-min 1.2
tls-crypt tls-crypt.key
cipher AES-256-GCM
auth SHA256
user nobody
group nogroup
status /var/log/openvpn-status.log
verb 3
```

4. Routing i bezpieczeństwo ruchu

- Wysyłaj do klientów tylko niezbędne trasy (push "route ..."), unikaj „push "route 0.0.0.0 0.0.0.0"” jeśli nie chcesz pełnego ruchu.
- Użyj firewall na hoście do ograniczenia ruchu z/na interfejs tun.
- Jeżeli łączysz kilka sieci przez VPN (hub-and-spoke), rozważ filtrowanie ruchu między spokes (aby nie umożliwiać lateral movement).

5. Revocation / CRL

- Wystaw CRL i sprawdzaj ją regularnie po revokacji klienta.
- Dokumentuj procedurę revoke + dystrybucji CRL.

6. Logowanie i monitoring

- Loguj zdarzenia (połączenia, rozłączenia, próby nieudane) i agreguj do SIEM (Wazuh).
- Monitoruj nietypowy ruch i sesje.

7. Automatyzacja i bezpieczeństwo kluczy

- Nie trzymać prywatnych kluczy w repo. Użyj bezpiecznego magazynu (Vault, sops) do przechowywania kluczy używanych do provisioning.
- Przygotuj skrypt rotacji certyfikatów/klienckich profili.

8. Dodatkowe rekomendacje

- Wprowadź MFA przy zarządzaniu serwerem (SSH + 2FA dla paneli).
- Ogranicz liczbę jednoczesnych połączeń na klienta jeśli to konieczne.

Referencje i narzędzia: `easy-rsa`, `openssl`, `ufw`/`nftables`, `fail2ban` (do ochrony przed bruteforce SSH).
