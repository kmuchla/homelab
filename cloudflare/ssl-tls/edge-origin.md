# Cloudflare SSL/TLS — Edge ↔ Origin (edge-origin.md)

## Zakres

Dokument opisuje model szyfrowania TLS pomiędzy:

- klientem końcowym a Cloudflare Edge,
- Cloudflare Edge a origin (VPS z Apache2).

Celem konfiguracji jest zapewnienie:

- szyfrowania end-to-end,
- ochrony origin,
- jednoznacznej diagnostyki problemów TLS i redirectów.

## Model SSL/TLS w środowisku

### Terminacja TLS

- TLS terminowany na **Cloudflare Edge** (Universal SSL),
- połączenie **Edge → Origin** również szyfrowane (TLS).

Model:

```
Client ──TLS──> Cloudflare Edge ──TLS──> Origin (VPS)
```

## Tryb SSL/TLS

### Konfiguracja

- **SSL/TLS mode: Full (Strict)**

### Uzasadnienie

- wymusza poprawny certyfikat po stronie origin,
- eliminuje ryzyko:
  - MITM,
  - błędnych lub samopodpisanych certyfikatów,
- zapewnia rzeczywiste szyfrowanie end-to-end.

Tryby `Flexible` oraz `Full` (bez Strict) nie są stosowane ze względów bezpieczeństwa.

---

## Certyfikat po stronie origin

- origin (VPS) posiada ważny certyfikat TLS (Let's Encrypt),
- certyfikat odnawiany automatycznie,
- certyfikat nie jest przeznaczony do bezpośredniego dostępu publicznego,
- dostęp do origin ograniczony zaporą sieciową (patrz: ochrona origin).

---

## Wymuszanie HTTPS

### Zasada

Wymuszanie HTTPS realizowane jest **w jednym miejscu**, aby uniknąć pętli przekierowań.

Dopuszczalne scenariusze:

- redirect HTTPS na Cloudflare Edge (Redirect Rules),
- lub redirect na origin (Apache).

Niedopuszczalne:

- jednoczesne wymuszanie HTTPS na Edge i na origin.

---

## Typowy problem: pętla przekierowań (redirect loop)

### Objawy

- przeglądarka: `ERR_TOO_MANY_REDIRECTS`,
- wielokrotne odpowiedzi 301/302 w nagłówkach HTTP.

### Najczęstsza przyczyna

- wymuszanie HTTPS zarówno na Cloudflare Edge, jak i na origin.

### Diagnostyka

```
curl -I http://example.com
curl -I https://example.com
```

Interpretacja:

- analiza kodów odpowiedzi i nagłówków Location.

Rozwiązanie:

- pozostawienie mechanizmu redirectu tylko w jednym punkcie,
- usunięcie nadmiarowego redirectu.

## Testy w homelab

Bezpośredni dostęp po IP oraz weryfikacja na warstwie Edge dostępne [tutaj.](../dns/records.md##weryfikacja-dzialania-dns-i-proxy)

