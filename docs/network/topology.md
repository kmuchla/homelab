                 ┌────────────────────────┐
                 │          VPS           │
                 │   OpenVPN Server       │
                 │   Apache + MariaDB     │
                 └───────────┬────────────┘
                             │ 10.8.0.X
                     Hub-and-Spoke VPN
             ┌───────────────┴────────────────┐
             │                                │
        10.8.0.X                          10.8.0.X

┌───────────────────┐ ┌───────────────────┐
│ Lokalizacja A │ │ Lokalizacja B │
│ OPNsense Router │ │ AX1800 Router │
│ 192.168.100.0/24 │ │ 192.168.0.0/24 │
└───────┬────────────┘ └─────────┬──────────┘
│ │
VLAN 102 – CCTV ┌─────────┴───────────┐
192.168.102.0/24 │ │
│ │
Raspberry Pi (Pi-hole, HA) Synology NAS
192.168.0.X 192.168.0.X (zamaskowano)

flowchart TD

    subgraph VPS["VPS (OVH)"]
      V1["OpenVPN Server<br>10.8.0.X"]
      V2["Apache + MariaDB"]
    end

    subgraph A["Lokalizacja A"]
      Arouter["OPNsense<br>192.168.100.0/24<br>10.8.0.X"]
      AlAN["LAN 192.168.100.0/24"]
      Acctv["CCTV 192.168.102.0/24"]
    end

    subgraph B["Lokalizacja B"]
      Brouter["AX1800 Router<br>192.168.0.0/24<br>10.8.0.X"]
      BRPi["Raspberry Pi<br>Pi-hole + HA<br>192.168.0.X"]
      BNAS["Synology DS224+<br>192.168.0.X"]
      BHyper["Hyper-V Host<br>192.168.0.X"]
    end

    V1 --- Arouter
    V1 --- Brouter

    Arouter --- AlAN
    Arouter --- Acctv

    Brouter --- BRPi
    Brouter --- BNAS
    Brouter --- BHyper
