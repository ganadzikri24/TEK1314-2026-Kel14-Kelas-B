## Skenario Proyek

Proyek ini mensimulasikan skenario serangan dan pertahanan siber pada jaringan lab tertutup (isolated network) dengan tiga entitas utama:

- **Attacker Node** (Kali Linux) — melakukan reconnaissance (scanning) dan eksploitasi terhadap target menggunakan tools seperti Nmap dan Metasploit.
- **Target Server / Korban** (Metasploitable 2) — server yang sengaja memiliki celah keamanan default untuk didemonstrasikan proses eksploitasi oleh Red Team.
- **Monitoring Node** (Security Onion) — memantau seluruh lalu lintas jaringan pada segmen lab untuk mendeteksi aktivitas mencurigakan (IDS/IPS & log analysis) sebagai bagian dari peran Blue Team.

Seluruh node berada pada subnet `192.168.14.0/24` (host-only/internal network), terisolasi dari jaringan produksi maupun internet, guna memastikan seluruh aktivitas pengujian keamanan berjalan aman dan terkendali.

Alur kerja proyek mengikuti pembagian peran tim:
- **Red Team** melakukan riset celah keamanan pada Target Server dan mendemonstrasikan eksploitasi.
- **Blue Team** merancang topologi jaringan, skema IP address, dan memastikan Monitoring Node dapat mendeteksi aktivitas serangan tersebut.
- **Lead** mengoordinasikan kesepakatan desain antara kedua tim dan mengelola dokumentasi proyek.

Desain topologi lengkap tersedia di `docs/design/topology.png`, dan alokasi IP address di `docs/design/ip_plan.md`.
