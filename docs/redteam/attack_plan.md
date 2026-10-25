# Rencana Skenario Serangan (Attack Plan) - Fase 2
**Mata Kuliah:** TEK1314 Keamanan Siber Terapan  
**Proyek:** PiRoTech IoT Security Hardening  
**Kelompok:** Kelompok 14  
**Target:** Ubuntu Server (`192.168.14.5`)  
**Penyerang:** CyberOps Workstation VM (`192.168.14.100`)  
**Sensor Monitoring:** Security Onion VM (`192.168.14.50`)

---

## 1. Ringkasan Eksekutif
Rencana serangan Fase 2 bertujuan untuk mengevaluasi ketahanan berlapis dari ekosistem PiRoTech IoT (smart pyrolysis reactor) setelah penerapan kontrol defensif Fase 1. Pengujian difokuskan pada tiga vektor utama: manipulasi kendali aktuator fisik, degradasi ketersediaan layanan (*availability*), dan pengujian integritas autentikasi operator.

---

## 2. Matriks Pemetaan Skenario Serangan

| ID Skenario | Nama Skenario | Vektor Serangan / Cek OWASP | Target Endpoint | Dampak Operasional |
| :--- | :--- | :--- | :--- | :--- |
| **ATK-01** | BOLA & Unauthorized Actuator Hijacking | OWASP API1: Broken Object Level Authorization | `/control/buzzer.json` | Mematikan alarm darurat reaktor saat panas berlebih |
| **ATK-02** | Telemetry DoS & Resource Exhaustion | OWASP API4: Unrestricted Resource Consumption | `/sensor_data.json` & `/api/auth/` | Server freeze, data telemetri suhu terputus |
| **ATK-03** | Auth Enumeration & Credential Stuffing | OWASP API2: Broken Authentication | `/api/auth/lookup` & login handler | Pembobolan akun operator dan akses dashboard |

---

## 3. Rincian Teknis Skenario Serangan

### Skenario 1: BOLA & Unauthorized Actuator Tampering (ATK-01)
* **Deskripsi:** Penyerang berusaha mengubah status aktuator buzzer pendingin/darurat reaktor pirolisis tanpa menyediakan token otoritas yang sah.
* **Metodologi Pengujian:**
  1. Penyerang mengirimkan *request* HTTP dengan metode `PUT` dan `POST` yang membawa *payload* `{"buzzer": false}` atau data boolean `false` langsung ke alamat endpoint kendali.
  2. Melakukan variasi pengujian dengan manipulasi *header* (misal: *header spoofing* atau injeksi token acak).
* **Perintah Uji (Proof of Concept):**
  ```bash
  # Uji injeksi status tanpa header token
  curl -i -X PUT -d "false" http://192.168.14.5/control/buzzer.json

  # Uji dengan token palsu
  curl -i -X PUT -H "X-Actuator-Token: invalid_secret_123" -d "false" http://192.168.14.5/control/buzzer.json
  ```
* **Kriteria Keberhasilan Defensif:** Server mengembalikan kode `401 Unauthorized` atau `403 Forbidden`, dan status buzzer pada memori *state* tetap tidak berubah.

---

### Skenario 2: Telemetry DoS & Resource Exhaustion (ATK-02)
* **Deskripsi:** Penyerang mengirimkan lonjakan (*burst*) lalu lintas data telemetri berukuran besar (*oversized payload*) serta membanjiri antrean proses API backend untuk melumpuhkan penyerapan data sensor ESP32 asli.
* **Metodologi Pengujian:**
  1. Melakukan pengiriman *payload* berukuran >1 MB ke endpoint penerima sensor.
  2. Menjalankan *burst request concurrency* tinggi (>50 request/detik) ke endpoint publik backend.
* **Perintah Uji (Proof of Concept):**
  ```bash
  # Uji injeksi payload jumbo (>1MB)
  python3 -c 'import requests; r = requests.post("http://192.168.14.5/sensor_data.json", data="A"*1048576); print("Status:", r.status_code)'

  # Uji burst traffic concurrency
  for i in {1..30}; do curl -s -o /dev/null -w "%{http_code}\n" http://192.168.14.5/api/auth/lookup?username=test & done; wait
  ```
* **Kriteria Keberhasilan Defensif:** Nginx memutus paket raksasa dengan status `413 Request Entity Too Large`, serta menahan laju *burst* dengan status `429 Too Many Requests` tanpa mengganggu proses pembacaan simulator ESP32.

---

### Skenario 3: Auth Enumeration & Credential Stuffing (ATK-03)
* **Deskripsi:** Penyerang memanfaatkan perbedaan waktu respons (*timing attack*) atau respons pesan error pada endpoint autentikasi guna memvalidasi daftar nama pengguna (*username harvest*), dilanjutkan dengan pengujian kamus sandi (*dictionary attack*).
* **Metodologi Pengujian:**
  1. Mengirimkan daftar *wordlist* username ke endpoint pencarian akun.
  2. Mengamati kebocoran informasi apakah pengguna terdaftar atau tidak.
  3. Menjalankan skrip *brute-force* otomatis terhadap kredensial `admin`.
* **Perintah Uji (Proof of Concept):**
  ```bash
  # Enumerasi username
  for u in admin operator technician guest; do
    echo -n "User $u: "
    curl -s http://192.168.14.5/api/auth/lookup?username=$u | grep -o '"exists":[^,]*'
  done
  ```
* **Kriteria Keberhasilan Defensif:** Mekanisme pembatasan akses (*rate-limiting*) segera memblokir alamat IP penyerang setelah ambang batas terlampaui, dan pesan kesalahan dibuat generik tanpa mengungkap keberadaan user di database.

---

## 4. Rencana Observasi & Deteksi (Blue Team Interaction)
Selama skenario penyerangan dijalankan dari node `192.168.14.100`, aktivitas jaringan akan direkam dan dianalisis melalui:
1. **Log Web Server:** `/var/log/nginx/pirotech_access.log` dan `/var/log/nginx/pirotech_error.log`.
2. **Packet Capture:** Wireshark capture file (`.pcap`) pada segmen adapter host-only `vboxnet14`.
3. **Network Security Monitoring (NSM):** Deteksi rule alert berbasis Suricata & Zeek log di Security Onion VM.