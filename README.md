<h2 align="center">UDP Custom - Installer - ARM[64]<h2>

### Supported OS
- ubuntu 20.04 [arm] above ✅ _(recommended)_

### วิธีติดตั้ง
```
sudo -i
``` 
```
clear; wget --no-check-certificate "https://raw.githubusercontent.com/thoedrit13/UDP-Custom-Installer-arm64/main/udpc-installer.sh" -O udpc-installer.sh && chmod +x udpc-installer.sh && ./udpc-installer.sh
```
```
./udpc-installer.sh --help
```
อย่าเพิ่งใช้ ตั้งค่า config ก่อน
```
sudo nano /etc/config.json
```
ถ้าต้องการ ยกเว้น port ให้เพิ่ม 
```
"exclude": [
  53,
  5300,
  59209
]
```
```
{
  "listen": ":36712",
  "stream_buffer": 33554432,
  "receive_buffer": 83886080,
  "exclude": [
    59209
  ],
  "auth": {
    "mode": "passwords"
  }
}
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



หลังจากติดตั้ง ตั้งค่า config และสร้าง service เสร็จ ให้ reboot เครื่อง เพื่อสร้างกฏ iptables ขึ้นมา
เช็คจาก

```
sudo iptables -t nat -L PREROUTING -n --line-numbers
```


ถ้าต้องการยกเว้น port และทำผ่าน config ไม่ผ่าน

ลบ iptables ตัวเดิมออก
```
iptables -t nat -L PREROUTING -n --line-numbers
```
ดูว่าคือ rule ไหน เช่น 2 
```
iptables -t nat -D PREROUTING 2
```
เพิ่ม rule ตัวใหม่เอง
เช่น ยกเว้น 51820
```
iptables -t nat -I PREROUTING 1 -p udp -m multiport ! --dports 51820 -m addrtype --dst-type LOCAL -j DNAT --to-destination :36712
```
ยกเว้น 12451 51820
```
iptables -t nat -I PREROUTING 1 -p udp -m multiport ! --dports 51820,12451 -m addrtype --dst-type LOCAL -j DNAT --to-destination :36712
```
ใช้จริง
```
sudo iptables -t nat -I PREROUTING 1 -p udp -m multiport ! --dports 53,68,111,546,5353,7359,12451,41641,51820,53602 -m addrtype --dst-type LOCAL -j DNAT --to-destination :36712
```
เก็บถาวรด้วย
```
iptables-save > /etc/iptables/rules.v4
apt install iptables-persistent
```

จากนั้นขั้นตอนปกติ

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



วิธีลบ

ล้างกฎ Iptables
```
เช่น
iptables -t nat -D PREROUTING -p udp -m multiport ! --dports 51820 -m addrtype --dst-type LOCAL -j DNAT --to-destination :36712
iptables -t nat -D PREROUTING -p udp -m multiport ! --dports 51820,12451 -m addrtype --dst-type LOCAL -j DNAT --to-destination :36712
หรือใช้วิธี ลบตามเลข เช่น 1
iptables -t nat -L PREROUTING -n --line-numbers
iptables -t nat -D PREROUTING 1
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
