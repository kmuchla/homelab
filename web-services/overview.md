# Web Services Overview

Na VPS utrzymywane są cztery niezależne domeny (anonimizowane jako _domain1–domain4_).
Wszystkie domeny są obsługiwane przez serwer Apache2 i zabezpieczone przez Cloudflare (proxy + SSL).

## Architektura

- Serwer WWW: Apache2 (Ubuntu Server)
- Reverse proxy i ochrona: Cloudflare
- Certyfikaty: Cloudflare SSL + certyfikaty Let’s Encrypt na originie
- Dostęp: ruch HTTPS na VPS przepuszczany wyłącznie z adresów IP Cloudflare (filtrowany przez UFW)

## Zakres funkcjonalny

- Hostowanie stron statycznych oraz aplikacji lekkiego typu CMS
- Wsparcie dla wielu vhostów
- Przekierowania między domenami (np. .pl → .com) przez Cloudflare Page Rules
- Wsparcie dla Cloudflare Turnstile na wybranych stronach

## Szczegóły techniczne:

- vhosty znajdują się w `apache-vhost.conf`
- konfiguracja Cloudflare w `cloudflare-config.md`
- zabezpieczenia serwera opisane w `security.md`
