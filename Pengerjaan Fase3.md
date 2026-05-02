# Dokumentasi Pengerjaan Fase 3 - Deployment Frontend (Ansimbel & Multipass WSL)

Dokumen ini berisi detail teknis pengerjaan deployment backend menggunakan Ansible dan Docker oleh **Praktikan 3 (Dimas Satya A)**.

---

## 1. Menyiapkan Role Baru untuk Deploy Frontend

Pada fase ini dibuat role baru khusus untuk deployment frontend, yaitu:

```text
roles/frontend_deployment/
```

Struktur role yang dibuat:

```text
roles/frontend_deployment/
├── tasks/
│   └── main.yml
└── templates/
    ├── config.js.j2
    └── docker-compose.yml.j2
```

Role ini digunakan untuk mengatur seluruh proses deployment frontend, mulai dari membuka port, menyalin source code frontend, membuat file konfigurasi, menjalankan Docker Compose, hingga melakukan health check.

---

## 2. Menyiapkan Variabel Frontend

Variabel frontend diletakkan pada file:

```text
group_vars/frontend_nodes/vars.yml
```

Isi file:

```yaml
---
frontend_port: 8080
backend_url: "http://localhost:3000"
```

Keterangan:

- `frontend_port` digunakan untuk menentukan port frontend yang dibuka pada node frontend.
- `backend_url` digunakan oleh frontend untuk menentukan alamat backend.

Pada kondisi ideal, `backend_url` dapat dibuat berdasarkan Ansible inventory menggunakan `hostvars`, contohnya:

```yaml
backend_url: "http://{{ hostvars[groups['backend_nodes'][0]].ansible_host }}:{{ hostvars[groups['backend_nodes'][0]].backend_port }}"
```

Namun pada pengerjaan ini digunakan environment Windows → WSL → Multipass/VirtualBox, sehingga terjadi double layer virtualization. Akibatnya, browser utama Windows tidak dapat langsung mengakses backend VM menggunakan IP backend dari inventory.

Berdasarkan catatan pada soal, untuk kondisi Multipass dalam WSL diperbolehkan melakukan hardcode `backend_url`. Oleh karena itu, digunakan:

```yaml
backend_url: "http://localhost:3000"
```

Agar backend dapat diakses melalui `localhost:3000`, digunakan SSH tunnel dari Windows ke node backend.

---

## 3. Menyiapkan Dockerfile Frontend

Source code frontend berada pada folder:

```text
Resource Soal Modul 3/frontend/
```

Dockerfile frontend berada pada file:

```text
Resource Soal Modul 3/frontend/Dockerfile
```

Isi Dockerfile:

```dockerfile
FROM nginx:alpine

COPY . /usr/share/nginx/html

EXPOSE 80
```

Dockerfile ini menggunakan image `nginx:alpine` untuk menyajikan file static frontend. Seluruh file frontend disalin ke direktori default Nginx, yaitu:

```text
/usr/share/nginx/html
```

Port yang dibuka di dalam container adalah port `80`.

---

## 4. Menyiapkan Template Jinja2

Pada role frontend dibuat dua template Jinja2, yaitu `config.js.j2` dan `docker-compose.yml.j2`.

### Template `config.js.j2`

File:

```text
roles/frontend_deployment/templates/config.js.j2
```

Isi:

```js
const API_BASE_URL = '{{ backend_url }}';
```

Template ini digunakan untuk membuat file `config.js` pada node frontend. Nilai `backend_url` diambil dari variabel yang ada pada `group_vars/frontend_nodes/vars.yml`.

Hasil file `config.js` pada node frontend:

```js
const API_BASE_URL = 'http://localhost:3000';
```

File ini digunakan oleh frontend untuk mengirim request ke backend.

### Template `docker-compose.yml.j2`

File:

```text
roles/frontend_deployment/templates/docker-compose.yml.j2
```

Isi:

```yaml
services:
  frontend:
    build: ./frontend
    container_name: tka-frontend
    ports:
      - "{{ frontend_port }}:80"
    restart: unless-stopped
```

Template ini digunakan untuk menjalankan service frontend menggunakan Docker Compose. Port host yang digunakan berasal dari variabel `frontend_port`, lalu diarahkan ke port `80` pada container.

---

## 5. Membuat Playbook pada Role Frontend

Task utama role frontend berada pada file:

```text
roles/frontend_deployment/tasks/main.yml
```

Isi task:

