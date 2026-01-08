# Cloudflare Tunnel — architektura i model dostępu

## Zakres

Dokument opisuje architekturę udostępniania usług HTTP/HTTPS z wykorzystaniem Cloudflare Tunnel w środowisku homelab, obejmującą:

- sposób zestawienia połączenia,
- routing ruchu po hostname,
- relację z Cloudflare DNS i SSL/TLS,
- wpływ na bezpieczeństwo origin.

Celem rozwiązania jest eliminacja bezpośredniej ekspozycji usług oraz uproszczenie modelu dostępu.

---

## Kontekst środowiska

Cloudflare Tunnel wykorzystywany jest do udostępniania wybranych usług:

- bez otwierania portów na zaporze sieciowej,
- bez publicznego adresu IP po stronie usług backendowych,
- z pełnym wykorzystaniem warstwy edge Cloudflare (DNS, TLS, Access).

Tunnel działa w środowisku z:

- centralnym VPS,
- zaporą UFW,
- usługami backendowymi działającymi lokalnie lub na VPS,
- domenami obsługiwanymi przez Cloudflare DNS.

---

## Model działania Cloudflare Tunnel

### Zestawienie połączenia

- proces `cloudflared` inicjuje **połączenie wychodzące** z origin do Cloudflare,
- nie są przyjmowane żadne połączenia przychodzące na origin,
- brak konieczności wystawiania portów 80/443 na firewallu.

Model logiczny:

```
Client ──HTTPS──> Cloudflare Edge ──Tunnel──> cloudflared ──HTTP/HTTPS──> Service
```

---

## Lokalizacja komponentów

### cloudflared

- uruchomiony jako usługa systemowa,
- działa na VPS pełniącym rolę punktu styku z Internetem,
- posiada dostęp do usług backendowych (lokalnych lub routowanych).

### Usługi backendowe

- nie są publicznie dostępne,
- komunikacja odbywa się wyłącznie wewnętrznie (localhost, sieć prywatna, VPN).

---

## Routing ruchu (ingress rules)

### Zasada

Routing realizowany jest na podstawie **hostname**, a nie portów.

- każda usługa posiada przypisaną nazwę DNS,
- reguły `ingress` mapują hostname → konkretną usługę,
- stosowana jest reguła domyślna (fallback) zwracająca błąd dla nieznanych hostów.

Korzyści:

- brak wildcardów bez kontroli,
- jednoznaczne mapowanie usług,
- łatwiejsza diagnostyka błędów routingu.

---

## Relacja z Cloudflare DNS

- hostnames używane przez Tunnel są zdefiniowane w Cloudflare DNS,
- rekordy DNS działają w trybie **Proxied**,
- Cloudflare kieruje ruch do odpowiedniego tunelu na podstawie nazwy hosta.

Efekt:

- brak ujawnienia adresów IP origin,
- pełna kontrola ruchu na warstwie edge.

---

## Relacja z SSL/TLS

- TLS terminowany na Cloudflare Edge,
- komunikacja Edge → Tunnel zabezpieczona przez Cloudflare,
- usługi backendowe nie wymagają publicznych certyfikatów TLS.

Model:

- certyfikaty i polityki TLS zarządzane centralnie na Cloudflare,
- brak potrzeby zarządzania certyfikatami po stronie usług wewnętrznych.

---

## Integracja z Zero Trust Access

Cloudflare Tunnel umożliwia bezpośrednią integrację z:

- Cloudflare Zero Trust Access,
- politykami dostępu opartymi o tożsamość użytkownika.

Efekt:

- dostęp do usług możliwy tylko po spełnieniu polityk Access,
- brak klasycznego VPN dla dostępu do aplikacji webowych,
- rozdzielenie dostępu administracyjnego i użytkowego.

---

## Bezpieczeństwo i ochrona origin

### Eliminacja exposed ports

- brak otwartych portów HTTP/HTTPS na firewallu origin,
- brak możliwości skanowania usług backendowych z Internetu.

### Redukcja powierzchni ataku

- origin nie przyjmuje ruchu bezpośredniego,
- jedynym punktem wejścia jest Cloudflare Edge,
- polityki bezpieczeństwa egzekwowane centralnie.

---

## Aktualizacje i utrzymanie

- `cloudflared` uruchomiony jako usługa systemowa,
- aktualizacje realizowane przez pakiet systemowy,
- restart usługi kontrolowany (systemd).

---

## Weryfikacja działania

### Status usługi

```bash
systemctl status cloudflared
```

Oczekiwane:

- usługa aktywna (running),
- brak błędów połączenia z Cloudflare.

### Weryfikacja dostępu do usługi

```
curl -I https://wazuh.muchla.pl
```

Oczekiwane:

- odpowiedź HTTP obsługiwana przez Cloudflare,
- brak bezpośredniego dostępu do backendu po IP.

Wnioski końcowe

- Cloudflare Tunnel eliminuje konieczność wystawiania portów publicznych,
- routing oparty o hostname zapewnia precyzyjną kontrolę dostępu,
- integracja z DNS, TLS i Zero Trust upraszcza architekturę,
- origin pozostaje niewidoczny i chroniony,
- rozwiązanie jest spójne z założeniami bezpieczeństwa środowiska homelab.

### Weryfikacja procesu `cloudflared` (origin / VPS)

```bash
systemctl status cloudflared
```

Wybrane linie wyniku:

```
Loaded: loaded (...; enabled; ...)
Active: active (running) since ...
Main PID: ... (cloudflared)
└─ ... /usr/bin/cloudflared --no-autoupdate --config /etc/cloudflared/config.yml tunnel run
```

Interpretacja:

- usługa cloudflared jest zainstalowana jako systemd service i jest włączona do autostartu (enabled),
- proces działa w trybie ciągłym (active (running)), co potwierdza stabilne zestawienie tunelu,
- wykorzystywana jest jawna ścieżka do pliku konfiguracyjnego (/etc/cloudflared/config.yml),
- wyłączona jest automatyczna aktualizacja (--no-autoupdate), co stabilizuje zachowanie komponentu w środowisku produkcyjnym.

Wniosek:

- Cloudflare Tunnel jest utrzymywany w sposób operacyjny (systemd, autostart, kontrolowana konfiguracja),
- stan usługi pozwala na szybkie potwierdzenie dostępności tunelu podczas diagnostyki incydentów.

### Weryfikacja kontroli dostępu (Cloudflare Edge)

```bash
curl -I https://wazuh.muchla.pl
```

Wynik:

```
HTTP/2 403
cf-mitigated: challenge
server: cloudflare
cf-ray: <id>-WAW
```

Interpretacja:

- żądanie HTTP zostało obsłużone przez Cloudflare Edge,
- dostęp do zasobu został zablokowany na warstwie edge,
- nagłówek cf-mitigated: challenge wskazuje na zastosowanie mechanizmu ochronnego Cloudflare (challenge),
- żądanie nie zostało przekazane do origin.

Wniosek:

- publiczny dostęp do usługi Wazuh UI jest ograniczony polityką Cloudflare,
- ochrona realizowana jest przed dotarciem ruchu do backendu,
- mechanizm kontroli dostępu działa zgodnie z założeniami bezpieczeństwa (default deny).
