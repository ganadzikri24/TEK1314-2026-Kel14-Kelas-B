# Checklist & Verifikasi Hardening Sistem - Fase 1
**Mata Kuliah:** TEK1314 Keamanan Siber Terapan  
**Proyek:** PiRoTech IoT Security Hardening  
**Kelompok:** Kelompok 14  
**Target Node:** Ubuntu Server 24.04 LTS (`192.168.14.5`)  
**Reverse Proxy:** Nginx 1.28.x  

---

## 1. Matriks Status Implementasi Kontrol Hardening

| No | Nama Kontrol / Mekanisme | Target Komponen | Standar Acuan | Status | Berkas Bukti (Evidence) |
| :-: | :--- | :--- | :--- | :-: | :--- |
| **01** | Isolasi & Pemblokiran Debug Route | `/api/debug/*` | CWE-215: Info Leak | **TERVERIFIKASI** | `vuln_env_leak_before.png`<br>`hardening_env_leak_after.png` |
| **02** | Rate Limiting & Proteksi DoS | `/api/auth/*` | OWASP API4: Resource Limit | **TERVERIFIKASI** | `vuln_ratelimit_before.png`<br>`hardening_ratelimit_after.png` |
| **03** | Pembatasan Metode Kontrol Aktuator | `/control/buzzer.json` | OWASP API1: BOLA / Tampering | **TERVERIFIKASI** | `vuln_buzzer_tamper_before.png`<br>`hardening_buzzer_tamper_after.png` |
| **04** | Server Banner & Token Masking | Header HTTP Server | CWE-200: Exposure of Info | **TERVERIFIKASI** | `vuln_server_banner_before.png`<br>`hardening_server_banner_after.png` |
| **05** | HTTP Security Headers Enactment | Global Nginx Config | OWASP Secure Headers | **TERVERIFIKASI** | `vuln_security_headers_before.png`<br>`hardening_security_headers_after.png` |
| **06** | Pembatasan Ukuran Payload HTTP | Client Request Body | CWE-400: Resource Exhaustion | *Direncanakan (Fase 2)* | `vuln_payload_size_before.png`<br>`hardening_payload_size_after.png` |

---

## 2. Rincian Konfigurasi & Hasil Validasi

### Kontrol 01: Pemblokiran Debug Endpoint
* **Tujuan:** Mencegah kebocoran kredensial runtime, Firebase service account keys, dan konfigurasi environment.
* **Konfigurasi Nginx:**
  ```nginx
  location ^~ /api/debug/ {
      return 403 '{"error": "Forbidden - Debug endpoints disabled in production"}\n';
      add_header Content-Type application/json;
  }
  ```
* **Hasil Pengujian:**
  * **Before:** `HTTP/1.1 200 OK` (Private key dan service account terekspos dalam JSON).
  * **After:** `HTTP/1.1 403 Forbidden` (Akses ditolak di layer reverse proxy sebelum mencapai Node.js).

---

### Kontrol 02: Rate Limiting pada Rute Autentikasi
* **Tujuan:** Mencegah enumerasi akun secara agresif, serangan brute-force dictionary, dan flooding request.
* **Konfigurasi Nginx:**
  ```nginx
  limit_req_zone $binary_remote_addr zone=api_limit:10m rate=5r/s;

  location /api/auth/ {
      limit_req zone=api_limit burst=5 nodelay;
      limit_req_status 429;
      proxy_pass http://127.0.0.1:3000;
  }
  ```
* **Hasil Pengujian:**
  * **Before:** 15 request berurutan diterima seluruhnya (`200 OK`).
  * **After:** Request ke-6 ke atas langsung diputus dengan respons kode `429 Too Many Requests`.

---

### Kontrol 03: Proteksi Integritas Aktuator Buzzer
* **Tujuan:** Mengamankan aktuator fisik sistem pirolisis agar status kontrol buzzer tidak bisa diubah semena-mena via HTTP PUT/POST liar.
* **Konfigurasi Nginx:**
  ```nginx
  location = /control/buzzer.json {
      if ($request_method != GET) {
          return 401 '{"error": "Unauthorized: actuator token required"}\n';
      }
      proxy_pass http://127.0.0.1:3000;
  }
  ```
* **Hasil Pengujian:**
  * **Before:** `curl -X PUT -d "false"` menghasilkan `HTTP/1.1 200 OK` (Buzzer dimatikan).
  * **After:** `curl -X PUT -d "false"` menghasilkan `HTTP/1.1 401 Unauthorized`.

---

### Kontrol 04: Server Banner Masking
* **Tujuan:** Meminimalisasi celah fingerprinting versi software web server dari penyerang.
* **Konfigurasi Nginx:**
  ```nginx
  server_tokens off;
  ```
* **Hasil Pengujian:**
  * **Before:** Header memuat detail `Server: nginx/1.28.3 (Ubuntu)`.
  * **After:** Header hanya memuat label umum `Server: nginx`.

---

### Kontrol 05: HTTP Security Headers
* **Tujuan:** Melindungi dashboard monitoring sistem dari serangan umum peramban seperti clickjacking dan MIME-type confusion.
* **Konfigurasi Nginx:**
  ```nginx
  add_header X-Frame-Options "DENY" always;
  add_header X-Content-Type-Options "nosniff" always;
  add_header X-XSS-Protection "1; mode=block" always;
  add_header Referrer-Policy "no-referrer-when-downgrade" always;
  ```
* **Hasil Pengujian:**
  * **Before:** Pemeriksaan header dengan curl menghasilkan output kosong.
  * **After:** Seluruh header keamanan terkonfirmasi muncul pada respons `curl -I`.

---

## 3. Direktori Penyimpanan Bukti (Evidence Directory)
Seluruh berkas tangkapan layar verifikasi tersimpan di dalam struktur repositori:
* Bukti Kerentanan Awal: `docs/redteam/evidence/`
* Bukti Pasca-Hardening: `docs/blueteam/evidence/`