Berikut **rancangan lengkap dan terstruktur** untuk membangun **file sharing + printer sharing** dengan **server Ubuntu**, dilengkapi **keamanan data, perlindungan virus**, serta opsi **upgrade MinIO dan Cloudflare Tunnel** agar bisa diakses online secara aman.

---

# ✅ **1. Rancangan Infrastruktur**

## **1.1 Topologi Dasar**

```
           [Client Windows]
                 |
           [Switch / LAN]
                 |
        ┌───────────────────┐
        │   Ubuntu Server   │
        │ (Samba + CUPS)    │
        └───────────────────┘
                 |
       (Opsional Upgrade)
 ┌──────────────┴─────────────────┐
 │                                  │
[MinIO Object Storage]    [Cloudflare Tunnel]
```

---

# ✅ **2. Sharing File Dengan Ubuntu + Samba**

## **2.1 Instalasi Samba**

```bash
sudo apt update
sudo apt install samba -y
```

## **2.2 Buat Folder Sharing**

Misal 3 folder:

* `/srv/share-public`
* `/srv/share-dept`
* `/srv/share-admin`

```bash
sudo mkdir -p /srv/share-public
sudo mkdir -p /srv/share-dept
sudo mkdir -p /srv/share-admin
```

## **2.3 Izin Folder**

```bash
sudo chown -R nobody:nogroup /srv/share-public
sudo chmod -R 777 /srv/share-public
```

Folder yang sensitif:

```bash
sudo chown -R adminuser:adminuser /srv/share-admin
sudo chmod -R 770 /srv/share-admin
```

## **2.4 Konfigurasi Samba**

Edit file:

```bash
sudo nano /etc/samba/smb.conf
```

Tambahkan:

### 📁 **Share Public**

```
[public]
    path = /srv/share-public
    read only = no
    browsable = yes
    guest ok = yes
```

### 📁 **Share Dept (Hanya user tertentu)**

```
[dept]
    path = /srv/share-dept
    valid users = dept1, dept2
    read only = no
```

### 📁 **Share Admin**

```
[admin]
    path = /srv/share-admin
    valid users = adminuser
    read only = no
    force group = admin
```

## **2.5 Buat user Samba**

```bash
sudo useradd dept1
sudo smbpasswd -a dept1
```

## **2.6 Restart Samba**

```bash
sudo systemctl restart smbd
```

## **2.7 Koneksikan di Windows**

Di Windows Explorer:

```
\\IP-UBUNTU\public
```

Atau:

```
net use Z: \\IP-UBUNTU\dept /user:dept1
```

---

# ✅ **3. Sharing Printer Dengan Ubuntu Server + CUPS**

## **3.1 Install CUPS**

```bash
sudo apt install cups -y
```

## **3.2 Izinkan akses LAN**

```bash
sudo cupsctl --remote-any
sudo systemctl restart cups
```

## **3.3 Masuk ke Web Interface**

Buka dari client Windows:

```
http://IP-UBUNTU:631
```

## **3.4 Tambahkan printer**

* Add Printer
* Pilih driver
* Centang "Share This Printer"

## **3.5 Install di Windows**

Windows → **Add Printer** → **Add via Network**
Atau manual:

```
\\IP-UBUNTU\NamaPrinter
```

---

# ✅ **4. Keamanan Data & Keamanan Virus**

## 🔐 **4.1 Keamanan Akses**

1. Gunakan **ACL** untuk membatasi user.
2. Matikan guest share jika tidak dibutuhkan.
3. Pisahkan folder public vs sensitive.

## 🛡️ **4.2 Firewall UFW**

```bash
sudo ufw allow samba
sudo ufw allow 631/tcp    # CUPS
sudo ufw enable
```

## 🚫 **4.3 Antivirus ClamAV di Server**

```bash
sudo apt install clamav clamav-daemon -y
```

Scan rutin:

```bash
sudo clamscan -r /srv/share --move=/srv/quarantine
```

## 🔄 **4.4 Jadwalkan Scan Otomatis**

```bash
sudo crontab -e
```

Tambah:

```
0 9 * * * clamscan -r /srv/share --move=/srv/quarantine
```

## 🧱 **4.5 Hardening Ubuntu**

* Disable root login
* Backup Samba config
* Monitoring via `fail2ban`

---

# ✅ **5. Upgrade ke MinIO (Object Storage)**

## **5.1 Alasan Upgrade**

* Bisa jadi cloud storage internal
* Public link
* Mirip AWS S3
* Backup otomatis

## **5.2 Instalasi MinIO**

```bash
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
sudo mv minio /usr/local/bin/
```

Buat folder data:

```bash
sudo mkdir /mnt/minio
```

Jalankan:

```bash
minio server /mnt/minio --console-address ":9001"
```

Akses:

```
http://IP:9001
```

---

# ✅ **6. Upgrade Online Dengan Cloudflare Tunnel**

## **Tujuan**

* Sharing file via dashboard MinIO online
* Akses remote printer admin
* Aman karena HTTPS langsung dari Cloudflare

## **6.1 Instal Cloudflare Tunnel**

```bash
curl -fsSL https://developers.cloudflare.com/cloudflared/install.sh | sudo bash
```

Login:

```bash
cloudflared tunnel login
```

## **6.2 Buat Tunnel**

```bash
cloudflared tunnel create share-server
```

## **6.3 Tambah route untuk MinIO & Samba Web Admin**

```bash
cloudflared tunnel route dns share-server minio.domainkamu.com
```

## **6.4 Konfigurasi ingress**

Edit:

```bash
sudo nano ~/.cloudflared/config.yml
```

Isi:

```
tunnel: share-server
credentials-file: /root/.cloudflared/share-server.json

ingress:
  - hostname: minio.domainkamu.com
    service: http://localhost:9001
  - service: http_status:404
```

Jalankan:

```bash
cloudflared tunnel run share-server
```

---

# ✅ **7. Alur Implementasi (Step-by-Step)**

### **Step 1: Persiapan Server Ubuntu**

* Update paket
* Atur IP static
* Aktifkan firewall

### **Step 2: Build Sharing File**

* Install Samba
* Buat folder & user
* Konfigurasi share
* Test di Windows

### **Step 3: Build Sharing Printer**

* Install CUPS
* Tambahkan printer
* Share printer ke Windows

### **Step 4: Hardening & Antivirus**

* Install ClamAV
* Setup cron
* Setup fail2ban / firewall

### **Step 5: Implementasi Backup & Versioning**

* MinIO sebagai object storage
* Samba → backup ke MinIO

### **Step 6: Akses Online Aman**

* Instal Cloudflare Tunnel
* Buat subdomain
* Proteksi via Zero Trust

---

# Siap dibuatkan diagram visual topologinya juga?

Atau mau saya buatkan tutorial versi **script otomatis (bash installer)**?
