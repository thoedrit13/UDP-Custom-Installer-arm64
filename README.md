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
```
{
  "listen": ":36712",
  "stream_buffer": 33554432,
  "receive_buffer": 83886080,
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
Wants=network-online.target
After=network-online.target docker.service

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



ถ้าต้องการยกเว้น port 

ไม่ต้องลบ iptables ตัวเดิมออก
ให้ทำการแก้ที่ systemctl
```
nano /etc/systemd/system/udp-custom.service
```
แก้ตรง ExecStart เพิ่ม --exclude เช่น 53,68,111,546,5353,7359,12451,41641,51820,53602 ใช้จริง
```
ExecStart=/root/udp/udp-custom server --config /etc/config.json --exclude 53,68,111,546,5353,7359,12451,41641,51820,53602
```
ยกเว้น 51820
```
ExecStart=/root/udp/udp-custom server --config /etc/config.json --exclude 51820
```

บันทึกแล้ว
```
systemctl daemon-reload
systemctl restart udp-custom.service
```
ถ้าต้องการเช็ค หรือ ถ้าต้องการลบ iptables ตัวเดิมออก
```
iptables -t nat -L PREROUTING -n --line-numbers
```
ดูว่าคือ rule ไหน เช่น 2
```
iptables -t nat -D PREROUTING 2
sudo iptables-save > /etc/iptables/rules.v4
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

ล้างกฎ Iptables ให้หมด

```
iptables -t nat -L PREROUTING -n --line-numbers
```
ดูว่าคือ rule ไหน เช่น 2
```
iptables -t nat -D PREROUTING 2
sudo iptables-save > /etc/iptables/rules.v4
apt install iptables-persistent
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
