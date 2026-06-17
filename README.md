<h2 align="center">UDP Custom - Installer - ARM[64]<h2>

### Supported OS
- ubuntu 20.04 [arm] above ✅ _(recommended)_

### Install
```
sudo -i
``` 
```
clear; wget --no-check-certificate "https://raw.githubusercontent.com/thoedrit13/UDP-Custom-Installer-arm64/main/udpc-installer.sh" -O udpc-installer.sh && chmod +x udpc-installer.sh && ./udpc-installer.sh
```
```
./udpc-installer.sh --help
```

### Manually Port Config ต้องรีเครื่อง iptable ถึงจะสร้างกฏขึ้นมา reboot ทุกครั้งที่เปลี่ยนการตั้งค่า
```
sudo nano /etc/config.json
```
Add "exclude": "22,53,80,443,1194,2096,8088" etc0

```
{
  "listen": ":36712",
  "max-connections": 1000,
  "max-drop": 10,
  "stream_buffer": 209715200,
  "receive_buffer": 209715200,
  "exclude": "59209",
  "auth": {
    "mode": "passwords"
  }
}
```

```
หลังจากติดตั้งและตั้งค่า config เสร็จ ให้ reboot เครื่อง เพื่อสร้างกฏ iptables ขึ้นมา
เช็คจาก
sudo iptables -t nat -L PREROUTING -n --line-numbers
```

จากนั้น

# สร้าง User ป้องกันการรีโมท 
```
sudo useradd -m -s /bin/false userrr
```

# ตั้งรหัสผ่านให้ User (พิมพ์ 2 รอบ จะไม่แสดงบนจอ)
```
sudo passwd userrr
```
จากนั้น

```
sudo ufw allow 36712/udp
sudo ufw reload
```

sudo nano /etc/config.json

จากนั้น
```
sudo iptables -t nat -A PREROUTING -p udp --dport 1:65535 -j REDIRECT --to-ports 36712
```

ถ้าใช้ wireguard ด้วย ให้เว้น 59209 ไว้
```
sudo iptables -t nat -A PREROUTING -p udp --dport 1:59208 -j REDIRECT --to-ports 36712
sudo iptables -t nat -A PREROUTING -p udp --dport 59210:65535 -j REDIRECT --to-ports 36712
```

จากนั้น สร้าง Service ให้ทำงานเบื้องหลังตลอดกาล

```
sudo tee /etc/systemd/system/udp-custom.service > /dev/null <<EOF
[Unit]
Description=UDP Custom ARM64 Server
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/udp
ExecStart=/root/udp/udp-custom server --config /etc/config.json
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
EOF
```

```
sudo systemctl daemon-reload
sudo systemctl enable udp-custom
sudo systemctl start udp-custom

sudo systemctl status udp-custom
```
วิธีลบ

ล้างกฎ Iptables
```
sudo iptables -t nat -D PREROUTING -p udp --dport 1:65535 -j REDIRECT --to-ports 36712
sudo iptables -t nat -D PREROUTING -p udp --dport 1:59208 -j REDIRECT --to-ports 36712
sudo iptables -t nat -D PREROUTING -p udp --dport 59210:65535 -j REDIRECT --to-ports 36712

```

ปิดการทำงานและลบ Systemd Service
```
sudo systemctl stop udp-custom
sudo systemctl disable udp-custom
sudo rm /etc/systemd/system/udp-custom.service
sudo systemctl daemon-reload
```

ลบโฟลเดอร์และไฟล์โปรแกรมทิ้ง

```
sudo rm -rf /root/udp
sudo rm -rf /root/UDP-Custom-Installer-arm64
sudo rm /root/udpc-installer.sh
sudo rm /root/udp.sh
sudo rm /etc/config.json
```

ลบบัญชีผู้ใช้

```
sudo userdel -r userrrr
```
