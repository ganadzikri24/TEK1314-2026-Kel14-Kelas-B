# 🛡️ PBL Keamanan Siber (TEK1314) - Kelompok 14
## Proyek: Keamanan Sistem Monitoring IoT PiRoTech (Smart Plastic Pyrolysis)

Repositori ini memuat dokumentasi, konfigurasi infrastruktur, skrip pengujian keamanan (*Red Team*), dan matriks pengerasan (*Blue Team Hardening*) untuk sistem telemetri serta kontrol reaktor pirolisis plastik cerdas **PiRoTech**.

---

## 📌 Ringkasan Skenario Proyek
Proyek ini mensimulasikan lingkungan keamanan siber terpadu pada jaringan lab tertutup (*isolated environment*) untuk mengamankan ekosistem Internet of Things (IoT) berbasis telemetri reaktor pirolisis dari ancaman modifikasi status aktuator (*unauthorized tampering*), kebocoran kredensial, dan serangan penolakan layanan (*Application DoS*).

### 🖥️ Entitas & Arsitektur Jaringan (Subnet `192.168.14.0/24`)

| Entitas | Peran | Alamat IP | Deskripsi Sistem & Peran |
| :--- | :--- | :--- | :--- |
| **Defender Node** | Target / Gateway | `192.168.14.5` | **Ubuntu Server 24.04 LTS** yang menjalankan Nginx Reverse Proxy dan backend API Node.js/Express (`server.js` di port 3000), mengelola data telemetri ESP32 dan replika Firebase RTDB. |
| **Attacker Node** | Red Team | `192.168.14.100` | **CyberOps Lab Workstation** yang digunakan untuk melakukan *reconnaissance*, *endpoint fuzzing*, *rate limit abuse*, serta serangan *actuator tampering*. |
| **Monitoring Node** | Blue Team / SIEM | `192.168.14.50` | **Security Onion (Zeek, Suricata, Wazuh/Kibana)** untuk mendeteksi anomali lalu lintas HTTP, inspeksi paket, dan analisis log audit terpusat. |
| **IoT Node** | Edge Device Simulator | *Localhost/Host* | Simulator Python ESP32 yang mengirimkan metrik suhu (100°C–200°C) dan tekanan reaktor secara berkala via HTTP/JSON. |

---

## 🛠️ Pemetaan Tumpukan Teknologi (Tech Stack)

* **Edge IoT Simulator**: Skrip Python (`esp32_simulator.py`) mengemulasikan mikrokontroler ESP32 mengirim data telemetri pirolisis plastik ke endpoint gateway.
* **API Gateway & Reverse Proxy**: **Nginx 1.28.x** (berperan memvalidasi header, masking banner, menerapkan mitigasi rate-limit, dan memblokir akses rute terlarang).
* **Application Backend**: **Node.js + Express** (port internal 3000) yang memproses logika autentikasi, status aktuator buzzer, dan integrasi data.
* **Database & Cloud Storage**: Replika lokal **Firebase Realtime Database (RTDB)** untuk sinkronisasi telemetri ke antarmuka Web dan Mobile (Android) Client.
* **Testing & Exploitation Tools**: cURL, Python Requests, Nmap, dan Wfuzz dari VM CyberOps.

---

## 🚀 Progres Fase 1: Implementasi Kontrol Hardening (Blue Team)

Seluruh kontrol berikut telah dikonfigurasi pada Nginx Reverse Proxy dan divalidasi keberhasilannya melalui pengujian *Before vs After* dari Attacker VM (`192.168.14.100`):

| No | Kontrol Keamanan | Celah Sebelum Hardening (*Before*) | Hasil Setelah Hardening (*After*) | Status |
| :---: | :--- | :--- | :--- | :---: |
| **1** | **Debug Route Blocking** | `/api/debug/env` membocorkan Private Key Firebase dan variabel sistem (`HTTP 200 OK`). | Akses ditutup total dengan respons eksplisit **`HTTP 403 Forbidden`**. | ✅ **Verified** |
| **2** | **Rate Limiting** | Endpoint `/api/auth/lookup` rentan diserang *brute-force* / enumerasi akun tanpa batas. | Dibatasi zona `5r/s` (`burst=5`). Permintaan berlebih langsung ditolak **`HTTP 429 Too Many Requests`**. | ✅ **Verified** |
| **3** | **Actuator Tampering Restriction** | Penyerang dapat mematikan buzzer keselamatan reaktor via metode `PUT` (`HTTP 200 OK`). | Metode selain `GET` tanpa token autentikasi ditolak dengan kode **`HTTP 401 Unauthorized`**. | ✅ **Verified** |
| **4** | **Server Banner Masking** | Header respons membocorkan versi detail aplikasi (`Server: nginx/1.28.3 (Ubuntu)`). | Versi dan identitas OS disembunyikan menjadi **`Server: nginx`** (`server_tokens off`). | ✅ **Verified** |
| **5** | **HTTP Security Headers** | Header pengaman web standar tidak ditemukan sama sekali pada respons server. | Terpasang `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `X-XSS-Protection`, dan `Referrer-Policy`. | ✅ **Verified** |

> *Bukti tangkapan layar pengujian teknis tersimpan rapi di folder `docs/redteam/evidence/` dan `docs/blueteam/evidence/`.*

---

## 🎯 Rencana Skenario Serangan Fase 2 (Red Team Attack Plan)

Pada Fase 2, pengujian akan diperdalam melalui tiga skenario utama:
1. **Skenario 1 — Actuator Tampering & Authorization Bypass**: Menguji celah kontrol aktuator keselamatan reaktor pirolisis (buzzer & heater) untuk mensimulasikan sabotase operasional.
2. **Skenario 2 — Application-Layer DoS & Resource Exhaustion**: Melancarkan banjir permintaan (*stress-testing*) pada rute telemetri dan autentikasi untuk memverifikasi keandalan konfigurasi Nginx *burst handling*.
3. **Skenario 3 — API Reconnaissance & Information Disclosure**: Melakukan pemindaian otomatis terhadap endpoint internal, rute pengujian (*staging/debug*), serta analisis kebocoran header.

Rencana detail skenario serangan dapat dilihat pada berkas [`docs/redteam/attack_plan.md`](docs/redteam/attack_plan.md).

---

## 📂 Struktur Repositori

```text
├── README.md                              # Dokumentasi utama proyek
├── Presentasi_Progress_PBL_Fase1.pdf      # Slide deck presentasi Progress Fase 1 (PDF)
├── docs/
│   ├── design/
│   │   ├── topology.png                   # Diagram arsitektur & topologi jaringan lab
│   │   └── ip_plan.md                     # Skema alokasi IP address dan subnetting
│   ├── redteam/
│   │   ├── attack_plan.md                 # Rencana pengujian serangan Fase 2
│   │   └── evidence/                      # Screenshot bukti celah kerentanan (Before)
│   └── blueteam/
│       ├── hardening_checklist.md         # Matriks checklist kontrol keamanan
│       └── evidence/                      # Screenshot bukti hasil hardening (After)
└── config/
    └── nginx_pirotech.conf                # Berkas konfigurasi reverse proxy Nginx