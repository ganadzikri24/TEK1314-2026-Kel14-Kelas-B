# Deklarasi Tech Stack & Replikasi Sistem (Target Node) – Kelompok 14

**Proyek IoT:** PiRoTech (*Sistem Monitoring, Analisis, dan Kendali Reaktor Pirolisis Sampah Plastik Berbasis IoT & Cloud Web*)
**Lokasi Implementasi Asli:** Fasilitas Pengolahan Sampah Kelurahan Mulyaharja, Kota Bogor (Sekolah Vokasi IPB University)

---

## 1. Prinsip Isolasi Sistem Produksi vs Simulasi Lab

Sesuai dengan ketentuan *Rules of Engagement (RoE)* pada Panduan PBL TEK1314:

1. **Sistem Produksi Asli Tidak Disentuh:** Seluruh pengujian keamanan, pemindaian (*scanning*), dan simulasi eksploitasi **TIDAK** dilakukan terhadap domain produksi `https://pirotech.id`, `https://monitoring.pirotech.id`, maupun basis data cloud `prikitiw-18dde-default-rtdb.asia-southeast1.firebasedatabase.app`.
2. **Replikasi Penuh pada Target Node VM:** Kelompok 14 membangun replika *tech stack* yang identik di dalam Virtual Machine **Target Node (`192.168.14.5`)** pada segmen jaringan terisolasi `192.168.14.0/24` menggunakan kode sumber aplikasi PiRoTech yang diarahkan ke basis data lokal dan disuplai dengan data telemetri sintetis (*dummy*).

---

## 2. Tabel Pemetaan Tech Stack Asli ke Target Node Replika

| Komponen | Teknologi pada Proyek IoT Asli (PiRoTech Produksi) | Direplikasi sebagai (Target Node VM Lab - `192.168.14.5`) |
| :--- | :--- | :--- |
| **Sistem Operasi Backend** | Cloud Serverless Environment (Node.js 20+ Runtime pada Vercel / Google Cloud) | **Ubuntu Server 22.04 LTS** di dalam Virtual Machine (`192.168.14.5`) yang menjalankan **Node.js v20.x** dan **Nginx Reverse Proxy** |
| **Database / Penyimpanan Data** | **Firebase Realtime Database (NoSQL JSON Tree)** (`prikitiw-18dde-default-rtdb`) & **Firebase Authentication** (JWT + Custom Claims `role` & `username`) | **Firebase Local Emulator Suite** (RTDB Emulator Port `9000` & Auth Emulator Port `9099`) dengan skema node identik (`/sensor_data`, `/batches`, `/log_activity`, `/config`, `/control/buzzer`, `/pirotech/usernames`) dan `firebase.rules.json` berisi data sintetis |
| **Protokol Komunikasi Sensor** | **HTTPS / TLS 1.2 Direct REST API** dari ESP32 Dual-Core FreeRTOS (Core 0 `TaskNetworkCloud` melakukan `POST /sensor_data.json`, `GET /pirotech/config.json`, dan `GET /control/buzzer.json` setiap 2.000 ms) | **HTTP REST API** melalui skrip simulator sensor (`esp32_simulator.py`) yang meniru perilaku ESP32 Core 0: mengirimkan payload JSON telemetri suhu setiap 2 detik serta melakukan polling konfigurasi dan status buzzer ke `http://192.168.14.5` |
| **Dashboard / Monitoring** | **Next.js 16.3.0 (App Router), React 19.2, TypeScript 5, Tailwind CSS v4, Recharts 3.10** pada `monitoring.pirotech.id` dengan *Edge Middleware Routing*, *Watchdog Timer 15 Detik*, serta *Server Route Handlers* | **Aplikasi Web Next.js 16.3.0 PiRoTech** yang dijalankan secara lokal pada Target Node (`http://192.168.14.5`), menyajikan halaman konsol `/overview`, `/dashboard`, `/log-activity`, `/pengaturan`, serta seluruh endpoint REST API internal Next.js yang sama persis dengan sistem asli |