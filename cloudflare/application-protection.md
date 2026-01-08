# Cloudflare — Application Protection

## Zakres

Dokument opisuje mechanizmy ochrony aplikacyjnej realizowane na Cloudflare Edge
w środowisku homelab. Uwzględnione są wyłącznie funkcje faktycznie używane
i możliwe do zweryfikowania operacyjnie.

Celem ochrony aplikacyjnej jest:

- ograniczenie nieautoryzowanego dostępu,
- przeniesienie decyzji bezpieczeństwa na warstwę edge,
- ochrona backendów przed bezpośrednią ekspozycją.

## Kontekst środowiska

- aplikacje webowe hostowane są na VPS lub udostępniane przez Cloudflare Tunnel,
- brak publicznego dostępu do backendów po IP,
- Cloudflare pełni rolę jedynego punktu wejścia (edge).

## Mechanizmy ochronne używane w środowisku

### 1. Redirect Rules / Page Rules

#### Zastosowanie

- wymuszanie HTTPS,
- normalizacja ruchu (http → https),
- kontrola zachowania URL na edge.

#### Zasady

- redirecty realizowane są **na Cloudflare Edge**,
- unika się dublowania redirectów na origin,
- zapobiega to pętlom przekierowań.

Efekt:

- spójne zachowanie aplikacji,
- uproszczona diagnostyka TLS.

---

### 2. Challenge / Mitigations (Edge Enforcement)

### Charakterystyka

- Cloudflare podejmuje decyzję o dostępie przed dotarciem żądania do backendu,
- żądania niespełniające polityk są blokowane lub challengowane.

```bash
curl -I https://wazuh.example.com
```

#### Wynik:

```
HTTP/2 403
cf-mitigated: challenge
server: cloudflare
```

#### Interpretacja:

- żądanie zostało obsłużone i zablokowane na Cloudflare Edge,
- backend nie został osiągnięty,
- decyzja ma charakter ochronny, nie aplikacyjny.

### 3. Cloudflare Turnstile

#### Zastosowanie

- ochrona wybranych punktów aplikacyjnych,
- eliminacja klasycznych CAPTCHA,
- weryfikacja użytkownika bez pogarszania UX.

#### Charakterystyka

- walidacja wykonywana po stronie Cloudflare,
- backend otrzymuje jedynie wynik weryfikacji,
- brak konieczności implementacji logiki antybotowej po stronie aplikacji.

Szczegółowa konfiguracja dostępna [tutaj](https://github.com/kmuchla/wordpress-phone-protection).

### Relacja z Zero Trust

- Mechanizmy ochrony aplikacyjnej:
- uzupełniają Cloudflare Zero Trust Access,
- nie zastępują kontroli tożsamości,
- działają jako dodatkowa warstwa ochronna na edge.

#### Weryfikacja

Testy potwierdzające działanie ochrony aplikacyjnej
zebrane są w dokumencie: 📄 [cloudflare/verification.md](/cloudflare/verification.md)
