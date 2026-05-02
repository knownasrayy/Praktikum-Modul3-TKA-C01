# 🧪 Panduan Pengujian Fase 1 — Setup Infra & Docker

> **Praktikan 1**: Rayhan Agnan Kusuma  
> **Tujuan**: Memverifikasi bahwa Dockerization aplikasi (lokal) dan konfigurasi Ansible berjalan dengan benar.

---

## 📋 Daftar Isi

1. [Prasyarat](#1-prasyarat)
2. [Uji Lokal — Docker Compose](#2-uji-lokal--docker-compose)
3. [Uji Endpoint Backend (API)](#3-uji-endpoint-backend-api)
4. [Uji Frontend di Browser](#4-uji-frontend-di-browser)
5. [Verifikasi Koneksi Database](#5-verifikasi-koneksi-database)
6. [Uji Ansible — Ping Node VM](#6-uji-ansible--ping-node-vm)
7. [Cleanup (Bersihkan Container)](#7-cleanup-bersihkan-container)
8. [Checklist Hasil Pengujian](#8-checklist-hasil-pengujian)

---

## 1. Prasyarat

Sebelum menjalankan pengujian, pastikan kondisi berikut terpenuhi:

| Prasyarat | Cara Cek |
|---|---|
| Docker Desktop sedang berjalan | Lihat icon Docker di taskbar (harus berwarna, bukan abu-abu) |
| Berada di direktori project | `cd` ke folder `Modul 3 Praktikum` |
| Tidak ada container lama yang konflik | Jalankan cleanup dulu jika perlu (lihat bagian 7) |

```powershell
# Pastikan Docker Engine aktif
docker info
```

**Output yang diharapkan**: Muncul informasi server Docker (versi, OS, dll). Jika error, buka Docker Desktop terlebih dahulu dan tunggu sampai ready.

---

## 2. Uji Lokal — Docker Compose

### 2.1 — Build dan Jalankan Semua Container

Perintah ini akan **mem-build image** dari Dockerfile Backend dan Frontend, lalu **menjalankan** 3 container sekaligus (DB, Backend, Frontend) secara bersamaan di background (`-d`).

```powershell
docker compose up -d --build
```

**Penjelasan flag:**
- `up` → Buat dan jalankan container
- `-d` → *Detached mode* (berjalan di background, terminal tidak tertahan)
- `--build` → Paksa rebuild image dari Dockerfile (agar perubahan kode terbaru ikut masuk)

**Output yang diharapkan:**
```
 Container tka-db       Started
 Container tka-backend  Started
 Container tka-frontend Started
```

---

### 2.2 — Verifikasi Status Container

Cek apakah ketiga container berjalan normal:

```powershell
docker compose ps
```

**Output yang diharapkan:**

| NAME | STATUS | PORTS |
|---|---|---|
| `tka-db` | `Up X seconds (healthy)` | `0.0.0.0:5432->5432/tcp` |
| `tka-backend` | `Up X seconds` | `0.0.0.0:3000->3000/tcp` |
| `tka-frontend` | `Up X seconds` | `0.0.0.0:8080->80/tcp` |

> ⚠️ **Perhatikan** kolom STATUS pada `tka-db`. Harus ada kata `(healthy)` karena backend menunggu database sehat sebelum mulai (`depends_on: condition: service_healthy`). Jika masih `(health: starting)`, tunggu 10–15 detik lalu cek ulang.

---

### 2.3 — Lihat Log Container (Opsional / Debugging)

Jika ada container yang tidak mau naik, gunakan perintah ini untuk melihat error:

```powershell
# Lihat log semua container sekaligus
docker compose logs

# Lihat log container tertentu saja
docker compose logs backend
docker compose logs db
docker compose logs frontend
```

---

## 3. Uji Endpoint Backend (API)

Setelah container berjalan, uji semua endpoint API backend. Jalankan blok perintah ini satu per satu di PowerShell.

### 3.1 — Health Check Backend

Memverifikasi bahwa server Express.js berhasil start dan merespon request HTTP.

```powershell
Invoke-WebRequest -Uri "http://localhost:3000" -UseBasicParsing
```

**Output yang diharapkan:**
```json
{"message":"Backend is UP and RUNNING!"}
```
**StatusCode: 200** ✅

---

### 3.2 — Register User Baru

Menguji endpoint registrasi. Perintah ini akan membuat user baru di database PostgreSQL, dengan password yang sudah di-hash menggunakan `bcryptjs`.

```powershell
Invoke-WebRequest -Uri "http://localhost:3000/api/register" `
  -Method POST `
  -Body '{"username":"testuser1","password":"password123"}' `
  -ContentType "application/json" `
  -UseBasicParsing
```

**Output yang diharapkan:**
```json
{"message":"Registrasi berhasil! Silakan login."}
```
**StatusCode: 201** ✅

> 💡 **Catatan**: Jika dijalankan dua kali dengan username yang sama, akan muncul HTTP `409 Conflict` dengan pesan `"Username sudah digunakan!"` — ini adalah behavior yang **benar**.

---

### 3.3 — Login User

Menguji endpoint login. Jika berhasil, server akan mengembalikan **JWT Token** yang nantinya dipakai untuk autentikasi request selanjutnya.

```powershell
Invoke-WebRequest -Uri "http://localhost:3000/api/login" `
  -Method POST `
  -Body '{"username":"testuser1","password":"password123"}' `
  -ContentType "application/json" `
  -UseBasicParsing
```

**Output yang diharapkan:**
```json
{
  "message": "Login berhasil!",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...."
}
```
**StatusCode: 200** ✅

---

### 3.4 — Login dengan Password Salah (Uji Keamanan)

Memastikan sistem **menolak** percobaan login dengan password yang salah. Ini membuktikan validasi bcrypt berjalan dengan benar.

```powershell
Invoke-WebRequest -Uri "http://localhost:3000/api/login" `
  -Method POST `
  -Body '{"username":"testuser1","password":"passwordsalah"}' `
  -ContentType "application/json" `
  -UseBasicParsing
```

**Output yang diharapkan:**
```
Invoke-WebRequest : The remote server returned an error: (401) Unauthorized.
```
**StatusCode: 401** ✅ — Ini adalah hasil yang **BENAR** (bukan error sistem, tapi penolakan yang disengaja).

---

### 3.5 — Jalankan Semua Tes Sekaligus (Script Otomatis)

Salin dan jalankan seluruh blok ini di PowerShell untuk pengujian komprehensif dalam satu kali jalan:

```powershell
Write-Host "============================================" -ForegroundColor Cyan
Write-Host "  PENGUJIAN FASE 1 - LOCAL DOCKER TEST" -ForegroundColor Cyan
Write-Host "============================================" -ForegroundColor Cyan

# --- Test 1: Backend Health ---
Write-Host "`n[1/5] Backend Health Check (GET /)..." -ForegroundColor Yellow
$be = Invoke-WebRequest -Uri "http://localhost:3000" -UseBasicParsing -TimeoutSec 10
Write-Host "      Status : $($be.StatusCode)" -ForegroundColor Green
Write-Host "      Response: $($be.Content)" -ForegroundColor Green

# --- Test 2: Frontend ---
Write-Host "`n[2/5] Frontend Nginx Check (GET :8080)..." -ForegroundColor Yellow
$fe = Invoke-WebRequest -Uri "http://localhost:8080" -UseBasicParsing -TimeoutSec 10
Write-Host "      Status : $($fe.StatusCode)" -ForegroundColor Green
Write-Host "      Content-Type: $($fe.Headers['Content-Type'])" -ForegroundColor Green

# --- Test 3: Register ---
Write-Host "`n[3/5] Register User Baru (POST /api/register)..." -ForegroundColor Yellow
try {
    $reg = Invoke-WebRequest -Uri "http://localhost:3000/api/register" -Method POST `
        -Body '{"username":"auto_testuser","password":"pass123"}' `
        -ContentType "application/json" -UseBasicParsing -TimeoutSec 10
    Write-Host "      Status : $($reg.StatusCode)" -ForegroundColor Green
    Write-Host "      Response: $($reg.Content)" -ForegroundColor Green
} catch [System.Net.WebException] {
    $reader = New-Object System.IO.StreamReader($_.Exception.Response.GetResponseStream())
    Write-Host "      Status : $([int]$_.Exception.Response.StatusCode)" -ForegroundColor Yellow
    Write-Host "      Response: $($reader.ReadToEnd())" -ForegroundColor Yellow
}

# --- Test 4: Login ---
Write-Host "`n[4/5] Login User (POST /api/login)..." -ForegroundColor Yellow
try {
    $login = Invoke-WebRequest -Uri "http://localhost:3000/api/login" -Method POST `
        -Body '{"username":"auto_testuser","password":"pass123"}' `
        -ContentType "application/json" -UseBasicParsing -TimeoutSec 10
    $json = $login.Content | ConvertFrom-Json
    Write-Host "      Status : $($login.StatusCode)" -ForegroundColor Green
    Write-Host "      Message: $($json.message)" -ForegroundColor Green
    Write-Host "      Token  : $($json.token.Substring(0,30))..." -ForegroundColor Green
} catch [System.Net.WebException] {
    $reader = New-Object System.IO.StreamReader($_.Exception.Response.GetResponseStream())
    Write-Host "      Status : $([int]$_.Exception.Response.StatusCode) ERROR" -ForegroundColor Red
    Write-Host "      Response: $($reader.ReadToEnd())" -ForegroundColor Red
}

# --- Test 5: Login salah ---
Write-Host "`n[5/5] Login Password Salah - Harus Ditolak (POST /api/login)..." -ForegroundColor Yellow
try {
    Invoke-WebRequest -Uri "http://localhost:3000/api/login" -Method POST `
        -Body '{"username":"auto_testuser","password":"salah_banget"}' `
        -ContentType "application/json" -UseBasicParsing -TimeoutSec 10 | Out-Null
    Write-Host "      Status : 200 - PERINGATAN! Seharusnya ditolak!" -ForegroundColor Red
} catch [System.Net.WebException] {
    $code = [int]$_.Exception.Response.StatusCode
    if ($code -eq 401) {
        Write-Host "      Status : 401 Unauthorized - BENAR, akses ditolak!" -ForegroundColor Green
    } else {
        Write-Host "      Status : $code - Tidak diharapkan" -ForegroundColor Yellow
    }
}

Write-Host "`n============================================" -ForegroundColor Cyan
Write-Host "  PENGUJIAN SELESAI" -ForegroundColor Cyan
Write-Host "============================================" -ForegroundColor Cyan
```

---

## 4. Uji Frontend di Browser

Buka browser dan akses URL berikut secara manual:

### 4.1 — Halaman Utama Frontend
```
http://localhost:8080
```
**Yang diharapkan**: Halaman login/register tampil dengan benar (form Registrasi dan Login terlihat).

### 4.2 — Backend API (Raw JSON)
```
http://localhost:3000
```
**Yang diharapkan**: Browser menampilkan `{"message":"Backend is UP and RUNNING!"}`.

---

## 5. Verifikasi Koneksi Database

Masuk ke dalam container database dan pastikan tabel `users` berhasil dibuat oleh backend:

```powershell
# Masuk ke dalam container PostgreSQL
docker exec -it tka-db psql -U user_tka -d db_tka

# Setelah masuk ke prompt psql, jalankan query berikut:
\dt

# Output yang diharapkan: ada tabel 'users'
# Lalu cek isi tabel (jika sudah ada registrasi):
SELECT id, username FROM users;

# Keluar dari psql
\q
```

**Output yang diharapkan setelah `\dt`:**
```
        List of relations
 Schema | Name  | Type  |  Owner
--------+-------+-------+----------
 public | users | table | user_tka
```

---

## 6. Uji Ansible — Ping Node VM

> ⚠️ **Bagian ini hanya bisa dijalankan jika VM Multipass sudah aktif** dan IP di `inventory.yml` sudah diisi dengan benar.

### 6.1 — Cek Koneksi ke Semua Node
```bash
ansible all -m ping -i inventory.yml
```

**Output yang diharapkan:**
```json
node_backend | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
node_frontend | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### 6.2 — Jalankan Playbook Instalasi Docker di VM
```bash
ansible-playbook main.yml -i inventory.yml
```

**Output yang diharapkan**: Setiap task berstatus `ok` atau `changed`, tidak ada `failed`.

### 6.3 — Verifikasi Docker Terinstal di VM
```bash
ansible all -m shell -a "docker --version" -i inventory.yml
```

**Output yang diharapkan:**
```
node_backend  | CHANGED | rc=0 >>
Docker version 26.x.x, build xxxxxxx

node_frontend | CHANGED | rc=0 >>
Docker version 26.x.x, build xxxxxxx
```

---

## 7. Cleanup (Bersihkan Container)

Setelah pengujian selesai, gunakan perintah berikut untuk mematikan dan menghapus semua container:

```powershell
# Hentikan dan hapus container + network
docker compose down

# Hentikan, hapus container + network + volume database (data hilang)
docker compose down -v
```

**Kapan pakai `-v`?**
- Gunakan `down -v` jika ingin **reset total** (data database ikut terhapus, berguna untuk uji ulang dari awal).
- Gunakan `down` saja jika hanya ingin **mematikan** container tanpa kehilangan data.

---

## 8. Checklist Hasil Pengujian

Centang setiap item setelah pengujian berhasil dijalankan:

### 🐳 Docker Compose (Lokal)
- [ ] `docker compose up -d --build` berhasil tanpa error
- [ ] `tka-db` berstatus **Up (healthy)**
- [ ] `tka-backend` berstatus **Up**
- [ ] `tka-frontend` berstatus **Up**

### 🔌 Endpoint API Backend
- [ ] `GET /` → HTTP `200`, response `"Backend is UP and RUNNING!"`
- [ ] `POST /api/register` → HTTP `201`, user berhasil dibuat
- [ ] `POST /api/login` (benar) → HTTP `200`, JWT token diterima
- [ ] `POST /api/login` (salah) → HTTP `401`, akses ditolak

### 🌐 Frontend
- [ ] `http://localhost:8080` dapat diakses di browser
- [ ] Form Registrasi dan Login tampil dengan benar

### 🗄️ Database
- [ ] Tabel `users` berhasil dibuat di PostgreSQL
- [ ] Data user hasil registrasi tersimpan di tabel

### ⚙️ Ansible (Jika VM Aktif)
- [ ] `ansible all -m ping` → semua node `pong`
- [ ] `ansible-playbook main.yml` → tidak ada `failed`
- [ ] Docker berhasil terinstal di kedua VM

---

> 📝 **Catatan**: Semua pengujian di bagian 2–5 sudah diverifikasi berhasil pada environment lokal (Windows + Docker Desktop) pada **2 Mei 2026**.
