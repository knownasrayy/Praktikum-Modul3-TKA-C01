# Praktikum Modul 3: Ansible - Kelompok C01

Repositori ini berisi hasil pengerjaan Praktikum Modul 3 mata kuliah **Teknologi Komputasi Awan (TKA 2026)** mengenai *Infrastructure as Code* (IaC) menggunakan Ansible dan Docker.

---

## 👥 Profil Kelompok C01

| No | Nama | NRP | Peran |
| :--- | :--- | :--- | :--- |
| 1 | **Rayhan Agnan Kusuma** | 5027241102 | Praktikan 1 (Setup Infra & Docker) |
| 2 | **Zahra Hafizhah** | 5027241121 | Praktikan 2 (Deployment Backend) |
| 3 | **Dimas Satya A** | 5027241032 | Praktikan 3 (Deployment Frontend) |

---

## 📋 Ringkasan Tugas
Tujuan utama praktikum ini adalah melakukan *deployment* simulasi *web login* secara otomatis ke beberapa *node Virtual Machine* (VM) menggunakan Ansible. Aplikasi terdiri dari tiga komponen utama:
1. **Frontend**: Static HTML/JS served by Nginx.
2. **Backend**: Node.js Express API.
3. **Database**: PostgreSQL.

---

## 🚀 Alur & Progres Pengerjaan

### 1️⃣ Fase 1: Setup Infrastruktur & Docker
**Status:** ✅ **SELESAI**
**Tanggung Jawab:** Praktikan 1 (Rayhan)

Fokus pada instalasi environment dasar dan otomatisasi instalasi Docker Engine menggunakan Ansible.
- [x] Membuat `inventory.yml` dengan pembagian group node.
- [x] Menyiapkan *Role* Ansible `docker_setup`.
- [x] Konfigurasi *Firewall* (hanya port 22 terbuka di awal).
- [x] Pengujian koneksi (Ping) dan manual Docker "Hello World".

> **Dokumentasi Lengkap Fase 1:** [Pengerjaan Fase1.md](./Pengerjaan%20Fase1.md)

---

### 2️⃣ Fase 2: Deployment Backend
**Status:** ⏳ **PROSES**
**Tanggung Jawab:** Praktikan 2 (Zahra)

Fokus pada konfigurasi layanan database dan backend menggunakan Ansible.
- [ ] Menyiapkan *Role* Ansible `backend_deployment`.
- [ ] Pengaturan variabel sensitif (db_password, jwt_secret) via Vault.
- [ ] Build & run container Postgres & Backend via Docker Compose.
- [ ] Verifikasi Health Check API.

---

### 3️⃣ Fase 3: Deployment Frontend
**Status:** ⏳ **PROSES**
**Tanggung Jawab:** Praktikan 3 (Dimas)

Fokus pada penyediaan antarmuka web dan integrasi dengan backend.
- [ ] Menyiapkan *Role* Ansible `frontend_deployment`.
- [ ] Konfigurasi `config.js` dinamis menggunakan Template Jinja2.
- [ ] Deployment Frontend via Docker Compose.
- [ ] Pengujian UI End-to-End (Register & Login).

---

## 🛠️ Cara Menjalankan (Local Development)

Untuk mencoba aplikasi secara lokal di komputer Anda:
1. Clone repo ini.
2. Jalankan perintah:
   ```bash
   docker compose up -d
   ```
3. Buka browser:
   - Frontend: `http://localhost:8080`
   - Backend: `http://localhost:3000`

---

## 🗓️ Timeline & Ketentuan
* **Deadline:** Sabtu, 2 Mei 2026 (23.59 WIB).
* **Format Pengumpulan:** Video demonstrasi YouTube (Unlisted).
* **Playlist:** `C01 - TKA 2026`.

Selamat Mengerjakan! 💖