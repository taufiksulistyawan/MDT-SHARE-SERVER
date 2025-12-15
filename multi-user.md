# Tujuan akhir:

* `/mnt/share/editor` → **diakses sebagai useredt**
* `/mnt/share/ctp` → **diakses sebagai userec**
* Samba **tidak error permission**
* Windows bisa paste file normal

---

# 🧩 KONSEP SINGKAT (biar jelas)

* Disk fisik NTFS tetap **dimiliki userec** (tidak diubah)
* `bindfs` membuat **mount virtual** dengan owner palsu
* Samba **HANYA pakai mount virtual**
* Ownership jadi seolah-olah ext4

---

# ✅ LANGKAH 1 — Install bindfs

```bash
sudo apt update
sudo apt install bindfs -y
```

Cek:

```bash
bindfs --version
```

---

# ✅ LANGKAH 2 — Buat mount point virtual

```bash
sudo mkdir -p /mnt/editor
sudo mkdir -p /mnt/ctp
```

Ini **bukan folder data**, hanya tampilan virtual.

---

# ✅ LANGKAH 3 — Mount bindfs (INI INTI SOLUSI)

### Editor → useredt

```bash
sudo bindfs \
  --force-user=useredt \
  --force-group=useredt \
  --create-for-user=useredt \
  --create-for-group=useredt \
  --perms=0775 \
  /mnt/share/editor /mnt/editor
```

### CTP → userec

```bash
sudo bindfs \
  --force-user=userec \
  --force-group=userec \
  --create-for-user=userec \
  --create-for-group=userec \
  --perms=0775 \
  /mnt/share/ctp /mnt/ctp
```

---

# ✅ LANGKAH 4 — Verifikasi (PENTING)

```bash
ls -ld /mnt/editor
ls -ld /mnt/ctp
```

HARUS terlihat:

```
/mnt/editor → owner useredt
/mnt/ctp    → owner userec
```

Kalau ini benar → **permission problem selesai 100%**

---

# ✅ LANGKAH 5 — Ubah Samba agar pakai mount virtual

Edit:

```bash
sudo nano /etc/samba/smb.conf
```

### 🔁 GANTI konfigurasi lama menjadi:

```
[editor]
   path = /mnt/editor
   browseable = yes
   read only = no
   valid users = useredt
   force user = useredt
   create mask = 0775
   directory mask = 0775

[ctp]
   path = /mnt/ctp
   browseable = yes
   read only = no
   valid users = userec
   force user = userec
   create mask = 0775
   directory mask = 0775
```

Restart Samba:

```bash
sudo systemctl restart smbd
```

---

# ✅ LANGKAH 6 — Windows (WAJIB)

Di Windows **CMD (Run as Administrator)**:

```cmd
net use * /delete /y
```

Hapus juga:
Control Panel → Credential Manager → Windows Credentials

---

# 🔌 Connect ulang:

```
\\IP-SERVER\editor   → login useredt
\\IP-SERVER\ctp      → login userec
```

---

# 🎯 HASIL AKHIR (INI YANG ANDA DAPAT)

✔ Bisa paste file
✔ Bisa buat folder
✔ Tidak ada “You need permission”
✔ Multi user WALAU disk NTFS
✔ Samba stabil

---
