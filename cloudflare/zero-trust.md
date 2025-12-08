# Cloudflare Zero Trust

> Praktyczne wdrożenie architektury Zero Trust dla infrastruktury homelabowej obejmującej 3 lokalizacje, z eliminacją tradycyjnych wymagań VPN i otwartych portów.

## 🎯 Założenia

- ✅ **Zero Trust Access** dla 4+ wewnętrznych serwisów z obowiązkowym MFA
- ✅ **Łączność wielolokalizacyjna** przez Cloudflare Tunnel (3 lokalizacje)
- ✅ **Zero otwartych portów** - kompletna ochrona origin IP
- ✅ **Dostęp oparty na tożsamości** - email OTP + ograniczenia geograficzne
- ✅ **Pełny audit trail** - kompleksowe logowanie dostępu
- ✅ **Wysoka dostępność** - 4 redundantne połączenia tunelowe per serwis

---

## 🛠 Technologia

### Infrastruktura

![Cloudflare](https://img.shields.io/badge/Cloudflare-F38020?logo=cloudflare&logoColor=white)
![OPNsense](https://img.shields.io/badge/OPNsense-D94F00?logo=opnsense&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![FreeBSD](https://img.shields.io/badge/FreeBSD-AB2B28?logo=freebsd&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?logo=ubuntu&logoColor=white)

### Bezpieczeństwo i monitoring

![Wazuh](https://img.shields.io/badge/Wazuh-005571?logo=wazuh&logoColor=white)
![Pi-hole](https://img.shields.io/badge/Pi--hole-96060C?logo=pihole&logoColor=white)

### Hardware

![Raspberry Pi](https://img.shields.io/badge/Raspberry%20Pi-A22846?logo=raspberrypi&logoColor=white)

---

## 🚀 Wdrożone serwisy

| Serwis         | Lokalizacja    | Dostęp             | Funkcje                                                    |
| -------------- | -------------- | ------------------ | ---------------------------------------------------------- |
| **Wazuh SIEM** | Lokalizacja B  | wazuh.muchla.pl    | Monitoring bezpieczeństwa, 7+ agentów, centralne logowanie |
| **Pi-hole**    | Raspberry Pi 5 | pihole.muchla.pl   | Filtrowanie DNS, blokowanie reklam, deployment Docker      |
| **OPNsense**   | Lokalizacja A  | opnsense.muchla.pl | Zarządzanie firewallem, konfiguracja VLAN, brama VPN       |
| **SSH (VPS)**  | OVH Cloud      | ssh.muchla.pl      | Terminal w przeglądarce, zabezpieczony MFA                 |

**Wszystkie serwisy wymagają:**

- ✅ Uwierzytelnienia opartego na emailu
- ✅ Jednorazowego kodu PIN (OTP) przez email
- ✅ Zarządzania sesją (24 godziny)
- ✅ Kompleksowego logowania audytu

---

## ![Cloudflare](https://img.shields.io/badge/Cloudflare-F38020?logo=cloudflare&logoColor=white) Zero Trust Access

### Aplikacje Zero Trust Access

![Access Applications](screenshots/access-applications-dashboard.png)

_Cztery aplikacje self-hosted chronione przez polityki Zero Trust: Wazuh SIEM, Pi-hole DNS, OPNsense Firewall oraz terminal SSH_

---

### Aktywne Tunele Cloudflare

![Tunnels Overview](screenshots/tunnels-overview.png)

_Wszystkie cztery tunele pokazujące status HEALTHY z uptimem 2-5 godzin w rozproszonych instancjach konektorów_

---

### Przepływ Uwierzytelniania Wieloskładnikowego

**Krok 1: Weryfikacja Email**

![Login Screen](screenshots/mfa-flow-1-login.png)

_Formularz logowania z prośbą o adres email_

---

**Krok 2: Weryfikacja Turnstile**

![Turnstile](screenshots/turnstile-challenge.png)

_Cloudflare Turnstile - ochrona przed botami_

---

**Krok 3: Wprowadzenie Kodu OTP**

![OTP Entry](screenshots/mfa-flow-2-otp.png)

_Wprowadzenie 6-cyfrowego kodu jednorazowego wysłanego na email_

---

**Krok 4: Pomyślny dostęp**

![Wazuh Dashboard](screenshots/mfa-flow-3-success.png)

_Dashboard Wazuh po pomyślnym uwierzytelnieniu_

---

**Kompletny przepływ uwierzytelniania** demonstrujący weryfikację tożsamości opartą na emailu z jednorazowym kodem PIN oraz ochroną Cloudflare Turnstile przed botami.

---

### Konfiguracja Polityki Dostępu

![Policy Details](screenshots/app-config-policy.png)

_Przykładowa polityka: Włączenie oparte na emailu z czasem trwania sesji 24 godziny_

---

### Konfiguracja Aplikacji

**Ustawienia Podstawowe**

![App Config Basic](screenshots/app-config-basic.png)

_Konfiguracja hostname i czasu trwania sesji_

---

**Metody Logowania**

![Login Methods](screenshots/app-config-login.png)

_One-time PIN włączony jako domyślna metoda uwierzytelniania_

---

**Polityki Dostępu**

![Policies](screenshots/app-config-policy.png)

_Szczegółowa konfiguracja pokazująca setup hostname, uwierzytelnianie One-time PIN oraz szczegółową kontrolę dostępu_

---

### Audyt aktywności infrastruktury

![Admin Logs](screenshots/admin-activity-logs.png)

_Kompleksowe logowanie wszystkich zmian w infrastrukturze: tworzenie tuneli, routing DNS oraz aktualizacje polityk_

---

## 🔐 Funkcje bezpieczeństwa

### Wdrożenie Zero Trust

**Brak wymaganego VPN**

- Bezpośredni dostęp przez Cloudflare Edge
- Eliminacja tradycyjnych endpointów VPN jako powierzchni ataku

**Brak otwartych portów**

- Serwery origin całkowicie ukryte
- Dostęp wyłącznie przez Cloudflare Tunnel z wychodzącymi połączeniami

**Weryfikacja tożsamości**

- Email + OTP dla każdego dostępu
- Brak dostępu opartego na sieci - tylko weryfikacja użytkownika

**Ograniczenia geograficzne**

- Opcjonalna kontrola dostępu oparta na kraju
- Możliwość ograniczenia do określonych zakresów IP

**Zarządzanie sesją**

- Konfigurowalny timeout (1-24 godziny)
- Automatyczne wylogowanie po okresie nieaktywności
- Możliwość wymuszenia ponownego uwierzytelnienia

---

## 📋 Szczegóły Ttchniczne

### Konfiguracja Tuneli

Każdy tunel używa następującej struktury:

```yaml
tunnel:
credentials-file: /etc/cloudflared/.json

ingress:
  - hostname: service.muchla.pl
    service: https://local-ip:port
    originRequest:
      noTLSVerify: true # Dla self-signed certificates

  - service: http_status:404 # Catch-all
```

**Kluczowe cechy:**

- Wychodzące połączenia z origin (outbound-only)
- Brak wymagania public IP
- Automatyczne load balancing między 4 połączeniami
- Failover w przypadku awarii połączenia

---

### Polityki Dostępu

Standardowa polityka dla wszystkich serwisów:

```
Policy Name: Admin Access
Action: Allow

Include:
  - Emails: authorized-user@gmail.com

Require:
  - One-time PIN (email OTP)

Optional:
  - Country: Poland
  - IP ranges: 10.8.0.0/24 (VPN network)

Session Duration: 24 hours
```
