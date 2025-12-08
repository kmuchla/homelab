# Cloudflare Security Model

Cloudflare jest główną warstwą ochrony dla wszystkich usług WWW hostowanych na VPS.
Model bezpieczeństwa obejmuje:

## 1. Proxy (reverse proxy)

- ukrycie prawdziwego adresu VPS,
- ruch trafia na origin wyłącznie przez określone IP CF.

## 2. Firewall Rules

- blokowanie ruchu z określonych krajów,
- filtrowanie botów i skryptów,
- dodatkowe reguły dla dynamicznych paneli.

## 3. SSL Full Strict

- weryfikacja certyfikatu na VPS,
- ochrona przed MITM.

## 4. Bot Fight Mode

- ogranicza automaty botnetowe i scraping.

### Turnstile

Turnstile jest wdrożone w dwóch scenariuszach:

1. **Zabezpieczenie numeru telefonu** — numer jest ujawniany dopiero po poprawnej weryfikacji Turnstile.
2. **Zabezpieczenie logowania** — Turnstile aktywne na jednej stronie, aby ograniczyć automatyczne próby logowania.

Pełna konfiguracja Cloudflare Turnstile [dostępna tutaj](waf-concepts.md#4-turnstile).
