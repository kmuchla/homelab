# Cloudflare DNS — model rekordów i zasady konfiguracji

## Kontekst środowiska

W środowisku homelab funkcjonują dwie warstwy DNS:

- **Publiczny DNS (Cloudflare)** — obsługuje domeny i subdomeny wystawiane publicznie (serwisy WWW na VPS, usługi dostępne przez Cloudflare Tunnel).
- **DNS lokalny (Pi-hole, Lokalizacja B)** — pełni rolę głównego resolvera dla sieci lokalnej 192.168.0.0/24, zapewniając filtrację, statystyki zapytań oraz opcjonalne rekordy lokalne.

Połączenie między lokalizacjami realizowane jest przez **OpenVPN (model hub-and-spoke)** z centralnym węzłem na VPS (sieć tunelu: 10.8.0.0/24).
Publiczny VPS jest zabezpieczony zaporą **UFW**, a porty 80/443 są ograniczone wyłącznie do zakresów adresów IP Cloudflare (ochrona origin).

---

## Zasady konfiguracji rekordów DNS w Cloudflare

### 1. Proxied vs DNS-only

#### Proxied (orange cloud)

Stosowane dla:

- wszystkich publicznych serwisów HTTP/HTTPS hostowanych na VPS (Apache2, vhosty),
- nazw domenowych, które mają korzystać z funkcji edge Cloudflare (TLS, redirecty, ochrona origin).

Uzasadnienie:

- ukrycie publicznego adresu IP origin,
- możliwość egzekwowania polityk na edge (TLS, redirect rules),
- ograniczenie powierzchni ataku na origin.

#### DNS-only (grey cloud)

Stosowane wyłącznie dla:

- usług nie-HTTP (np. OpenVPN),
- rekordów technicznych, które nie powinny być proxowane przez Cloudflare.

Uwaga:

- rekordy DNS-only ujawniają publiczny adres IP. W tym środowisku nie są stosowane dla serwisów WWW.

---

### 2. Rekordy DNS dla serwisów WWW (VPS / Apache2)

Zalecany wzorzec:

- rekordy `A/AAAA` lub `CNAME` dla `@`, `www` oraz subdomen serwisów,
- status: **Proxied**,
- TTL: **Auto** (zarządzany przez Cloudflare).

Efekt:

- serwisy WWW dostępne wyłącznie przez Cloudflare,
- origin chroniony regułami zapory (80/443 tylko z IP Cloudflare).

---

### 3. Rekordy DNS dla Cloudflare Tunnel

Dla usług wystawianych przez Cloudflare Tunnel:

- rekordy typu `CNAME` (hostname → endpoint tunelu),
- status: **Proxied**.

Cel:

- brak konieczności otwierania portów na zaporze origin,
- routing oparty o hostname,
- integracja z Cloudflare Zero Trust.

## Standardy bezpieczeństwa DNS w środowisku

### Brak DNS-only dla serwisów WWW

- żaden publiczny serwis HTTP/HTTPS nie posiada rekordu DNS-only wskazującego na publiczny adres IP VPS.

### Ochrona origin

- porty 80/443 na VPS dostępne wyłącznie z zakresów IP Cloudflare,
- brak ekspozycji portów administracyjnych na WAN,
- dostęp administracyjny realizowany przez VPN lub mechanizmy Zero Trust.

---

## Relacja Cloudflare DNS ↔ Pi-hole

W Lokalizacji B Pi-hole pełni funkcję głównego resolvera DNS dla sieci lokalnej:

- filtracja zapytań DNS,
- statystyki i monitoring,
- opcjonalne rekordy lokalne dla usług wewnętrznych.

Publiczne domeny obsługiwane przez Cloudflare nie są zależne od Pi-hole i działają niezależnie od lokalnego resolvera.

---

## Checklist weryfikacyjny

### Weryfikacja proxowania serwisów WWW

```bash
dig +short ventabe.com
curl -I https://ventabe.com
```

Oczekiwane:

- brak bezpośredniego ujawnienia origin IP w kontekście WWW,
- odpowiedzi HTTP obsługiwane przez Cloudflare (TLS, redirecty).

### Weryfikacja ochrony origin

```bash
curl -I http://<PUBLIC_IP_VPS>
curl -I https://<PUBLIC_IP_VPS>
```

Oczekiwane:

- brak poprawnej odpowiedzi serwisu WWW (blokada lub drop),
- brak możliwości ominięcia Cloudflare

## Weryfikacja działania DNS i proxy

### Sprawdzenie rozwiązywania DNS

```bash
dig +short ventabe.com
```

Wynik

```bash
188.114.96.11
188.114.97.11
```

Interpretacja:

- zwrócone adresy IP należą do infrastruktury Cloudflare,
- publiczny adres IP origin (VPS) nie jest ujawniany w odpowiedzi DNS,
- rekord domeny działa w trybie Proxied (orange cloud).

### Sprawdzenie obsługi ruchu HTTPS

```bash
curl -I https://ventabe.com
```

Wybrane nagłówki odpowiedzi:

```bash
HTTP/2 200
server: cloudflare
cf-cache-status: DYNAMIC
cf-ray: <id>-WAW
```

Interpretacja:

- połączenie HTTPS terminowane na Cloudflare Edge,
- ruch HTTP/2 obsługiwany przez Cloudflare,
- żądanie nie trafia bezpośrednio do origin z pominięciem proxy,
- origin jest skutecznie ukryty za warstwą Cloudflare.

Wniosek:

- Konfiguracja DNS dla domeny publicznej spełnia założenia bezpieczeństwa:
- domena jest proxowana przez Cloudflare,
- origin IP nie jest ujawniony przez DNS,
- ruch HTTPS obsługiwany jest na warstwie edge Cloudflare,
- dostęp do origin realizowany jest wyłącznie przez Cloudflare (zgodnie z polityką zapory na VPS).

### Weryfikacja ochrony origin (bezpośredni dostęp po IP)

Test wykonany z sieci zewnętrznej (spoza infrastruktury Cloudflare):

```bash
curl -I http://<PUBLIC_IP_VPS>
curl -I https://<PUBLIC_IP_VPS>
```

Wynik

```bash
curl: (28) Failed to connect to <PUBLIC_IP_VPS> port 80/443
```

Interpretacja:

- publiczny adres IP VPS nie odpowiada na połączenia HTTP/HTTPS,
- zapora sieciowa blokuje ruch spoza zakresów IP Cloudflare,
- bezpośredni dostęp do origin z pominięciem Cloudflare nie jest możliwy.

Wniosek:

- polityka „Cloudflare-only” dla portów 80/443 jest skutecznie egzekwowana,
- origin jest chroniony przed bezpośrednią ekspozycją w Internecie.
