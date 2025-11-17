# Hardening — Home Assistant

Zasady zabezpieczenia instalacji Home Assistant (HA) uruchamianej w Docker/Compose na Raspberry Pi.

1. Architektura dostępu

- Nigdy nie wystawiaj panelu HA bezpośrednio do internetu.
- Użyj reverse-proxy (np. Nginx, Traefik) z TLS i opcjonalnym dodatkowym uwierzytelnianiem (OAuth2 proxy).

2. Uwierzytelnianie

- Włącz MFA dla kont użytkowników HA (dostępne w integracjach lub przez proxy).
- Ustal silne hasła i ogranicz liczbę kont o podwyższonych uprawnieniach.

3. Docker Compose — wskazówki

- Używaj dedykowanego wolumenu dla konfiguracji i backupu.
- Trzymaj plik `.env` poza repo i ignoruj go w `.gitignore`.

4. Integracje i dodatki

- Review integracje i usuń nieużywane.
- Ogranicz integracje zdalne (zewnętrzne webhooki) do whitelist.

5. Backup i odtwarzanie

- Regularne snapshoty konfiguracji HA i backupy na Synology (zaszyfrowane).

6. Monitoring i alerty

- Integruj HA z systemem monitoringu (Wazuh/Prometheus) do wykrywania nieautoryzowanych zmian.

7. Automatyczne aktualizacje

- Testuj aktualizacje na środowisku staging przed wdrożeniem w produkcji.

8. Example Nginx reverse-proxy (minimalny)

```
server {
  listen 443 ssl;
  server_name ha.example.com;
  ssl_certificate /etc/ssl/certs/fullchain.pem;
  ssl_certificate_key /etc/ssl/private/privkey.pem;

  location / {
    proxy_pass http://192.168.0.X:8123/; # wewnętrzny adres HA (zamaskowano)
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```

Uwaga: zamiast tylko TLS rozważ dodatkowe warstwy (VPN lub Access Proxy) dla krytycznych paneli.
