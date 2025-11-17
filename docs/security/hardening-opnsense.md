# Hardening — OPNsense (firewall / router)

Krótkie, praktyczne kroki do zabezpieczenia OPNsense.

1. Aktualizacje

- Włącz automatyczne powiadomienia o aktualizacjach i planuj regularne okna serwisowe.

2. Dostęp administracyjny

- Zmień domyślny port panelu (opcjonalnie), ale nie polegaj tylko na ukryciu.
- Włącz 2FA dla kont administratorów (jeżeli dostępne).
- Ogranicz dostęp do GUI/SSH tylko do zaufanych adresów (np. via VPN / bastion).
- Wyłącz konta domyślne, upewnij się, że używasz silnych haseł.

3. SSH

- Używaj kluczy publicznych, wyłącz logowanie hasłem.
- Ogranicz połączenia SSH do określonych adresów IP lub VPN.

4. Firewall rules

- Implementuj politykę deny-by-default; tylko explicite allow dla potrzebnych usług.
- Zastosuj reguły między VLANami (np. IoT / CCTV nie ma dostępu do LAN serwerów).
- Loguj odrzucone pakiety i regularnie sprawdzaj logi.

5. NAT i port forwarding

- Minimalizuj reguły NAT; stosuj reverse-proxy dla usług webowych zamiast wystawiania portów bezpośrednio.
- Używaj forwardów z ograniczeniem adresów źródłowych.

6. IDS/IPS i monitoring

- Rozważ włączenie Suricata/IDS; monitoruj alerty i tunele reguły.

7. Backup konfiguracji

- Eksportuj zaszyfrowane backupy konfiguracji i trzymaj je poza siecią produkcyjną.

8. Automatyzacja

- Utrzymuj wersjonowane skrypty konfiguracyjne i dokumentację zmian.

Przykłady: zdefiniuj strefy (LAN, MGMT, IOT, CCTV), a następnie stwórz reguły deny all między strefami i dopuszczaj jedynie niezbędny ruch (np. z IOT do internetu na TCP/80,443 oraz do lokalnego DNS).
