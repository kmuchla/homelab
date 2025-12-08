# Cloudflare Configuration

Wszystkie cztery domeny na VPS korzystają z Cloudflare jako warstwy zabezpieczeń oraz jako reverse proxy.

## Aktywne funkcje Cloudflare

### 1. DNS

- Wszystkie rekordy A wskazują publiczny adres VPS.
- Proxy (orange cloud) aktywny dla wszystkich domen.
- Rekordy CNAME dla wariantów www.

### 2. SSL/TLS

- Tryb: **Full (strict)**
- Origin: certyfikaty Let’s Encrypt
- Cloudflare: certyfikaty edge

### 3. Page Rules

- Przekierowania domen, np.:
  - `*.domainX.pl` → `https://domainX.com`
- Wymuszenie HTTPS.

### 4. Turnstile

- Aktywne na wybranych stronach do ochrony formularzy i ukrywania danych przed botami.

### 5. Firewall Rules

Zastosowane selektywnie — różne dla każdej domeny.
Przykłady:

- Blokada ruchu z wybranych krajów,
- Ograniczenie dostępu do `/wp-admin`.

### 6. Bot Fight Mode

- Włączony globalnie na poziomie domen.
- Ogranicza automatyczny scraping i masowe próby odpytywania.

### 7. Zero Trust [w trakcie]

---

## Origin Protection

Na VPS ruch przychodzący jest filtrowany:

- UFW dopuszcza porty 80/443 **wyłącznie z adresów IP Cloudflare**.
- Bezpośrednie połączenia z internetu są odrzucane.

---

## Schemat ruchu

Użytkownik → Cloudflare (WAF/SSL/Firewall Rules)
→ VPS (UFW + Apache2)
→ aplikacja/strona
