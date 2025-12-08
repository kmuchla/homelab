# DNS Setup (Cloudflare)

Każda z czterech domen korzysta z Cloudflare DNS.

## Rekordy

### 1. Rekordy A

- `domainX.com` → publiczny adres VPS (proxied)
- `www.domainX.com` → CNAME na `domainX.com`

### 2. Rekordy CNAME

- Warianty www lub aliasy migracyjne

### 3. Rekordy TXT

- SPF (jeśli domena obsługuje pocztę — opcjonalnie)
- Weryfikacje Cloudflare, Google itp.

### 4. CAA

- Zezwolenie na Let’s Encrypt

---

## Proxy (Orange Cloud)

Wszystkie domeny mają aktywny Cloudflare Proxy:

- CF ukrywa IP VPS,
- ochrona przed DDoS,
- caching dla statycznych zasobów.

---

## SSL/TLS

- Tryb: **Full Strict**
- Origin: Let’s Encrypt
- Edge: certyfikaty CF

---

## Page Rules

Używane do:

- przekierowań .pl → .com,
- wymuszania HTTPS,
- ustawienia cache dla statycznych plików (opcjonalnie).
