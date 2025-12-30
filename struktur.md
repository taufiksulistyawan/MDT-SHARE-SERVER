# 1. STRUKTUR FINAL YANG AKAN KITA BANGUN
```
[NVMe]
 └─ Ubuntu Server + Samba + CUPS + Cloudflare + MinIO (opsional)

[HDD]
 ├─ /mnt/share/ctp
 ├─ /mnt/share/admin
 └─ /mnt/share/quarantine   # untuk antivirus
```
# 2. AKSES FILE SHARING
```
\\192.168.20.97
```
```
\\mdt-share
```
# 3. Cockpit
Monitoring CPU, RAM, Disk, SMART HDD, Log, Network, dan Container.
```
https://192.168.20.97:9090/
```
# 4. CUPS
Sharing printer via samba
```
http://192.168.20.97:631/
```
# 5. Akses
CTP
```
userec
@ec$1234
```
Editor
```
useredt
@ec$1234
```
# 5. Scan
```
clamscan -r /mnt/share/ctp --move=/mnt/share/quarantine
```
```
clamscan -r /mnt/share/editor --move=/mnt/share/quarantine
```
