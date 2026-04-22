### CEK KESEHATAN HARDISK DAN SSD
Menggunakan Smartmoon Tool
1. Install
```
apt update
apt install smartmontools -y
```

2. Cek Nama Hardisknya
```
lsblk
```

3. SSD
```
smartctl -H /dev/nvme0n1
smartctl -A /dev/nvme0n1
```

4. HARDISK
```
smartctl -H /dev/sda
```
smartctl -A /dev/sda
```
