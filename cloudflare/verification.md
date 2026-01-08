# Cloudflare — weryfikacja

Dokument zawiera zestaw testów potwierdzających poprawną konfigurację
i egzekwowanie usług Cloudflare w środowisku homelab.

Testy mają charakter operacyjny i mogą być używane jako:

- checklist przed zmianą,
- checklist po zmianie,
- materiał do diagnostyki i audytu.

---

## 1. DNS i proxy (orange cloud)

### Test

```bash
dig +short ventabe.com
```

### Oczekiwany wynik

- adresy IP należące do infrastruktury Cloudflare.

### Wniosek

- domena działa w trybie proxied,
- publiczny adres IP origin nie jest ujawniany przez DNS.

## 2. TLS na Cloudflare Edge

### Test

```
curl -I https://ventabe.com
```

### Wynik

```
HTTP/2 200
server: cloudflare
cf-ray: <id>
```

### Wniosek

- TLS terminowany na Cloudflare Edge,
- ruch obsługiwany przez infrastrukturę Cloudflare.

## 3. Ochrona origin (brak dostępu po IP)

### Test

```
curl -I http://<PUBLIC_IP_VPS>
curl -I https://<PUBLIC_IP_VPS>
```

### Oczekiwany wynik

- brak odpowiedzi / timeout.

### Wniosek

- porty 80/443 na origin nie są publicznie dostępne,
- dostęp do usług WWW możliwy wyłącznie przez Cloudflare.

## 4. Cloudflare Tunnel

### Test prowadzone na VPS

```
systemctl status cloudflared
```

### Oczekiwany stan

- active (running),
- usługa uruchamiana przez systemd,
- jawna ścieżka do pliku konfiguracyjnego.

### Wniosek

- tunel jest zestawiony w sposób trwały i operacyjny,
- brak zależności od ręcznego uruchamiania procesu.

## 5. Zero Trust / Application Protection

## Test

```
curl -I https://wazuh.example.com
```

### Oczekiwany wynik

```
HTTP/2 403
cf-mitigated: challenge
server: cloudflare
```

### Wniosek

- dostęp zablokowany na Cloudflare Edge,
- backend nie jest osiągalny bez spełnienia polityk,
- egzekwowanie zasad default deny.
