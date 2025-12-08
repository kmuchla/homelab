# Web Services Security

## 1. Apache Hardening

- `ServerTokens Prod`
- `ServerSignature Off`
- Wyłączone listowanie katalogów (`Options -Indexes`)
- Obsługa vhostów z odseparowanymi katalogami
- Wymuszenie HTTPS przez Cloudflare oraz po stronie serwera

---

## 2. Firewall — UFW (Origin Protection)

Na VPS ochrona origin jest realizowana **wyłącznie przez UFW**, nie przez reguły Apache.

### Logika bezpieczeństwa

- Porty 80/443 są **dostępne wyłącznie z adresów IP Cloudflare**.
- Połączenia HTTP/HTTPS spoza Cloudflare są automatycznie blokowane.
- Port 22/tcp (SSH) jest dostępny jedynie z:
  - sieci VPN `10.8.0.0/24`,
  - prywatnej podsieci `192.168.0.0/16` (zdalne zarządzanie z Lokalizacji B).
- Port 1194/udp jest otwarty globalnie do obsługi tunelu OpenVPN.
- Brak bezpośredniego dostępu do usług www bez Cloudflare.

### UFW – aktywna konfiguracja

```
1194/udp                   ALLOW       Anywhere
22/tcp                     ALLOW       10.8.0.0/24
22/tcp                     ALLOW       192.168.0.0/16
80,443/tcp                 ALLOW       173.245.48.0/20
80,443/tcp                 ALLOW       103.21.244.0/22
80,443/tcp                 ALLOW       103.22.200.0/22
80,443/tcp                 ALLOW       103.31.4.0/22
80,443/tcp                 ALLOW       141.101.64.0/18
80,443/tcp                 ALLOW       108.162.192.0/18
80,443/tcp                 ALLOW       190.93.240.0/20
80,443/tcp                 ALLOW       188.114.96.0/20
80,443/tcp                 ALLOW       197.234.240.0/22
80,443/tcp                 ALLOW       198.41.128.0/17
80,443/tcp                 ALLOW       162.158.0.0/15
80,443/tcp                 ALLOW       104.16.0.0/13
80,443/tcp                 ALLOW       104.24.0.0/14
80,443/tcp                 ALLOW       172.64.0.0/13
80,443/tcp                 ALLOW       131.0.72.0/22
80,443/tcp on any          DENY        Anywhere
1194/udp (v6)              ALLOW       Anywhere (v6)
80,443/tcp                 ALLOW       2400:cb00::/32
80,443/tcp                 ALLOW       2606:4700::/32
80,443/tcp                 ALLOW       2803:f800::/32
80,443/tcp                 ALLOW       2405:b500::/32
80,443/tcp                 ALLOW       2405:8100::/32
80,443/tcp                 ALLOW       2a06:98c0::/29
80,443/tcp                 ALLOW       2c0f:f248::/32
```

---

## 3. Reverse Proxy (Cloudflare)

Wszystkie domeny:

- mają aktywne proxy (orange cloud),
- korzystają z SSL Full Strict,
- mają wymuszone HTTPS.

---

## 4. Specyficzne reguły dla domen

- Wybrane domeny mają działa­jące **Firewall Rules** blokujące ruch z wybranych krajów.
- **Page Rules** wykorzystane do przekierowań (np. `.pl` → `.com`).
- **Turnstile** chroni:
  - formularz logowania na jednej z domen,
  - numer telefonu, który pojawia się dopiero po przejściu Turnstile (ochrona przed scrapingiem i botami).

---

## 5. Dodatkowe funkcje CF

- **Bot Fight Mode** włączony globalnie.
- **Turnstile** aktywny na wybranych stronach.
- **Zero Trust** — konfiguracja w trakcie dokumentowania.
