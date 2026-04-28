# Dokumentasi Pengerjaan Fase 1 - Setup Infra & Docker (Ansible)

Dokumen ini berisi detail teknis pengerjaan infrastruktur dasar menggunakan Ansible dan Docker oleh **Praktikan 1 (Rayhan Agnan Kusuma)**.

## 1. Persiapan Inventory (`inventory.yml`)
Kami mendaftarkan dua node VM (Multipass) dan membaginya ke dalam dua group berbeda: `backend_nodes` dan `frontend_nodes`.

```yaml
all:
  children:
    backend_nodes:
      hosts:
        node_backend:
          ansible_host: 192.168.x.x # Ganti dengan IP VM Backend
          ansible_user: ubuntu
    frontend_nodes:
      hosts:
        node_frontend:
          ansible_host: 192.168.x.x # Ganti dengan IP VM Frontend
          ansible_user: ubuntu
```

## 2. Struktur Role Ansible (`roles/docker_setup`)
Kami membuat role khusus untuk instalasi Docker Engine agar manajemen konfigurasi lebih rapi dan reusable.

Struktur folder:
- `roles/docker_setup/tasks/main.yml`: Berisi langkah-langkah instalasi.
- `roles/docker_setup/handlers/main.yml`: Berisi trigger untuk restart service.

## 3. Playbook Instalasi & Firewall
Playbook ini menjalankan role `docker_setup` dan mengonfigurasi firewall (UFW) untuk keamanan.

### Isi `tasks/main.yml` (Ringkasan):
1. **Update Repository**: Memastikan list paket OS terbaru.
2. **Instalasi Docker Engine**: Menginstal `docker-ce`, `docker-ce-cli`, dan `containerd.io`.
3. **Konfigurasi Firewall (UFW)**:
   - `ufw allow 22/tcp`: Mengizinkan akses SSH.
   - `ufw default deny`: Menutup port lainnya secara default.
   - `ufw enable`: Mengaktifkan firewall.
4. **Setup User Docker**: Menambahkan user ke group `docker` agar bisa menjalankan perintah tanpa `sudo`.

## 4. Playbook Utama (`main.yml`)
Digunakan untuk menjalankan seluruh konfigurasi ke semua node.

```yaml
- name: Setup Docker on all nodes
  hosts: all
  become: yes
  roles:
    - docker_setup
```

## 5. Verifikasi & Pengujian
Setelah Ansible dijalankan, dilakukan pengujian manual:
1. **Ping Test**: `ansible all -m ping -i inventory.yml` (Memastikan koneksi ke semua VM).
2. **SSH Test**: Mencoba masuk ke masing-masing node secara manual.
3. **Hello World**: Menjalankan `docker run hello-world` di masing-masing VM untuk memastikan Docker Engine berjalan sempurna.

---

## 🐳 Dockerization Aplikasi (Local Test)
Sebelum di-deploy via Ansible ke VM, aplikasi telah diuji secara lokal menggunakan Docker Compose:
- **Backend Dockerfile**: Menggunakan `node:18-alpine`.
- **Frontend Dockerfile**: Menggunakan `nginx:alpine`.
- **Docker Compose**: Mengorkestrasi Backend, Frontend, dan Database PostgreSQL.

> Cara menjalankan local test: `docker compose up -d`
