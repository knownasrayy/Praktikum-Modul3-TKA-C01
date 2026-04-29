# Dokumentasi Pengerjaan Fase 2 - Deployment Backend (Ansible)

Dokumen ini berisi detail teknis pengerjaan deployment backend menggunakan Ansible dan Docker oleh **Praktikan 2 (Zahra Hafizhah)**.

---

## 1. Persiapan Variabel Group (`group_vars/backend_nodes/`)

Kami memisahkan variabel menjadi dua file sesuai tingkat sensitifitasnya, yang diletakkan di folder `group_vars/backend_nodes/` khusus untuk host dalam group `backend_nodes`.

### `vars.yml` — Variabel Non-Sensitif
```yaml
db_name: db_tka
db_username: user_tka
backend_port: 3000
```

### `vault.yml` — Variabel Sensitif
File ini menyimpan variabel yang bersifat rahasia. Pada implementasi ini menggunakan plaintext untuk keperluan demo, namun pada kondisi produksi seharusnya dienkripsi menggunakan perintah:
```bash
ansible-vault encrypt group_vars/backend_nodes/vault.yml
```

Isi variabel sensitif:
```yaml
db_password: password_tka
jwt_secret: supersecretvibe2026
```

---

## 2. Struktur Role Ansible (`roles/backend_deployment`)

Kami membuat role khusus untuk deployment backend agar manajemen konfigurasi lebih rapi dan reusable.

Struktur folder:
- `roles/backend_deployment/tasks/main.yml` — Berisi langkah-langkah deployment backend.
- `roles/backend_deployment/templates/.env.j2` — Template Jinja2 untuk file environment backend.
- `roles/backend_deployment/templates/docker-compose.yml.j2` — Template Jinja2 untuk Docker Compose.

---

## 3. Dockerfile Backend

Backend menggunakan image `node:18-alpine` sesuai dengan source code yang diberikan (Node.js + Express). Dockerfile sudah tersedia di `Resource Soal Modul 3/backend/Dockerfile`:

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

---

## 4. Template Jinja2

### `templates/.env.j2`
Template ini akan dirender oleh Ansible menggunakan variabel dari `group_vars/backend_nodes/` dan menghasilkan file `.env` di dalam node backend.

```
DB_HOST=db
DB_USER={{ db_username }}
DB_PASSWORD={{ db_password }}
DB_NAME={{ db_name }}
JWT_SECRET={{ jwt_secret }}
PORT={{ backend_port }}
```

### `templates/docker-compose.yml.j2`
Template ini mendefinisikan dua service: `db` (PostgreSQL) dan `backend` (Node.js). Service backend menunggu database sehat sebelum mulai berjalan.

```yaml
services:
  db:
    image: postgres:15-alpine
    container_name: tka-db
    environment:
      POSTGRES_USER: {{ db_username }}
      POSTGRES_PASSWORD: {{ db_password }}
      POSTGRES_DB: {{ db_name }}
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U {{ db_username }} -d {{ db_name }}"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  backend:
    build: ./backend
    container_name: tka-backend
    ports:
      - "{{ backend_port }}:{{ backend_port }}"
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

volumes:
  pgdata:
```

---

## 5. Tasks Role (`tasks/main.yml`)

Playbook role `backend_deployment` menjalankan langkah-langkah berikut secara berurutan:

1. **Buka Port Backend via UFW**: Membuka port `3000` agar backend bisa diakses.
2. **Buat Direktori Aplikasi**: Membuat folder `/opt/backend` di dalam VM sebagai working directory.
3. **Copy Source Code**: Menyalin source code backend dari host ke dalam VM di `/opt/backend/backend/`.
4. **Render Template `.env`**: Ansible merender `.env.j2` → `/opt/backend/.env` menggunakan variabel yang sudah didefinisikan.
5. **Render Template `docker-compose.yml`**: Ansible merender `docker-compose.yml.j2` → `/opt/backend/docker-compose.yml`.
6. **Jalankan Docker Compose**: Menjalankan `docker compose up -d --build` untuk build image backend dan menjalankan semua service.
7. **Health Check**: Ansible menggunakan modul `uri` untuk melakukan GET ke `http://localhost:3000/` dan memastikan status `200 OK`.

---

## 6. Modifikasi Playbook Utama (`main.yml`)

Playbook utama ditambahkan satu play baru khusus untuk group `backend_nodes`:

```yaml
- name: Setup Docker on all nodes
  hosts: all
  become: yes
  roles:
    - docker_setup

- name: Deploy Backend pada backend_nodes
  hosts: backend_nodes
  become: yes
  roles:
    - backend_deployment
```

---

## 7. Cara Menjalankan Fase 2

### Prasyarat
- Fase 1 sudah selesai dijalankan (Docker terinstall di semua node).
- Repository sudah di-clone di dalam salah satu node (node-backend).
- Inventory sudah dikonfigurasi dengan `ansible_connection: local`.

### Perintah
```bash
# Masuk ke folder project
cd ~/Praktikum-Modul3-TKA-C01

# Jalankan playbook utama (Fase 1 + Fase 2)
ansible-playbook -i inventory.yml main.yml
```

---

## 8. Verifikasi & Pengujian (Demo)

Setelah playbook selesai, lakukan verifikasi berikut di dalam `node-backend`:

### Cek Container Berjalan
```bash
docker ps
```
Output yang diharapkan (2 container Running):
```
CONTAINER ID   IMAGE              NAMES
xxxx           backend-backend    tka-backend
xxxx           postgres:15-alpine tka-db
```

### Health Check
```bash
curl http://localhost:3000/
```
```json
{"message":"Backend is UP and RUNNING!"}
```

### Register User Baru
```bash
curl -X POST http://localhost:3000/api/register \
  -H "Content-Type: application/json" \
  -d '{"username":"zahra","password":"rahasia123"}'
```
```json
{"message":"Registrasi berhasil! Silakan login."}
```

### Register Duplikat (Harus Error)
```bash
curl -X POST http://localhost:3000/api/register \
  -H "Content-Type: application/json" \
  -d '{"username":"zahra","password":"rahasia123"}'
```
```json
{"message":"Username sudah digunakan!"}
```


