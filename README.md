# Praktikum Modul 3 - Teknologi Komputasi Awan

Selamat datang di repository kelompok **C01** untuk Praktikum Modul 3 Teknologi Komputasi Awan. Project ini berfokus pada implementasi infrastruktur cloud menggunakan Docker dan otomatisasi.

## 👥 Anggota Kelompok (C01)

| No | Nama | NRP | Peran |
| :--- | :--- | :--- | :--- |
| 1 | **Rayhan Agnan Kusuma** | 5027241102 | Praktikan 1 (Setup Infra & Docker) |
| 2 | **Zahra Hafizhah** | 5027241121 | Praktikan 2 |
| 3 | **Dimas Satya A** | 5027241032 | Praktikan 3 |

---

## 📝 Soal Praktikum
Praktikum Modul 3 mencakup implementasi aplikasi fullstack (Backend Node.js & Frontend HTML/JS) yang terhubung dengan database PostgreSQL. Fokus utama modul ini adalah:
1. Dockerization aplikasi (Frontend & Backend).
2. Orchestration menggunakan Docker Compose.
3. Otomatisasi deployment (untuk fase selanjutnya).

---

## 🚀 Progres Pengerjaan

### [Fase 1: Setup Infrastructure & Docker]
Status: ✅ **Selesai**

Pada fase ini, kami telah menyiapkan:
- **Dockerization Backend**: Membuat Dockerfile untuk API Node.js.
- **Dockerization Frontend**: Membuat Dockerfile untuk Nginx web server.
- **Docker Compose**: Orkestrasi Backend, Frontend, dan Database PostgreSQL.
- **Dokumentasi**: Panduan lengkap cara menjalankan aplikasi baik via Docker maupun Manual.

> Detail teknis pengerjaan Fase 1 dapat dilihat pada: **[Pengerjaan Fase1.md](./Pengerjaan%20Fase1.md)**

### [Fase Selanjutnya]
Status: ⏳ **Pending**
- *Menunggu instruksi selanjutnya untuk integrasi atau otomatisasi.*

---

## 🛠️ Cara Menjalankan

1. Clone repository:
   ```bash
   git clone https://github.com/knownasrayy/Praktikum-Modul3-TKA-C01.git
   ```
2. Jalankan Docker Compose:
   ```bash
   docker compose up -d
   ```
3. Akses aplikasi:
   - Frontend: `http://localhost:8080`
   - Backend: `http://localhost:3000`
