# Dokumentasi Pengerjaan Fase 1 - Setup Infra & Docker

Dokumen ini menjelaskan langkah-langkah yang telah dilakukan untuk menyiapkan infrastruktur Docker dan cara menjalankan aplikasi secara manual.

## 1. Langkah Pengerjaan Dockerization

### A. Backend (`Resource Soal Modul 3/backend/Dockerfile`)
Dibuat menggunakan image `node:18-alpine` untuk efisiensi ukuran.
- Menentukan working directory di `/app`.
- Menyalin `package.json` dan menginstall dependensi.
- Menyalin seluruh source code.
- Mengekspos port `3000`.

### B. Frontend (`Resource Soal Modul 3/frontend/Dockerfile`)
Dibuat menggunakan image `nginx:alpine`.
- Menyalin file `index.html` dan `config.js` ke direktori default Nginx (`/usr/share/nginx/html`).
- Mengekspos port `80`.

### C. Orchestration (`docker-compose.yml`)
Menghubungkan tiga layanan utama:
1. **db**: PostgreSQL sebagai database.
2. **backend**: Node.js API yang bergantung pada database.
3. **frontend**: Nginx sebagai web server untuk UI.

---

## 2. Cara Menjalankan Menggunakan Docker (Rekomendasi)

Jika Docker sudah terinstall, jalankan perintah berikut di direktori utama project:

```powershell
# Membangun image dan menjalankan container di background
docker compose up -d --build

# Melihat status container
docker ps

# Melihat log backend (untuk debugging)
docker logs -f tka-backend
```

Aplikasi dapat diakses di:
- **Frontend**: `http://localhost:8080`
- **Backend Health Check**: `http://localhost:3000`

---

## 3. Cara Menjalankan Secara Manual (Tanpa Docker)

Jika ingin menjalankan secara manual di sistem lokal, ikuti langkah berikut:

### A. Persiapan Database
Pastikan PostgreSQL sudah terinstall di komputer kamu.
1. Buat database baru bernama `db_tka`.
2. Pastikan user dan password sesuai dengan yang diinginkan.

### B. Menjalankan Backend
1. Buka terminal di folder `Resource Soal Modul 3/backend`.
2. Install dependensi:
   ```powershell
   npm install
   ```
3. Set environment variables (Windows PowerShell):
   ```powershell
   $env:DB_HOST="localhost"
   $env:DB_USER="user_kamu"
   $env:DB_PASSWORD="password_kamu"
   $env:DB_NAME="db_tka"
   $env:JWT_SECRET="supersecretvibe"
   $env:PORT="3000"
   ```
4. Jalankan backend:
   ```powershell
   npm start
   ```

### C. Menjalankan Frontend
1. Pastikan file `Resource Soal Modul 3/frontend/config.js` berisi:
   ```javascript
   const API_BASE_URL = 'http://localhost:3000';
   ```
2. Karena frontend hanyalah file HTML statis, kamu bisa membukanya langsung dengan cara:
   - Klik kanan `index.html` -> **Open with Browser**.
   - Atau gunakan extension VS Code seperti **Live Server**.

---

## Ringkasan Port
| Service | Port (Docker) | Port (Local Manual) |
| :--- | :--- | :--- |
| Frontend | 8080 | Tergantung Live Server (biasanya 5500) |
| Backend | 3000 | 3000 |
| Database | 5432 | 5432 |
