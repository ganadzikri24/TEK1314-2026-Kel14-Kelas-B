# IP Plan – Kelompok 14

**Subnet Kelompok:** `192.168.14.0/24`
**Mode Jaringan:** Host-only / Internal Network (isolated lab)

| Hostname               | IP Address       | OS Direncanakan |Keterangan                                                             |
|------------------------|------------------|-----------------|-----------------------------------------------------------------------|
| Target Server (Korban) | 192.168.14.5/24  | Metasploitable 2| Berisi service dengan celah keamanan default (FTP, Samba, MySQL, dsb.) untuk didemonstrasikan Red Team             |
| Attacker Node          | 192.168.14.100/24| Kali Linux      | Digunakan Red Team untuk scanning & exploitation                                |
| Monitoring Node        | 192.168.14.50/24 | Security Onion  | Diposisikan agar dapat memantau seluruh trafik di segmen 192.168.14.0/24                                          |                          


| Port | Service | Potensi Celah Keamanan                                                                                   |
|------|---------|----------------------------------------------------------------------------------------------------------|
| 80   | HTTP    | Trafik dikirim tanpa enkripsi, jadi data yang lewat bisa disadap                                         |
| 443  | HTTPS   | Sudah dienkripsi, tapi tetap rawan disalahgunakan lewat sertifikat palsu atau serangan Man-in-the-Middle |
| 3306 | MySQL   | Kalau kredensialnya masih default atau lemah, database bisa diakses langsung tanpa izin                  |
| 21   | FTP     | Login dikirim tanpa enkripsi; di Metasploitable versinya bahkan punya backdoor yang sudah dikenal umum   |
| 22   | SSH     | Aksesnya sudah terenkripsi, tapi sering jadi sasaran brute-force login                                   |
| 23   | Telnet  | Tidak ada enkripsi sama sekali, semua yang diketik termasuk password bisa disadap                        |