```yaml
---
- name: Allow frontend port {{ frontend_port }} on UFW
  ufw:
    rule: allow
    port: "{{ frontend_port }}"
    proto: tcp

- name: Create frontend application directory
  file:
    path: /opt/frontend
    state: directory
    mode: '0755'

- name: Copy frontend source code to node
  copy:
    src: "{{ playbook_dir }}/Resource Soal Modul 3/frontend/"
    dest: /opt/frontend/frontend/
    mode: '0644'

- name: Template config.js
  template:
    src: config.js.j2
    dest: /opt/frontend/frontend/config.js
    mode: '0644'

- name: Template docker-compose.yml
  template:
    src: docker-compose.yml.j2
    dest: /opt/frontend/docker-compose.yml
    mode: '0644'

- name: Build and run frontend with Docker Compose
  shell: docker compose up -d --build
  args:
    chdir: /opt/frontend

- name: Health check frontend
  uri:
    url: "http://localhost:{{ frontend_port }}/"
    method: GET
    status_code: 200
  register: frontend_health
  retries: 12
  delay: 5
  until: frontend_health.status == 200
```

Task tersebut melakukan beberapa hal berikut:

1. Membuka port frontend menggunakan UFW.
2. Membuat direktori `/opt/frontend`.
3. Menyalin source code frontend ke node frontend.
4. Membuat file `config.js` dari template Jinja2.
5. Membuat file `docker-compose.yml` dari template Jinja2.
6. Menjalankan frontend menggunakan Docker Compose.
7. Melakukan health check frontend menggunakan modul `uri`.

---

## 6. Modifikasi Playbook Utama

Playbook utama berada pada file:

```text
main.yml
```

Pada file tersebut ditambahkan play untuk menjalankan role frontend pada group `frontend_nodes`.

```yaml
- name: Deploy Frontend pada frontend_nodes
  hosts: frontend_nodes
  become: yes
  roles:
    - frontend_deployment
```

Dengan konfigurasi tersebut, role `frontend_deployment` hanya dijalankan pada host yang berada di dalam group `frontend_nodes`.

---

## 7. Menjalankan Deployment dan Verifikasi

Deployment dijalankan menggunakan command berikut dari WSL:

```bash
ansible-playbook -i inventory.yml main.yml
```

Deployment dianggap berhasil apabila hasil recap menunjukkan tidak ada host yang gagal.

Contoh hasil akhir:

```text
node_backend               : ok=20   changed=15   unreachable=0    failed=0
node_frontend              : ok=20   changed=15   unreachable=0    failed=0
```

Untuk memastikan container frontend berjalan, digunakan command:

```bash
ansible -i inventory.yml node_frontend -m shell -a "sudo docker ps"
```

Output yang diharapkan menunjukkan container `tka-frontend` berjalan dan port `8080` diarahkan ke port `80` container.

Contoh:

```text
CONTAINER ID   IMAGE               PORTS                  NAMES
xxxx           frontend-frontend   0.0.0.0:8080->80/tcp   tka-frontend
```

Untuk mengakses frontend, browser dibuka pada alamat:

```text
http://localhost:8080
```

Karena backend diakses dari browser Windows melalui SSH tunnel, digunakan command berikut di PowerShell:

```powershell
ssh -i "$env:USERPROFILE\multipass_key" -L 3000:127.0.0.1:3000 ubuntu@127.0.0.1 -p 57808 -N
```

Terminal tersebut harus tetap terbuka selama pengujian aplikasi.

Backend dapat dicek melalui:

```powershell
curl.exe http://localhost:3000/
```

Output yang diharapkan:

```json
{"message":"Backend is UP and RUNNING!"}
```

---

## 8. Pengujian Fitur Aplikasi

Setelah frontend berhasil dibuka melalui browser, dilakukan pengujian fitur sesuai instruksi soal.

### 1. Register dengan user baru

Pengujian pertama dilakukan dengan register menggunakan username dan password baru.

Hasil yang diharapkan:

```text
Registrasi berhasil! Silakan login.
```

### 2. Register lagi dengan user yang sama

Pengujian kedua dilakukan dengan register menggunakan username yang sama seperti sebelumnya.

Hasil yang diharapkan:

```text
Username sudah digunakan!
```

### 3. Login dengan password salah

Pengujian ketiga dilakukan dengan login menggunakan username benar tetapi password salah.

Hasil yang diharapkan:

```text
Password salah!
```

### 4. Login berhasil

Pengujian terakhir dilakukan dengan login menggunakan username dan password yang benar.

Hasil yang diharapkan:

```text
Login berhasil!
```

Dengan hasil tersebut, frontend berhasil dijalankan menggunakan Docker Compose melalui Ansible dan berhasil terhubung ke backend.