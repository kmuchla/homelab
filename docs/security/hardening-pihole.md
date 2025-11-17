# Hardening — Pi-hole (Docker)

Kroki zabezpieczenia instalacji Pi-hole uruchomionej w Docker Compose.

1. Uruchomienie w izolowanym środowisku

- Uruchom Pi-hole w dedykowanym bridge network lub macvlan (rozważ izolację od hosta).

2. Docker Compose (przykład minimalny)

```
version: '3.8'
services:
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    environment:
      - TZ=Europe/Warsaw
      - WEBPASSWORD=changeme  # zamiast trzymać w repo -> użyj env file poza repo
    volumes:
      - ./etc-pihole:/etc/pihole
      - ./etc-dnsmasq.d:/etc/dnsmasq.d
    networks:
      - pihole_net
    restart: unless-stopped

networks:
  pihole_net:
    driver: bridge
```

3. Sekrety i hasła

- Nie trzymać `WEBPASSWORD` w repo. Użyj `.env` w `.gitignore` lub secret managera.
- Ogranicz dostęp do panelu web: whitelist adresów lub reverse-proxy z dodatkową autoryzacją.

4. Upstream DNS i prywatność

- Preferuj DNS-over-HTTPS/DoT upstream (Cloudflare/Quad9) jeśli prywatność jest wymagana.
- Włącz DNSSEC jeśli twoje upstreamy wspierają.

5. Sieć i dostęp

- Pi-hole jako główny DNS dla sieci wewnętrznej — ale ogranicz kto może z niego korzystać (ACL).
- Jeśli udostępniasz Pi-hole przez VPN, pamiętaj o ograniczeniu dostępu panelu i rate-limiting.

6. Logowanie i retention

- Ustaw retention logów na sensowny poziom.
- Agreguj ważne logi do zewnętrznego systemu (Wazuh) jeśli to możliwe.

7. Aktualizacje

- Aktualizuj obraz Dockera kontrolowanie (test na staging) i unikaj automatycznych nagłych aktualizacji w produkcji bez testów.

8. Backup

- Regularne backupy `/etc/pihole` i `dnsmasq.d` na zaszyfrowane repo/postawienie kopii na Synology.

9. Rate-limiting / ochrony przed abuse

- Rozważ włączenie ochrony DNS flood na firewallu i ograniczeń na PoC.
